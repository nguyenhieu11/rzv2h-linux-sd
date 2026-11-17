# RZ/V2H Linux SD Driver Notes

Notes and source references for the SD card stack of the Renesas RZ/V2H (R9A09G057) EVK. They are based on the Renesas CIP kernel [rz_linux-cip](https://github.com/renesas-rz/rz_linux-cip), branch `rz-6.1-cip48`, commit `1cac8c7e4`.

| File | Content |
|---|---|
| [zdoc1_build_guide.md](zdoc1_build_guide.md) | How to clone and cross-compile the kernel (Image, DTB, modules) |
| [zdoc2_sd_driver.md](zdoc2_sd_driver.md) | SDHI driver structure and the SD read path down to register level: commands, responses, DMA, interrupts |
| [build.log](build.log) | Output of a successful `make Image dtbs modules` build |
| [related_source/](related_source/) | Kernel source files on the path from `read()` to the SDHI driver, with the kernel's folder layout kept. See its [README](related_source/README.md). |

The files in `related_source/` are unmodified copies from the Linux kernel. They are licensed under GPL-2.0 as stated in each file's SPDX header.
