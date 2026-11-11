# CLAUDE.md

ZMK firmware configuration for a Corne (crkbd) split keyboard with Nice!Nano controllers. Configuration files use DeviceTree format (.keymap, .conf).

## Build & Test

There is no local build. Firmware is compiled in the cloud via GitHub Actions whenever changes are pushed.

- `build.yaml` defines the board/shield matrix (board: `nice_nano//zmk`, shields: `corne_left`, `corne_right`, `settings_reset`)
- `.github/workflows/build.yml` runs the build
- Download `.uf2` firmware artifacts from the Actions tab and copy to the keyboard's USB drive
- To reset keyboard settings, flash the `settings_reset` firmware

## Configuration Files

All config lives in `config/`.

### config/corne.keymap

The keymap defines 5 layers, combos, and a custom hold-tap behavior. Each layer has ASCII art showing the layout — use these as the authoritative reference for key positions.

**Layers:**
- `STAY_HOME` (0): Primary layer. Home row mods — Cmd/Opt/Ctrl on A/S/D and K/L/; via the `&hm` hold-tap behavior.
- `NUMPAD` (1): Right-hand numpad (7-8-9 / 4-5-6 / 1-2-3 / 0), plus grave, backslash, apostrophe.
- `NUM2` (2): Left-hand numbers with brackets/parens, right-hand arrows + media controls. Sticky FNS on right thumb.
- `NAV` (3): Brackets/parens on left, arrows + media on right. Similar to NUM2 but without numbers. The left half is `&trans`, so home row mods stay active (e.g. hold A + arrow = Cmd+arrow).
- `FNS` (4): Bluetooth profile selection (1-5) and BT_CLR. Mute as a "canary" key. External power off.

**Combos:**
- F+J (positions 16+19): ESC
- G+H (positions 17+18): Caps Word

**Key position grid** (for combo definitions):
```
 0   1   2   3   4   5     6   7   8   9  10  11
12  13  14  15  16  17    18  19  20  21  22  23
24  25  26  27  28  29    30  31  32  33  34  35
                36  37  38    39  40  41
```

**Custom `hm` (homerow_mods) behavior:**
- `tapping-term-ms = 300` — hold duration before activating modifier
- `quick-tap-ms = 200` — window for rapid successive taps (taps immediately on second press)
- `global-quick-tap` — any recent keypress (not just this key) triggers quick-tap
- `flavor = "balanced"` — decides tap vs hold based on whether the key is released before or after another key

### config/corne.conf

- Keyboard name: "Geo's Corne"
- RGB underglow: disabled
- OLED display: disabled
- External power: enabled
- Deep sleep: enabled, 30-minute timeout (`CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=1800000`)

### config/west.yml

West manifest pointing to `zmkfirmware/zmk` on `main`. Defines the firmware source to build against.

## Editing Guidelines

- Use the ASCII art in each layer definition to identify key positions before making changes.
- A commented-out blank layer template is at the bottom of the keymap file — use it when adding new layers.
- Bluetooth profiles in the FNS layer use `&bt BT_SEL N` with N from 1–5.
- The `global-quick-tap` property on the `hm` behavior is deprecated in newer ZMK — if upgrading ZMK, it should be replaced with `require-prior-idle-ms`.
