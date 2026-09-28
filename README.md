# device_advan_6781-kernel

Kernel package for the Advan X1 (`6781`). Apache-2.0 for the packaging;
the binaries themselves are Advan/MediaTek's.

## Contents (all stock ADVAN_6781_S34NF2_V4.0_20250304 except Image.gz)

```
dtbo.img                  8,388,608 B, from the stock zip's dtbo.img
dtb/dtb                     189,223 B, from stock vendor_boot (sha256
                            5a424301176681c2 -- independently extracted
                            twice with identical bytes)
modules/vendor_dlkm/      180 .ko + modules.{alias,dep,load,softdep}
                          (modules.load: 159 entries, stock order)
modules/vendor_boot/      178 .ko + modules.{alias,dep,load,load.recovery,
                          softdep} (load: 161, recovery: 167, stock order)
Image.gz                  <-- ours, per variant (NOT shipped here; produced
                          by the kernel build or import-kernel.sh)
prebuilt/                 <-- import-kernel.sh target (Image.gz +
                          vmlinux.symvers + kernel.config + IMPORTED receipt).
                          Source: ~/itel-rs4-kernel, VANILLA variant -- the
                          same kernel the S666LN ships (operator, 2026-09-29;
                          the Advan-Kernel fork is retired as stale).
```

Every byte except `Image.gz` is stock output, so this is repackaging, not
authoring -- WITH ONE EXCEPTION, stated plainly:
`modules/vendor_dlkm/mali_kbase_mt6789.ko` is OUR r54p1 build (UK 11.46,
BM_2-correct, ged/DVFS integrated, ADVAN_GED_ON=1), built by
device/advan/6781/tools/kernel-patches/build-mali-kbase.sh against the
retired Advan-Kernel fork's vanilla build (5.10.268); it passes the KMI
gate unchanged against the shipped itel-rs4-kernel vanilla (5.10.269,
2026-09-29). Stock's r32p1 original is sha256
6c62585128317d1b020dad23cc25c0efaee16086ad969e847dc9550024633a91
(ours: 99d1cf6c9553ff71237d8d401c00fc87df8978d43bdb975bdf2388f52047d170,
2026-09-19). It installs under the stock filename so init.insmod finds it;
KMI holds (module_layout 0x7c24b32d, gate-verified). kbase and UMD must
land ATOMICALLY with the r54p1 userspace -- see the device-tree JOURNAL.
Module orders are stock's (`modules.load` files verbatim): a
wrong order is a watchdog reset ~5 s into boot, with no panic and no log.

## KMI

All 180 + 178 modules demand `module_layout = 0x7c24b32d`
(verified 2026-09-19 against the zip-chained images before staging, and
spot-checked again on the staged files). The kernel -- whether built from
source via `TARGET_KERNEL_SOURCE` or imported via `import-kernel.sh` --
must reproduce it; `Android.mk`'s KMI gate in the device tree enforces
this at build time against exactly these directories.
