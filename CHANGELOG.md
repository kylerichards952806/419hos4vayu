# HyperOS 4 vayu port — change log

Last updated: 2026-11-05

## Credits and attribution rules

- ROM credit is **kyler084**: Settings → About phone shows `OS4.0.1.0.XPMEUXM | kyler084`,
  release post signed `By @kyler084`.
- **No Codebuff / Freebuff references on GitHub.** This repository must not contain AI
  attribution in files, commit messages, or co-author trailers. The first commit was
  amended and force-updated to remove it; keep future commits free of it.
- Base-port maintainer line (`@dykssvayuuser`) removed from `updater-script`.
- Upstream components reused: duckyduckG 4.19 sm8150 kernel (`lineage-23.2_xiaomi`) and
  vayu device tree (`DUCKS-QCM-5-4/android_device_xiaomi_vayu`, `lineage-23.2_4.19`).

## Port (system side)

- Built `vayu_myron_OS4.0.1.0.XPMEUXM_hyperos4-port.zip`
  (4,040,341,946 bytes, md5 `117ab527e83e4a080b5fa4e7268beda1`).
- Donor: myron EEA `OS4.0.1.0.XPMEUXM` — took `system`, `system_ext`, `product`.
- Base: `vayubhima-HyperOS4.0.0.5.XOBCNXM` — kept `vendor`, `odm`, `mi_ext`, `boot`,
  `dtbo`, `vbmeta*`, firmware.
- Grafted vayu 32-bit runtime back into donor system/system_ext (libs, ART, binaries).
- Rebuilt `system`, `system_ext` and `product` as ext4 (boot ramdisk fstab has no erofs
  entry for them); `vendor`/`odm`/`mi_ext` unchanged from base.
- New `super.img` 9,126,805,504 bytes, 6 partitions, liblp verified.
- Debloated ~30 packages; identity/props tuned for vayu (fingerprint, board, heap, 120 Hz).
- Removed maintainer line from `updater-script`.
- Flash test: super/kernel/dtbo/vbmeta/firmware wrote fine, device still bootlooped.
- Root causes identified: (1) product filesystem mismatch — fixed by ext4 rebuild;
  (2) donor `com.android.runtime.apex` has no 32-bit payload while the vendor needs
  32-bit bionic — fix prepared (install base 32-bit linker+bionic as real files,
  `ro.zygote=zygote64`, arm64-only abilist) but **not yet run/verified**.

## Kernel (real 4.19)

- Cloned `duckyduckG/android_kernel_xiaomi_sm8150_419`, branch `lineage-23.2_xiaomi`
  (1.5 GB), vayu DTS at `arch/arm64/boot/dts/xiaomi/vayu`, configs
  `vendor/sm8150-perf_defconfig` + `vendor/xiaomi/sm8150-common.config` + `vendor/xiaomi/vayu.config`.
- Local build (clang 14, `-j12`) reached 1,242 objects with no errors, then was
  interrupted by tool time limits; no `Image`/`dtbo.img` produced yet.
- Tree has a real `dtbo.img` target (`arch/arm64/Makefile: KBUILD_DTBO_IMG`) and
  `scripts/mkdtboimg.py`, so dtbo can be built in CI.
- GitHub Actions workflow (ubuntu-22.04) builds Image, dtbo.img and modules. It is
  prepared locally but needs the `workflow` OAuth scope before it can be pushed
  (`gh auth refresh -h github.com -s workflow`).

## Vendor 4.19 adaptation — NOT FINISHED

Done so far:

- `vendor.new.dat.br` decompressed to `vendor419.img` (737,619,968 bytes; transfer list
  v4, 180,083 blocks, range-form commands `new <n>,<start>,<end>`).
- Inventory of the 4.19 vendor:
  - MIUI-14-blob based: `ro.vendor.build.fingerprint` = `Xiaomi/vayu_global/vayu:13/...V14.0.3.0.TJUMIXM`.
  - fstab: `system`/`system_ext`/`product` ext4, **`vendor`/`odm` erofs**.
  - 75 `etc/init` rc files, Lineage-style HAL set.
  - 32-bit HALs retained: audio, cas, media.omx; 312 lib32, 647 lib64, 7 32-bit bins.
  - **No `.ko` modules** — drivers built into ducky's kernel.

Remaining work:

1. Choose merge direction: 4.19 vendor + grafted HOS4/MIUI init/props, or base vendor +
   matching 4.19 modules.
2. Repack vendor/odm to the filesystem the boot ramdisk expects (or patch fstab).
3. Supply 4.19 modules + regenerated `modules.dep`/`modules.load` once the kernel builds.
4. Rebuild super with the adapted vendor and boot-test on device.

## 2026-11-05 evening — 4.14 baseline and vbmeta finding

- The verified base zip boots again after flashing its own super/boot/dtbo/vbmeta, so
  the device and flash path are healthy; the failure is in the donor content.
- Both zips ship vbmeta flags 0x0; forcing flags 0x3 on vbmeta and vbmeta_system did
  not make the 4.14 donor port boot, so verification is not the sole cause.
- 32-bit bionic installed as real files with `ro.zygote=zygote64`.
- Artifacts split into `414/` and `419/` folders (boot, dtbo, super, vbmeta).
- `super_donorsys.img` built (donor system only, base everything else) for isolation.

## 2026-11-05 late — debloat reverted, product moved to erofs

- All 30 debloated packages are back in product (2.35 GB restored from the donor).
- A full product tree (5.77 GB) does not fit as ext4 in the 8.5 GB super, so product
  is now erofs (3.97 GB, lz4hc), matching the donor's own format. The 4.14 base
  kernel has erofs built in and the 4.19 config sets `CONFIG_EROFS_FS=y`.
- Product fstab entry changed ext4 -> erofs in the boot ramdisk and both vendor
  images; boot_414/boot_419 repacked; supers rebuilt (220 MB free of 9.127 GB).
- Old ext4 supers kept as `*_ext4.img`; vbmeta flags 0x3 copies in both folders.
- Flashable 4.14 zip: `414/vayu_myron_OS4.0.1.0.XPMEUXM_hyperos4-port_4.14.zip`
  (5,414,559,611 bytes, md5 `0213bec2154715fc76a1b406016d4378`). Only the 4.14
  zip was built for now.
