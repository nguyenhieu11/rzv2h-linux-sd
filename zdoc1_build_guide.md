# RZ/V2H Linux Kernel Build Guide (rz-6.1-cip48)

Build the Renesas RZ Linux CIP kernel for the RZ/V2H (R9A09G057) EVK on an x86_64 Ubuntu host with the Ubuntu aarch64 cross-compiler.

| Item | Value |
|---|---|
| Source | https://github.com/renesas-rz/rz_linux-cip |
| Branch | `rz-6.1-cip48` |
| Commit built | `1cac8c7e4` (arm64: defconfig: enable RZ/G2L EDAC support) |
| Kernel release | `6.1.157-cip48-g1cac8c7e4cdb` |
| Host | Ubuntu 22.04 (WSL2), 12 cores |
| Cross-compiler | `aarch64-linux-gnu-gcc` 11.4.0 (Ubuntu package) |
| Working directory | `~/rzv2h_linux_kernel` |

---

## 1. Install host packages

```bash
sudo apt-get update
sudo apt-get install -y git make gcc bc bison flex \
    libssl-dev libelf-dev libncurses-dev \
    gcc-aarch64-linux-gnu
```

> **Note:** You need `libssl-dev`. Without it the build fails with:
> ```
> certs/extract-cert.c:21:10: fatal error: openssl/bio.h: No such file or directory
> ```

To check that the cross-compiler works:

```bash
aarch64-linux-gnu-gcc --version
```

## 2. Clone the source

A shallow clone of only the needed branch saves time and disk space:

```bash
mkdir -p ~/rzv2h_linux_kernel
cd ~/rzv2h_linux_kernel
git clone --depth 1 -b rz-6.1-cip48 https://github.com/renesas-rz/rz_linux-cip.git
cd rz_linux-cip
git log --oneline -1
```

> To get the full history later, run `git fetch --unshallow`.

## 3. Set the build environment

Run these in every new shell before building:

```bash
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-
```

## 4. Configure

The arm64 `defconfig` in this branch already enables RZ/V2H support:

```bash
make defconfig
```

To check that it is enabled:

```bash
grep -E 'CONFIG_ARCH_RENESAS|CONFIG_ARCH_R9A09G057' .config
# CONFIG_ARCH_RENESAS=y
# CONFIG_ARCH_R9A09G057=y
```

To change options (optional):

```bash
make menuconfig
```

## 5. Build

```bash
make -j$(nproc) Image dtbs modules 2>&1 | tee ../build.log
```

The build ends with:

```
  NM      System.map
  SORTTAB vmlinux
  OBJCOPY arch/arm64/boot/Image
```

## 6. Build outputs

| Output | Path | Size |
|---|---|---|
| Kernel image | `arch/arm64/boot/Image` | ~24 MB |
| EVK device tree | `arch/arm64/boot/dts/renesas/r9a09g057h44-rzv2h-evk.dtb` | ~83 KB |
| Kernel modules | `*.ko` across the tree | 38 modules |

To check the outputs:

```bash
ls -lh arch/arm64/boot/Image \
       arch/arm64/boot/dts/renesas/r9a09g057h44-rzv2h-evk.dtb
cat include/config/kernel.release
```

## 7. Install modules to a staging folder

```bash
make modules_install INSTALL_MOD_PATH=../modules_out
```

The modules are placed in `../modules_out/lib/modules/<kernel-release>/`. Copy that folder to `/lib/modules/` in the target root filesystem.

---

## Rebuilding

After changing source or config, set the environment again (step 3) and rerun step 5. Only the changed parts are rebuilt.

To start from a clean tree:

```bash
make mrproper      # removes .config and all build output
make defconfig
make -j$(nproc) Image dtbs modules
```

## Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| `openssl/bio.h: No such file or directory` | Install `libssl-dev` |
| `gelf.h: No such file or directory` | Install `libelf-dev` |
| `bison: not found` / `flex: not found` | Install `bison` and `flex` |
| `aarch64-linux-gnu-gcc: not found` | Install `gcc-aarch64-linux-gnu`, or check `CROSS_COMPILE` |
| Host x86 kernel is built instead of arm64 | `ARCH=arm64` was not exported in the current shell |

> **Alternative toolchain:** Renesas's official BSP builds this kernel with a Yocto SDK toolchain. If you have that SDK installed, run `source /opt/poky/<ver>/environment-setup-aarch64-poky-linux` instead of step 3. The SDK sets `ARCH` and `CROSS_COMPILE` for you.
