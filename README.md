# Mattis Dactyl — ZMK Firmware

Custom split keyboard (Dactyl Manuform) running [ZMK firmware](https://zmk.dev/) on two [nice!nano v2](https://nicekeyboards.com/nice-nano/) boards.

## Hardware

| | Left (central) | Right (peripheral) |
|---|---|---|
| Regular keys | 36 | 36 |
| 5-way switch | 5 positions (col 7) | — |
| EC11 encoder | — | rotation + push button |
| **Total** | **41 positions** | **37 positions** |

**78 physical key positions** + encoder rotation via `sensor-bindings`.

The left side is the central half — it connects to the computer via Bluetooth. The right side is the peripheral and communicates with the left over BLE.

**Bluetooth name:** `Mattis Dactyl`

## Pin Mapping (nice!nano v2 / pro_micro)

### Rows (shared both sides)
Pins 20, 19, 18, 15, 14, 16 (6 rows)

### Left columns
Pins 2, 3, 4, 5, 6, 7, 8, 9 (7 key columns + 1 five-way switch)

### Right columns
Pins 8, 7, 6, 5, 4, 3, 2 (7 key columns, reversed order to match physical wiring)

### EC11 Encoder (right side only)
- **A signal:** pin 21
- **B signal:** pin 10
- **Push button:** pin 9 (wired directly to GND, not in matrix)

## Keymap Layers

| # | Name | Description |
|---|---|---|
| 0 | default | Standard QWERTY with Kinesis-style layout |
| 1 | raise | Symbols, vim navigation, special characters |
| 2 | lower | F-keys, numbers, brackets |
| 3 | adjust | Bluetooth, mouse control, bootloader |
| 4 | gaming | WASD gaming layout (toggle from adjust layer) |

The encoder controls volume on all layers (clockwise = up, counter-clockwise = down). Push button = mute.

## File Structure

```
├── .github/workflows/build.yml     # GitHub Actions CI — builds firmware on push
├── boards/shields/dactyl/
│   ├── dactyl.dtsi                  # Shared hardware: transform map, matrix, composite kscan,
│   │                                #   encoder node, sensors node, split node
│   ├── dactyl_left.overlay          # Left side: 8 column GPIOs, enables split
│   ├── dactyl_right.overlay         # Right side: 7 column GPIOs (reversed), EC11 push button
│   │                                #   (kscan-gpio-direct), adds direct child to composite,
│   │                                #   enables encoder and split
│   ├── dactyl_left.conf             # EC11 Kconfig for central (processes peripheral sensor events)
│   ├── dactyl_right.conf            # EC11 + direct kscan polling Kconfig for peripheral
│   ├── Kconfig.defconfig            # Split roles (left=central, right=peripheral), BT name
│   ├── Kconfig.shield               # Shield detection booleans
│   └── dactyl.zmk.yml               # Shield metadata
├── build.yaml                       # GitHub Actions build matrix (nice_nano_v2 × left/right/reset)
├── config/
│   ├── dactyl.keymap                # Active keymap: 5 layers × 78 bindings, macros, sensor-bindings
│   ├── dactyl.conf                  # Shared Kconfig: BLE, split, mouse
│   ├── info.json                    # Visual layout for keymap-editor (78 entries + encoder)
│   └── west.yml                     # ZMK v0.2 manifest
└── zephyr/module.yml                # Zephyr module root
```

## Flashing

### Download firmware

1. Go to the **Actions** tab on GitHub
2. Click the latest successful **Build ZMK firmware** run
3. Download the **firmware** artifact (zip file)
4. Extract — you'll find:
   - `dactyl_left-nice_nano_v2-zmk.uf2`
   - `dactyl_right-nice_nano_v2-zmk.uf2`
   - `settings_reset-nice_nano_v2-zmk.uf2`

Or use the GitHub CLI:
```bash
gh run list --limit 1
gh run download <run-id> -n firmware
```

### Flash a half

1. Connect the nice!nano v2 via USB
2. **Double-tap the reset button** — it enters bootloader mode and mounts as a USB drive (`NICENANO`)
3. Drag the `.uf2` file onto the drive (left file for left half, right file for right half)
4. The board will automatically reboot with the new firmware

Flash **both halves** whenever you update the firmware.

### Settings reset

If the halves won't pair with each other or with your computer:

1. Flash `settings_reset-nice_nano_v2-zmk.uf2` to **both** halves (one at a time)
2. Then flash the regular left and right firmware again
3. The halves will re-pair with each other automatically

## Bluetooth Pairing

1. Flash both halves with the latest firmware
2. The left half (central) will advertise as **Mattis Dactyl**
3. On your computer, open Bluetooth settings and pair with **Mattis Dactyl**
4. The keyboard supports up to 5 Bluetooth profiles — switch between them using `BT_SEL 0` through `BT_SEL 4` on the adjust layer (layer 3)
5. Use `BT_CLR` on the adjust layer to clear the current profile's pairing

## Editing the Keymap

### Option 1: Keymap Editor (visual)

1. Go to [nickcoutsos keymap-editor](https://nickcoutsos.github.io/keymap-editor/)
2. Connect your GitHub account and select this repository
3. Edit keys visually — the `info.json` file provides the layout
4. Save — the editor commits changes to `config/dactyl.keymap` automatically
5. GitHub Actions builds the new firmware — download and flash

### Option 2: Edit manually

Edit `config/dactyl.keymap` directly. Each layer has 78 bindings in this order:

```
Row 0: 7 left + 5-way + 7 right = 15
Row 1: 7 left + 5-way + 7 right = 15
Row 2: 7 left + 5-way + 7 right = 15
Row 3: 6 left + 5-way + 6 right = 13
Row 4: 4 left + 5-way + 4 right = 9
Row 5: 5 left + 5 right          = 10
Row 6: EC11 push button          = 1
Total:                              78
```

Push to GitHub, wait for the Actions build, download and flash.

## Notes

- The 5-way switch physical-to-row mapping: UP=row2, DOWN=row0, LEFT=row4, RIGHT=row3, CENTER=row1
- Right side thumb cluster columns are wired in non-sequential order (verified via diagnostic keymap)
- The EC11 push button uses `kscan-gpio-direct` with polling mode (`CONFIG_ZMK_KSCAN_DIRECT_POLLING=y`)
