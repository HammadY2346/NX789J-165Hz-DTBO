# How it works

## The panel

The RedMagic 10 Pro / 10S Pro uses a **Raydium RM692H5** DDIC panel
(`BF237_RM692H5_6P8_DEV0/DEV1_DSC_CMD`), 1216×2688, DSC 8bpp, driven **command
mode** (`dsi_cmd_mode`) off a Snapdragon 8 Elite (SM8750, SoC id `0x26a`). It's the
same DDIC family as the RedMagic 11 Pro, which is already pushed to 165 Hz — so the
silicon has the headroom; stock firmware just doesn't expose a mode above 144.

## Stock timings

The panel node's `qcom,mdss-dsi-display-timings` ships four modes, all with the
same MDP transfer time:

| timing | framerate | transfer-time | h-back / h-front porch |
|--------|-----------|---------------|------------------------|
| @0 | 60 Hz  | `0x185b` (6235 µs) | 272 / 248 |
| @1 | 90 Hz  | `0x185b` | 128 / 108 |
| @2 | 120 Hz | `0x185b` | 60 / 32 |
| @3 | 144 Hz | `0x185b` | 12 / 8 |

For a command-mode panel the refresh is paced by the MDP transfer time / DSI clock,
not by a video-mode pixel clock, so adding a faster mode is mostly a matter of the
`transfer-time-us` value plus the `framerate` label.

## The edit

A new **`timing@4`** is cloned from the 144 Hz `timing@3`, changing only three
fields:

| field | 144 Hz | 165 Hz |
|-------|--------|--------|
| `qcom,mdss-dsi-panel-framerate` | `0x90` (144) | `0xa5` (165) |
| `qcom,mdss-mdp-transfer-time-us` | `0x185b` (6235) | `0x1542` (5442) |
| `cell-index` | `0x3` | `0x4` |

5442 µs comes from `6235 × 144/165 ≈ 5442`; it must be below 6060 µs
(= 1/165 s) for the DSI link to sustain 165. Everything else — DSC config,
`phy-timings`, porches, on-command init — is left identical to the stock 144 Hz mode.

This is applied to **both** the DEV0 and DEV1 panel variants, in every overlay that
contains the RM692H5 primary panel (the bootloader selects one by `board-id`). The
DTBO container is then rebuilt with corrected entry offsets/sizes. Built separately
from the A15 and A16 stock dtbos, because the DEV1 block and other overlays differ
between firmwares.

## Verification

After flashing and selecting 165, SurfaceFlinger confirms it's a true 165, not a
mislabel:

```
activeMode = {id=1, resolution=1216x2688, vsyncRate=165.00 Hz}
peak-refresh-rate : 165.00 Hz
FPS min:165 max:165 cur:165   TransferTime: 5442us
v_total: 2760  h_total: 1244  clk: 566517
```

Panel clock math: `566517 kHz / (2760 × 1244) = 165.0 Hz` exactly, and the active
`TransferTime` is the injected 5442 µs.
