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

- `modules/mtk_gpufreq_mt6789.ko`: one instruction patched to unlock the GPU's full signed-off
  OPP table (opt-in overclock). This MD-07P's GPU is efuse segment 1, which makes
  `__gpufreq_pdrv_probe` start the working table at signed OPP 7 (1003 MHz) and stop at 37.
  Segment 0 parts use the whole 45-entry table, up to 1100 MHz.
  - `orr w8, w8, w9` at `.text+0x7c68` (file offset `0x8c68`, `08 01 09 2a`) is changed to
    `mov w8, wzr` (`e8 03 1f 2a`). The "segment 1" flag is always clear, so the driver builds
    the segment 0 table (OPP 0-44).
  - Voltages are not changed. AVS still sets each OPP from this chip's efuse; on the test unit
    1100 MHz runs at 0.775 V (1003 MHz: 0.744 V).
  - The device tree caps devfreq `max_freq` at 1003000000 at boot (`init.mt6789.rc`), so the
    default stays stock. Settings > System > GPU boost (`persist.sys.md07p.gpu_boost`) raises
    it to 1100000000; writing `/sys/class/devfreq/13000000.mali/max_freq` directly (for
    example with a kernel tuning app) also works. Thermal throttling is a separate devfreq
    request and still applies. The working table indices shift by 7, so the device tree's
    `powerhint.json` GPU levels were shifted to match.

  The kernel has no module signing (`CONFIG_MODULE_SIG` unset).
