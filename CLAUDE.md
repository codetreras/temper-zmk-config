# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a ZMK firmware configuration for the Temper keyboard - a 36-key split mechanical keyboard with 10-column × 4-row matrix (3 rows of 5 keys per hand plus 3 thumb keys). The keyboard uses nice!nano v2 controllers and includes OLED display support.

## Build System

**Building firmware:**
- Firmware is built automatically via GitHub Actions on push/PR
- No local build setup required - download compiled .uf2 files from GitHub Actions artifacts
- The workflow uses ZMK's official build action: `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`

**Build configuration:**
- `build.yaml` defines the build matrix (nice_nano_v2 board with temper_left/temper_right shields)
- Builds produce separate firmware files for left and right halves plus settings_reset

**Triggering builds:**
- Push to any branch triggers a build
- Manual workflow dispatch is also available

## Architecture

**Shield Definition** (`boards/shields/temper/`):
- `temper.dtsi` - Base device tree with matrix transform, kscan configuration, and OLED setup
- `temper_left.overlay` / `temper_right.overlay` - Per-side GPIO pin definitions for column scanning
- `temper.keymap` - Keymap configuration with layer definitions and custom behaviors
- `Kconfig.shield` - Kconfig definitions for left/right shield variants
- `temper.zmk.yml` - Shield metadata (type, features, siblings)

**Matrix Configuration:**
- 4×5 matrix per side (10 columns × 4 rows total)
- COL2ROW diode direction
- Row GPIOs: pro_micro pins 4, 5, 6, 7
- Column GPIOs (left): pro_micro pins 21, 20, 19, 18, 15
- OLED on I2C bus (SSD1306 @ 0x3c)

**Keymap Layers** (`temper.keymap`):
- Layer 0 (QWERTY): Base alphanumeric layer
- Layer 1 (NAV): Navigation with vim-style arrows (H/J/K/L), media controls, window management
- Layer 2 (SYM): Symbol layer with brackets, operators, special characters
- Layer 3 (NUM): Number row and function keys (conditional layer - activates when NAV + SYM held)

**Custom Behaviors:**
- `super_bspc`: Mod-morph that becomes Option+Backspace when shift is held
- `tilde_n`: Macro for typing ñ character
- `swap_app`: Macro for Cmd+Tab app switching
- `swap_window`: Macro for Cmd+` window switching

**West Configuration** (`config/west.yml`):
- Pulls ZMK from `zmkfirmware/zmk` main branch
- Imports ZMK's app/west.yml for dependencies

## Keymap Visualization

**Automatic generation:**
- `.github/workflows/doc.yml` generates keymap SVG on changes to .keymap or .dtsi files
- Uses caksoylar/keymap-drawer action
- Auto-commits generated images to keymap_img/temper.svg
- Parses with `-c 10` flag (10 columns)
- Draws with chocofi layout and layer names: qwerty, nav, sym, num

**Configuration:**
- `keymap_img/keymap_drawer.config.yaml` - Custom styling and key symbol mappings
- Defines visual representations for custom behaviors (e.g., &tilde_n → ñ, &super_bspc → ⌫/⎇⌫)
- Custom SVG styling using monospace font and GitHub-like colors

## Making Keymap Changes

1. Edit `boards/shields/temper/temper.keymap`
2. Commit and push changes
3. GitHub Actions will build the firmware and generate updated keymap visualization
4. Download .uf2 files from Actions artifacts (temper_left, temper_right)
5. Flash each half by copying .uf2 to the mounted keyboard drive

## Common Modifications

**Changing key bindings:**
- Edit the `bindings` array in the relevant layer within `temper.keymap`
- Use ZMK keycodes (e.g., `&kp A`, `&mo NAV`, `&sk LSHFT`)

**Adding new macros:**
- Define in the `macros` section using `zmk,behavior-macro`
- Reference with `&macro_name` in keymap

**Adding new layers:**
- Define layer constant (e.g., `#define NEWLAYER 4`)
- Add layer to keymap with corresponding bindings array
- Use `&mo NEWLAYER` or `&to NEWLAYER` to activate

**Adjusting OLED settings:**
- Modify the `oled: ssd1306@3c` node in `temper.dtsi`

## File Organization

- `boards/shields/temper/` - All keyboard-specific configuration
- `config/` - West manifest for ZMK dependencies
- `keymap_img/` - Generated keymap visualizations and drawer config
- `.github/workflows/` - CI/CD for building firmware and documentation

## Notes

- This is a user config repository, not a ZMK fork - shield definitions are included in-tree
- The keyboard is compatible with nice!nano and nice!nano v2 boards
- Board-specific overlays in `boards/shields/temper/boards/` handle differences between controller versions
- Changes to the keymap trigger both firmware builds and documentation updates
