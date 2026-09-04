ZMK Studio Integration

Overview
- This repository contains keymaps and shield overlays for Cygnus. This document explains how to enable ZMK Studio support (live keymap editing) and how to import Studio exports into this repo.

Enable ZMK Studio in builds
- The firmware must be built with ZMK Studio enabled. Two ways:
  - GitHub Actions: this repo's `build.yaml` includes example entries that add the `studio-rpc-usb-uart` snippet and set CMake args `-DCONFIG_ZMK_STUDIO=y` for studio-enabled artifacts.
  - Local build: pass the CMake argument when building, e.g. add `-DCONFIG_ZMK_STUDIO=y` to your build invocation or use the same snippet used in CI.

Notes
- The `cygnus.zmk.yml` feature list already contains `studio` to mark shields that support studio.
- For full Studio functionality you must include the "Studio Unlock" behavior in your keymap (see ZMK Studio docs). This repository does not automatically modify your keymap bindings; you must add the unlock binding to a key or combo as documented by ZMK Studio.

Importing ZMK Studio exports
- If ZMK Studio can export a DTS/keymap file, use the helper script to import that file into the repo.

Usage (from repo root):

```bash
python scripts/import_studio_export.py path/to/studio_export.dtsi
```

- By default the script will write into `config/boards/shields/cygnus/cygnus.keymap` if that shield exists, otherwise `config/cygnus.keymap`.
- The script is conservative: it safely copies DTS-style keymap files or attempts to extract a `dts`/`keymap` field from a JSON export. If automatic extraction fails, it saves a parsed JSON copy under `scripts/imports/` for manual inspection.

Dongle mode
- With the Prospector dongle attached, the dongle is the central and holds the keymap, so Studio must be connected to the *dongle*, not to the left half. Build `cygnus_dongle_with_studio`. See `DONGLE.md`.

How to use ZMK Studio
1. Build and flash a firmware that was built with Studio support enabled.
2. Open https://zmk.studio/ in your browser.
3. Connect the keyboard over USB and follow the Studio prompts (unlock behavior required).
4. Use Studio to edit layers. For persistent changes in this repo, export a DTS/keymap from Studio and import it with the script above.

Troubleshooting and tips
- If Studio cannot connect, ensure firmware was built with `-DCONFIG_ZMK_STUDIO=y` and that the `studio-rpc-usb-uart` snippet is present in the build.
- If you want CI artifacts that include Studio support, the `build.yaml` entries now include example studio artifacts for `cygnus_left` and `cygnus_right`.

References
- ZMK Studio: https://zmk.studio/
- ZMK docs: https://zmk.dev/

If you want, I can try to add an automated mapping step to convert Studio's JSON layer layouts into this keymap layout — tell me which Studio export format you use and I will extend the import script accordingly.
