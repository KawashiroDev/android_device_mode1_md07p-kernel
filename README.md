# Mode1 Pocket (MD-07P) kernel prebuilts

Prebuilt kernel for `device/mode1/md07p`, taken from the stock firmware
(`MD-07P:15/AP3A.240905.015.A2`, slot A) of a Mode1 Pocket.

| Path | Source |
| --- | --- |
| `Image.gz` | Stock `boot_a` kernel (GKI 5.10.226-android12-9) |
| `dtb/mt6789.dtb` | Stock `vendor_boot_a` DTB blob, byte for byte |
| `dtbo.img.stock` | Stock `dtbo_a`, for reference / restoring stock (not used by the build) |
| `modules/` | Stock vendor ramdisk + `vendor_dlkm` kernel modules |
| `modules.load*` | Stock module load lists (`modules.load`, vendor ramdisk, recovery) |
| `kernel-headers/` | UAPI headers from 2by2's SX3 kernel source, `SUBLEVEL` set to 226 to match |

`Image.gz` is an Android GKI kernel; its source is Android's common kernel
(`android12-5.10`). The vendor modules are MediaTek's.

Used by the device tree via `KERNEL_PATH := device/mode1/md07p-kernel`.
