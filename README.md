# RedMagic 10 Pro / 10S Pro — NX789J 165Hz DTBO (Android 15 & 16): display overclocking to a stable 165 Hz refresh rate.

Adds a **native 165 Hz** display mode to the RedMagic 10 Pro / 10S Pro
(`NX789J` / `NX789S`), which ship capped at 144 Hz. Works on **Android 15 and
Android 16**. Confirmed running — SurfaceFlinger reports a true 165.00 Hz active
mode and the panel clock math checks out exactly.

![device](https://img.shields.io/badge/device-NX789J%20%2F%20NX789S-red)
![panel](https://img.shields.io/badge/panel-RM692H5-blue)
![android](https://img.shields.io/badge/Android-15%20%26%2016-green)
![refresh](https://img.shields.io/badge/refresh-144Hz%20%E2%86%92%20165Hz-orange)

> ⚠️ **Read the warnings.** This drives the panel above its rated 144 Hz by
> modifying the device-tree overlay. A bad flash gives a **black screen**
> (recoverable) and, in rare cases, could stress the panel. Flashing is at your
> own risk. Keep your stock `dtbo` backup.

---

## What it does

The RM692H5 is a command-mode DSC panel with stock timings of 60 / 90 / 120 /
144 Hz, all transferred in 6235 µs. This mod injects a new **`timing@4` = 165 Hz**
(cloned from the stock 144 Hz timing, transfer-time scaled to 5442 µs) into the
panel node of the device's own `dtbo`, so Android gets a real selectable 165 Hz
mode. Nothing else in the dtbo is changed. See
[docs/HOW-IT-WORKS.md](docs/HOW-IT-WORKS.md).

## Requirements

- RedMagic 10 Pro / 10S Pro (`NX789J` or `NX789S`), Android 15 **or** 16.
- **Unlocked bootloader** with AVB verification disabled (if you're already
  rooted via a patched `init_boot`, this is done).
- Root (KernelSU / Magisk) for the installer zip, **or** fastboot for the raw image.

## Pick the right file for your firmware

| Firmware | Installer zip (Magisk/KSU) | Raw image (fastboot) |
|----------|----------------------------|----------------------|
| **Android 16** | [NX789J-165Hz-dtbo-installer-v1.0.zip](downloads/NX789J-165Hz-dtbo-installer-v1.0.zip) | [NX789J-dtbo-165-A16.img](downloads/NX789J-dtbo-165-A16.img) |
| **Android 15** | [NX789J-165Hz-dtbo-installer-A15-v1.0.zip](downloads/NX789J-165Hz-dtbo-installer-A15-v1.0.zip) | [NX789J-dtbo-165-A15.img](downloads/NX789J-dtbo-165-A15.img) |

> Download: open a file above → **Download raw file** (the button top-right of the file view).

> Build from **Device firmware's own dtbo** — do not cross-flash A16 on A15.
> The DEV0 panel block is identical across the two, but the DEV1 variant and the
> rest of the container differ.

## Install

### Option A — Installer zip (recommended)
1. Download the zip for your firmware (above).
2. KernelSU / Magisk → **Modules → Install from storage** → pick the zip.
3. It checks your device, **backs up your current dtbo** to
   `/sdcard/dtbo_stock_<slot>_<date>.img`, writes the 165 Hz image, and asks you to reboot.
4. Reboot.

### Option B — Raw image (fastboot)
```sh
# back up first!
adb shell su -c "dd if=/dev/block/by-name/dtbo$(getprop ro.boot.slot_suffix) of=/sdcard/dtbo.stock.img"
adb reboot bootloader
fastboot flash dtbo NX789J-dtbo-165-A16.img     # or -A15 for Android 15
fastboot reboot
```

## Enable 165 Hz

The mode exists after flashing; tell Android to use it:
```sh
adb shell settings put system peak_refresh_rate 165
adb shell settings put system min_refresh_rate 60    # 60 lets it idle down; set 165 to force constant
```
Also set the highest option in **RedMagic Settings → Display → Refresh rate**, and
make sure no battery saver / 60 Hz lock is active.

Verify:
```sh
adb shell "dumpsys SurfaceFlinger | grep -iE 'activeMode|peak-refresh|FPS '"
# expect: activeMode ... 165.00 Hz   and   FPS ... cur:165
```

## Restore stock

```sh
adb reboot bootloader
fastboot flash dtbo dtbo.stock.img     # the backup you made / the zip saved to /sdcard
fastboot reboot
```

## After a firmware update

An OTA resets `dtbo` to stock. Just flash the installer zip again (it re-backs-up
and re-applies).

## Why this is a stable, fixed 165 Hz

This mod defines a **discrete, pinned 165 Hz mode** — a fixed `framerate` with a
`transfer-time` that leaves real DSI headroom — rather than letting the DDIC
free-run at its ceiling via an "auto/max" register. The result is a clean, locked
165.00 Hz (`DynFPS:false`, policy pinned `[165,165]`, pixel clock lands on exactly
165.0), instead of the refresh **hunting between 180 / 175 / 165** that an
auto-negotiated approach can produce. Stable by design, not by luck.

## Credits

- **Inspiration — [KahramanEdit](https://github.com/KahramanEdit/NX809J-165Hz-DTBO)**:
  their NX809J (RedMagic 11 Pro) project showed that an RM692H5 panel can be driven
  past its stock cap with a device-tree change. That proved the idea was possible —
  thanks for the spark.
- **Built by Hammad Yasin** ([@HammadY2346](https://github.com/HammadY2346)):
  this is an **independent, from-scratch** mod for the NX789J, **not a port**. No
  NX809J files are used — the two phones have different SoCs (SM8750 vs SM8850) and
  firmware. The RedMagic 10/10S Pro's own `dtbo` was analyzed directly, the RM692H5
  panel's stock timings were read off the device, and a 165 Hz mode was synthesized
  from the phone's own 144 Hz timing, built separately for Android 15 and 16.

## License

[MIT](LICENSE).

## Disclaimer

Provided as-is, with no warranty. Modifying device partitions voids your warranty
and can brick your device. You are solely responsible for anything that happens to
your phone. Always keep your stock `dtbo` backup.
