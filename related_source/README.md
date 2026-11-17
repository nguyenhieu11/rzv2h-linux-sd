# RZ/V2H SD Read Path: Related Source Files

These are the kernel source files on the path from an application's `read()` down to the RZ/V2H SDHI driver and its hardware description. They are copied from `rz_linux-cip/` (branch `rz-6.1-cip48`, commit `1cac8c7e4`) with the original folder structure kept. Every `.c` file here is compiled into the arm64 `defconfig` build.

See [../zdoc2_sd_driver.md](../zdoc2_sd_driver.md) for how these files work together.

The application itself is user-space code and isn't part of the kernel tree. The path starts at the `read()` system call.

| # | Layer | Files | Role |
|---|---|---|---|
| 1 | Syscall / VFS | `fs/read_write.c` | `read()` → `vfs_read()` |
| | | `block/fops.c` | direct reads of `/dev/mmcblk1` (e.g. `dd`) |
| 2 | Filesystem | `fs/ext4/file.c`, `fs/ext4/inode.c`, `fs/ext4/readpage.c` | ext4 (rootfs): file offset → disk blocks → `bio` |
| | | `fs/fat/inode.c` | FAT (boot partition) |
| 3 | Page cache | `mm/filemap.c`, `mm/readahead.c` | cache lookup, readahead, submits reads on a miss |
| 4 | Block layer | `block/bio.c`, `block/blk-core.c`, `block/blk-merge.c`, `block/blk-mq.c`, `block/blk-mq.h` | `bio` → merged `struct request` → driver queue |
| 5 | MMC core | `drivers/mmc/core/` (whole folder) | `queue.c`: blk-mq entry; `block.c`: `/dev/mmcblkN`, request → CMD23/CMD18; `core.c`: `mmc_start_request()`, `mmc_request_done()`; `sd.c`, `sd_ops.c`: SD card init; `host.c`, `bus.c`: host/card registration |
| 6 | MMC headers | `include/linux/mmc/*.h` | `struct mmc_host`, `mmc_request`, `mmc_command`, `mmc_data`, opcodes |
| 7 | SDHI host driver | `drivers/mmc/host/tmio_mmc_core.c`, `tmio_mmc.h` | generic TMIO engine: command send, IRQ handler, register map |
| | | `drivers/mmc/host/renesas_sdhi_core.c`, `renesas_sdhi.h` | Renesas probe, clocks, reset, SCC tuning, voltage switch |
| | | `drivers/mmc/host/renesas_sdhi_internal_dmac.c` | OF match `renesas,sdhi-r9a09g057`, internal DMAC ops |
| | | `include/linux/mfd/tmio.h` | TMIO platform flags (`TMIO_MMC_*`) |
| | | `drivers/mmc/host/Kconfig`, `Makefile` | build options (`CONFIG_MMC_SDHI*`) |
| 8 | Device Tree | `arch/arm64/boot/dts/renesas/r9a09g057.dtsi` | SoC: `sdhi0/1/2` nodes (address, IRQ, clocks, reset) |
| | | `arch/arm64/boot/dts/renesas/r9a09g057h44-rzv2h-evk.dts` | EVK: enables `sdhi1` (microSD), pins, VccQ regulator |
| | | `arch/arm64/boot/dts/renesas/rzv2-evk-cn15-sd.dtso`, `rzv2-evk-cn15-emmc.dtso` | overlays: SDHI0 on connector CN15 (SD or eMMC) |
| | | `include/dt-bindings/clock/renesas,r9a09g057-cpg.h`, `include/dt-bindings/pinctrl/renesas,r9a09g057-pinctrl.h` | constants used by the DT |
| | | `Documentation/devicetree/bindings/mmc/renesas,sdhi.yaml` | DT binding rules for SDHI nodes |
| 9 | SoC support drivers | `drivers/clk/renesas/r9a09g057-cpg.c`, `rzv2h-cpg.c`, `rzv2h-cpg.h` | SDHI clocks (`core`, `clkh`, `cd`, `aclk`) and reset |
| | | `drivers/pinctrl/renesas/pinctrl-rzg2l.c` | SD pin mux and the `default` / `state_uhs` pin states |
| | | `drivers/regulator/gpio-regulator.c` | VccQ 3.3 V / 1.8 V switch on GPIO PA2 |
