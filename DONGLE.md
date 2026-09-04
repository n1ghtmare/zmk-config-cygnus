# Prospector dongle

Dongle mode makes the Prospector (Seeed XIAO nRF52840 + 1.69" ST7789 display)
the **central**: it holds the keymap, connects to the host, and both keyboard
halves become peripherals that only report key positions.

The normal (no dongle) firmware is still built, so you can switch back at any
time by reflashing the halves.

## What is in this repo

| File | Purpose |
| --- | --- |
| `config/boards/shields/cygnus/cygnus_layout.dtsi` | Matrix transform + physical layout, shared by both halves and the dongle |
| `config/boards/shields/cygnus_dongle/` | The dongle shield: mock kscan, central role, 2 peripherals |
| `config/cygnus_dongle.conf` | Dongle-only settings (Prospector brightness, no sleep) |
| `config/cygnus.keymap` | Unchanged. The dongle build picks up this same file |
| `config/west.yml` | Adds the `prospector-zmk-module` dependency |

The keymap is **not** duplicated. When ZMK looks for keymap and `.conf` files
it also tries the shield name with its last `_`-separated suffix removed, so
`cygnus_dongle` resolves to the same `config/cygnus.keymap` and
`config/cygnus.conf` that `cygnus_left` and `cygnus_right` already use.
`config/cygnus_dongle.conf` is merged on top of that.

## Artifacts

Dongle mode:

- `cygnus_dongle` / `cygnus_dongle_with_studio` -> the Prospector
- `cygnus_left_peripheral` -> left half
- `cygnus_right_peripheral` -> right half

Non-dongle mode (unchanged): `cygnus_left`, `cygnus_right`, and the
`*_with_studio` variants.

## Flashing procedure

> **Read this first if you have ever edited the keymap in ZMK Studio.**
> Studio does not write to this repo. It saves bindings into the settings
> (NVS) flash of whichever device is **central** -- today that is the left
> half -- under `keymap/l/<layer>/<position>` keys, and those override the
> compiled-in keymap. The migration below wipes them twice over: step 1
> erases the settings partition, and the dongle becomes the new central with
> an empty one, so it boots whatever is in `config/cygnus.keymap`.
> ZMK Studio has no export feature yet (zmk-studio issue #124), so copy your
> layers into `config/cygnus.keymap` by hand **before** flashing anything.

Order matters, because all three devices hold stale bonding information.

1. Flash `settings_reset` to **all three** devices
   (`cygnus_dongle_settings_reset` for the Prospector,
   `settings_reset` for both halves).
2. Flash `cygnus_dongle_with_studio` to the Prospector and plug it into USB.
3. Flash `cygnus_left_peripheral` to the left half. Wait for it to appear on
   the dongle display.
4. Flash `cygnus_right_peripheral` to the right half.

Pair the left half first, then the right one. The dongle assigns peripheral
slots in the order they connect, and the display shows them in that order.

To go back to non-dongle mode, run a `settings_reset` on both halves again and
flash the plain `cygnus_left` / `cygnus_right` firmware.

## ZMK Studio

Studio talks to whichever device is central, so with the dongle plugged in over
USB you connect Studio to the **dongle**, not to the left half. The
`&studio_unlock` binding on the lower layer works the same way.

Keymap edits now live on the dongle only. The halves no longer need reflashing
when the keymap changes -- only when the matrix or physical layout changes.

## Hardware note

The beekeeb pre-soldered Prospector ships without the APDS9960 ambient light
sensor, so `config/cygnus_dongle.conf` sets a fixed backlight brightness:

```
CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=n
CONFIG_PROSPECTOR_FIXED_BRIGHTNESS=80
```

If the display is upside down in your case, set
`CONFIG_PROSPECTOR_ROTATE_DISPLAY_180=y` in the same file.
