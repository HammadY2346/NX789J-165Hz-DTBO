# RedMagic 10 Pro / 10S Pro — NX789J 165Hz DTBO (Android 16): display overclocking to a stable 165 Hz refresh rate.

Adds a **native 165 Hz** display mode to the RedMagic 10 Pro / 10S Pro
(`NX789J` / `NX789S`), which ship capped at 144 Hz. Made and tested on
**Android 16**, where it works the same on China and global firmware. Confirmed
running — SurfaceFlinger reports a true 165.00 Hz active mode and the panel clock
math checks out exactly.

![device](https://img.shields.io/badge/device-NX789J%20%2F%20NX789S-red)
![panel](https://img.shields.io/badge/panel-RM692H5-blue)
![android](https://img.shields.io/badge/Android-16-green)
![android15](https://img.shields.io/badge/Android%2015-not%20recommended-yellow)
![refresh](https://img.shields.io/badge/refresh-144Hz%20%E2%86%92%20165Hz-orange)

> ⚠️ **Read the warnings.** This drives the panel above its rated 144 Hz by
> modifying the device-tree overlay. A bad flash gives a **black screen**
> (recoverable) and, in rare cases, could stress the panel. Flashing is at your
> own risk. Keep your stock `dtbo` backup.

> ⚠️ **Android 15: not recommended for now.** On Android 15 (RedMagic OS 10),
> **Settings → Display → Refresh rate does not show 165 Hz** even after flashing;
> its list stops at 144 Hz. That's why selecting 165 Hz there needs an LSPosed
> module. On Android 15 **global** firmware, switching to 165 Hz through such a
> module caused **screen flicker and reboots**. Android 15 seems stricter than
> Android 16 here, and it has not been re-tested yet. Until it is, use this mod
> on **Android 16** (China or global), where Settings lists 165 Hz by itself and
> no module is needed. If you still try the Android 15 file, keep your stock
> `dtbo` backup ready, and don't force 165 Hz through a module:
> [AutoHz Max](https://github.com/HammadY2346/RedMagic-AutoHz-Max) 1.3 and later
> no longer run on Android 15.

---

## What it does

The RM692H5 is a command-mode DSC panel with stock timings of 60 / 90 / 120 /
144 Hz, all transferred in 6235 µs. This mod injects a new **`timing@4` = 165 Hz**
(cloned from the stock 144 Hz timing, transfer-time scaled to 5442 µs) into the
panel node of the device's own `dtbo`, so Android gets a real selectable 165 Hz
mode. Nothing else in the dtbo is changed. See
[docs/HOW-IT-WORKS.md](docs/HOW-IT-WORKS.md).

## Requirements

- RedMagic 10 Pro / 10S Pro (`NX789J` or `NX789S`) on **Android 16** (Android 15: see the warning above).
- **Unlocked bootloader** with AVB verification disabled (if you're already
  rooted via a patched `init_boot`, this is done).
- Root (KernelSU / Magisk) for the installer zip, **or** EDL / fastboot for the raw image.

## Pick the right file for your firmware

| Firmware | Installer zip (Magisk/KSU) | Raw image (EDL / fastboot) |
|----------|----------------------------|----------------------|
| **Android 16** | [installer zip](https://github.com/HammadY2346/NX789J-165Hz-DTBO/releases/download/v1.0/NX789J-165Hz-dtbo-A16-installer-v1.0.zip) | [dtbo_165hz.A16.img](https://github.com/HammadY2346/NX789J-165Hz-DTBO/releases/download/v1.0/dtbo_165hz.A16.img) |
| **Android 15** ⚠️ not recommended, see the warning | [installer zip](https://github.com/HammadY2346/NX789J-165Hz-DTBO/releases/download/v1.0/NX789J-165Hz-dtbo-installer-A15-v1.0.zip) | [dtbo_165_a15.img](https://github.com/HammadY2346/NX789J-165Hz-DTBO/releases/download/v1.0/dtbo_165_a15.img) |

> All files live on the [**Releases**](https://github.com/HammadY2346/NX789J-165Hz-DTBO/releases/latest) page — grab the one matching your firmware.

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

### Option B — Raw image via EDL (ZTE / nubia tool)
Recommended on RedMagic, where fastboot can be unreliable. Flash **only the `dtbo`
partition** — do not touch anything else.

1. Back up your current dtbo first (this is your restore image — pull it to your PC):
   ```sh
   adb shell su -c "dd if=/dev/block/by-name/dtbo$(getprop ro.boot.slot_suffix) of=/sdcard/dtbo.stock.img"
   ```
2. Boot the phone into **EDL / 9008** mode and open your **ZTE / nubia EDL tool box**
   (QFIL or `edl` work too, with the SM8750 firehose).
3. Write **only** the `dtbo` partition with `NX789J-dtbo-165-A16.img` (or `-A15` for
   Android 15):
   ```sh
   # e.g. with bkerler's edl:
   edl w dtbo NX789J-dtbo-165-A16.img
   ```
4. Reboot.

> `fastboot flash dtbo <img>` also works if your fastboot is set up, but EDL is the
> reliable, always-recoverable route on RedMagic. Do **not** use `fastboot boot`.

## Enable 165 Hz

The mode exists after flashing; tell Android to use it:
```sh
adb shell settings put system peak_refresh_rate 165
adb shell settings put system min_refresh_rate 60    # 60 lets it idle down; set 165 to force constant
```
Also set the highest option in **RedMagic Settings → Display → Refresh rate**, and
make sure no battery saver / 60 Hz lock is active. On Android 16 that page lists
**165 Hz** by itself. On Android 15 it stops at 144 Hz and only an LSPosed module
can add 165 Hz there, which is not recommended right now (see the Android 15
warning at the top).

Verify:
```sh
adb shell "dumpsys SurfaceFlinger | grep -iE 'activeMode|peak-refresh|FPS '"
# expect: activeMode ... 165.00 Hz   and   FPS ... cur:165
```

## Restore stock

Flash your stock dtbo back the same way — **EDL**, writing **only** the `dtbo`
partition with the backup you made (or the one the installer zip saved to `/sdcard`):
```sh
# with bkerler's edl (or use your ZTE/nubia tool box / QFIL):
edl w dtbo dtbo.stock.img
```
Because EDL can rewrite any partition, a bad dtbo flash is always recoverable.

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
