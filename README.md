# 419hos4vayu — HyperOS 4 on POCO X3 Pro (vayu), real 4.19 kernel

Builds the **ducky 4.19 sm8150 kernel** (`android_kernel_xiaomi_sm8150_419`,
branch `lineage-23.2_xiaomi`) for vayu with a HyperOS 4 config fragment.

## Why CI

The kernel is a ~1.5 GB tree and a full build is CPU heavy. GitHub Actions
builds `Image`, `dtbo.img` and all vendor modules, and uploads them as the
`vayu-hos4-kernel` artifact. Local work is then limited to repacking `boot.img`
with the HOS4 ramdisk and flashing.

## Config

`configs/vayu-hos4.config` is merged over:

1. `arch/arm64/configs/vendor/sm8150-perf_defconfig`
2. `arch/arm64/configs/vendor/xiaomi/sm8150-common.config`
3. `arch/arm64/configs/vendor/xiaomi/vayu.config`

It enables EROFS (the 4.19 vendor mounts vendor/odm as erofs), binder, 32-bit
compat, PSI, loadable modules and disables module signing/debug info.

## Artifacts

- `Image` — kernel for `boot.img`
- `dtbo.img` — vayu 4.19 device tree overlays
- `modules.tar.gz` — `.ko` set (audio, wlan, rmnet, exfat, mpq, tspp, rdbg)
- `kernel.config` — resolved config
- `module-list.txt` — module names
