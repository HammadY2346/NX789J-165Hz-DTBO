# v1.0 — first NX789J 165 Hz release

Native 165 Hz for the RedMagic 10 Pro / 10S Pro (NX789J/NX789S), Android 15 & 16.

> ⚠️ **Android 15: not recommended for now.** On Android 15 global firmware,
> switching to 165 Hz through an LSPosed module caused screen flicker and reboots.
> Android 15 has not been re-tested yet; use Android 16 (China or global).

**Assets**
- `NX789J-165Hz-dtbo-installer-v1.0.zip`     — Android 16, Magisk/KSU installer
- `NX789J-165Hz-dtbo-installer-A15-v1.0.zip` — Android 15, Magisk/KSU installer
- `NX789J-dtbo-165-A16.img`                  — Android 16, raw (fastboot flash dtbo)
- `NX789J-dtbo-165-A15.img`                  — Android 15, raw (fastboot flash dtbo)

**Checksums (md5)**
```
dd4c92ced4bc92af88a7ee51e75ed5e2  NX789J-165Hz-dtbo-installer-A15-v1.0.zip
03346cbdc88eb06d774dc026c7b513b5  NX789J-165Hz-dtbo-installer-v1.0.zip
75c4d1b265dd56650452fbd628636a71  NX789J-dtbo-165-A15.img
ac80074e4fd9676145c4d7ff2f46ff1d  NX789J-dtbo-165-A16.img
```

Installer zips back up your stock dtbo to /sdcard before writing. See README for
enabling 165 Hz and restoring stock. Re-flash the zip after any firmware OTA.
