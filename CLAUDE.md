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
- `STAY_HOME` (0): Primary layer. Home row mods — Cmd/Opt/Ctrl on A/S/D and K/L/; via the `&hml` / `&hmr` hold-tap behaviors.
- `NUMPAD` (1): Right-hand numpad (7-8-9 / 4-5-6 / 1-2-3 / 0), plus grave, backslash, apostrophe.
- `NUM2` (2): Left-hand numbers with brackets/parens, right-hand arrows + media controls. Sticky FNS on right thumb.
- `NAV` (3): Brackets/parens on left, arrows + media on right. Similar to NUM2 but without numbers. The left half is `&trans`, so home row mods stay active (e.g. hold A + arrow = Cmd+arrow).
- `FNS` (4): Bluetooth profile selection (1-5) and BT_CLR. Mute as a "canary" key. `&bootloader` and `&sys_reset` on the outer column of each half (these act only on the half where the key is pressed, so each half needs its own). `&soft_off` (hold 1 s) at the bottom-left; it powers both halves fully off, and each half's reset button turns it back on.

**Conditional layers:**
- NUMPAD + NAV held together activates FNS (`tri_layer` node). FNS must stay the highest-numbered of the three layers.

**Combos:**
- F+J (positions 16+19): ESC
- G+H (positions 17+18): Caps Word

Both combos use `require-prior-idle-ms = 150`: they do not fire if another key was pressed in the previous 150 ms. This stops accidental triggers during fast typing (e.g. "gh" in "night").

**Key position grid** (for combo definitions):
```
 0   1   2   3   4   5     6   7   8   9  10  11
12  13  14  15  16  17    18  19  20  21  22  23
24  25  26  27  28  29    30  31  32  33  34  35
                36  37  38    39  40  41
```

**Custom `hml` / `hmr` (homerow_mods_left / homerow_mods_right) behaviors:**

Two hold-tap behaviors with identical timing; they differ only in `hold-trigger-key-positions`. `hml` (A/S/D) resolves to hold only when the other key is on the right half or a thumb; `hmr` (K/L/;) only when it is on the left half or a thumb. The `KEYS_L`, `KEYS_R`, and `THUMBS` macros at the top of the keymap hold the position lists.
- `tapping-term-ms = 250` — hold duration before activating modifier
- `quick-tap-ms = 200` — window for rapid successive taps (taps immediately on second press)
- `require-prior-idle-ms = 200` — if any key was pressed in the last 200 ms, the key is always a tap
- `flavor = "balanced"` — decides tap vs hold based on whether the key is released before or after another key
- `hold-trigger-key-positions` + `hold-trigger-on-release` — same-hand keys resolve as tap, except other mods (so A+S = Cmd+Opt still works). A same-hand shortcut such as Cmd+C with A needs A held past the tapping term first; or use the opposite-hand mod.

### config/corne.conf

- Keyboard name: "Geo's Corne"
- No LEDs or OLED are installed, so RGB underglow, display, and external power control are left at defaults and have no keymap bindings
- Deep sleep: enabled, 30-minute timeout (`CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=1800000`)
- Peripheral battery proxy: enabled. The left half reports both battery levels to the host. Host support for two battery levels varies (macOS often shows only one).
- Soft off: enabled (`CONFIG_ZMK_PM_SOFT_OFF=y`) for the `&soft_off` key on FNS

### config/west.yml

West manifest pointing to `zmkfirmware/zmk` on `main`. Defines the firmware source to build against.

## Editing Guidelines

- Use the ASCII art in each layer definition to identify key positions before making changes.
- Each layer has a `display-name` (shown in ZMK Studio and on displays). Give new layers one too.
- A commented-out blank layer template is at the bottom of the keymap file — use it when adding new layers.
- Bluetooth profiles in the FNS layer use `&bt BT_SEL N` with N from 1–5.
