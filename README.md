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

## Changes from stock

- `modules/leds-mtk.ko`: two instructions patched to remove the backlight floor and cap. Stock
  `mtk_set_brightness` maps levels 1-34 to 35 and caps levels at 191 (of 255, i.e. hw 1533 of
  2047). The floor makes the bottom of the brightness slider do nothing and keeps the screen
  fairly bright at minimum:
  - floor: `cmp w8, #0x22` at `.text+0xd98` (file offset `0x1d98`, `1f 89 00 71`) is changed to
    `cmp w8, #0x0` (`1f 01 00 71`) so the floor check never matches. Level 0 still turns the
    backlight off.
  - cap: `csel w23, w8, w10, lo` at `.text+0xda8` (file offset `0x1da8`, `17 31 8a 1a`) is
    changed to `mov w23, w8` (`f7 03 08 2a`). The device tree's lights HAL applies the stock
    191 cap instead, unless Settings > Display > Brightness boost is on.

  The kernel has no module signing (`CONFIG_MODULE_SIG` unset).
