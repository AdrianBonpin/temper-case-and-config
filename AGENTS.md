# AGENTS.md - AI Agent Guidelines for temper-case-and-config

This document provides guidelines for AI coding agents working in this ZMK keyboard firmware configuration repository.

## Project Overview

This is a **ZMK firmware configuration** for the "temper" split keyboard (based on the Chocofi design). It is NOT a traditional software project - there is no source code to compile locally. Instead, configuration files define the keyboard's behavior, and builds happen via GitHub Actions.

### Key Characteristics
- **Framework**: ZMK (Zephyr Mechanical Keyboard)
- **Target Hardware**: nice!nano v2 microcontroller with SSD1306 OLED
- **Build System**: GitHub Actions (cloud-based)
- **Primary Languages**: Devicetree, Kconfig, ZMK Keymap DSL

---

## Build Commands

### Primary Build Method (GitHub Actions)
Builds are triggered automatically on push/PR via `.github/workflows/build.yml`.

```bash
# There is no local build command - push to GitHub to trigger builds
git push origin main
```

The workflow uses ZMK's official reusable workflow and produces firmware artifacts (.uf2 files) downloadable from the Actions tab.

### Local Build (Advanced - Requires Zephyr Toolchain)
If you have the Zephyr SDK and west installed:

```bash
# Initialize west workspace (run once)
west init -l config
west update

# Build left half
west build -s zmk/app -b nice_nano_v2 -- -DSHIELD="temper_left oled_adapter_pro_micro_128x32 nice_oled" -DZMK_CONFIG="${PWD}/config"

# Build right half
west build -s zmk/app -b nice_nano_v2 -- -DSHIELD="temper_right oled_adapter_pro_micro_128x32 nice_oled" -DZMK_CONFIG="${PWD}/config"
```

### Testing/Validation
There are no unit tests. Validation is done by:
1. Successful GitHub Actions build (check for errors in workflow logs)
2. Flashing firmware to keyboard and testing physically
3. Using ZMK Studio for live keymap editing (if enabled)

---

## Repository Structure

```
├── config/
│   ├── west.yml          # West manifest - external module dependencies
│   ├── temper.conf       # User Kconfig overrides (OLED, features)
│   ├── temper.keymap     # Keymap definitions
│   └── info.json         # Physical layout metadata for editors
├── boards/shields/temper/
│   ├── temper.dtsi       # Base devicetree (matrix, display, layout)
│   ├── temper_left.overlay   # Left half pin mappings
│   ├── temper_right.overlay  # Right half pin mappings
│   ├── temper.conf       # Shield-level Kconfig (BT, power, display)
│   ├── Kconfig.shield    # Shield detection config
│   ├── Kconfig.defconfig # Default config when shield is selected
│   └── boards/           # Board-specific overlays (underglow)
├── build.yaml            # GitHub Actions build matrix
└── zephyr/module.yml     # Zephyr module registration
```

---

## Code Style Guidelines

### Devicetree Files (.dtsi, .overlay)

```c
/* Use C-style block comments for file headers */
/* SPDX-License-Identifier: MIT */

#include <physical_layouts.dtsi>  // Angle brackets for system includes

/ {
    node_name: node_label {
        compatible = "vendor,device";
        property = <value>;
    };
};

// Reference nodes with &node_label syntax
&existing_node {
    status = "okay";
};
```

**Conventions:**
- Use lowercase with underscores for node names: `matrix_transform0`
- Use 4-space indentation
- Align property values in related blocks
- Include SPDX license header in new files

### Kconfig Files (.conf)

```ini
# Section comment explaining the group
CONFIG_FEATURE_NAME=y          # Enable boolean
CONFIG_FEATURE_NAME=n          # Disable boolean
CONFIG_NUMERIC_VALUE=1000      # Integer value
CONFIG_STRING_VALUE="text"     # String value (rare)
```

**Conventions:**
- Group related configs with comment headers
- Use `y`/`n` for booleans, not `true`/`false`
- Add inline comments for non-obvious settings
- Prefix comments with `##` for documentation vs `#` for commented-out code

### Keymap Files (.keymap)

```c
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>

/ {
    keymap {
        compatible = "zmk,keymap";

        layer_name {
            display-name = "ABC";  // Max 3 chars for OLED
            bindings = <
                &kp Q  &kp W  &kp E  &kp R  &kp T    &kp Y  &kp U  &kp I  &kp O  &kp P
            >;
        };
    };
};
```

**Conventions:**
- Use UPPERCASE for key codes: `&kp LEFT_SHIFT`
- Align columns in binding matrices for readability
- Layer names should be descriptive: `BASE`, `NAV`, `NUM`, `FUNC`
- `display-name` is limited to 3 characters for OLED display

### YAML Files (west.yml, build.yaml)

```yaml
manifest:
  remotes:
    - name: remote_name
      url-base: https://github.com/username
  projects:
    - name: project-name
      remote: remote_name
      revision: main  # or specific commit SHA
```

**Conventions:**
- Use 2-space indentation
- Use lowercase with hyphens for keys
- Pin to specific commits for reproducible builds (recommended)

---

## Common Tasks

### Adding/Modifying Keys
Edit `config/temper.keymap`. Use ZMK key codes from:
https://zmk.dev/docs/keymaps/list-of-keycodes

### Changing Bluetooth Settings
Edit `boards/shields/temper/temper.conf`:
- `CONFIG_ZMK_KEYBOARD_NAME` - Bluetooth device name
- `CONFIG_BT_MAX_CONN` - Max simultaneous connections

### Modifying OLED Animation
Edit `config/temper.conf`:
- `CONFIG_NICE_OLED_GEM_ANIMATION=y/n` - Enable/disable
- `CONFIG_NICE_OLED_GEM_ANIMATION_MS` - Animation speed (lower = faster)

### Adding External Modules
Edit `config/west.yml` to add new remotes and projects.

---

## External Dependencies

This repo pulls external modules via the west manifest:

| Module | Purpose | Source |
|--------|---------|--------|
| `zmk` | Core ZMK firmware | zmkfirmware/zmk |
| `zmk-oled-adapter` | OLED display adapter | mctechnology17 |
| `zmk-nice-oled` | OLED widgets & animations | mctechnology17 |

To use a fork of any module, update the `remote` in `config/west.yml`.

---

## Error Handling & Troubleshooting

### Common Build Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `undefined reference` | Missing include | Add required `#include` |
| `node not found` | Invalid node reference | Check `&node_name` spelling |
| `Kconfig warning` | Invalid config option | Check option exists in module |

### Debugging Tips
1. Check GitHub Actions logs for detailed error messages
2. Validate devicetree syntax matches ZMK documentation
3. Ensure `west.yml` module revisions are compatible

---

## Important Notes for Agents

1. **No Local Execution**: Do not attempt to run build commands locally unless specifically asked
2. **Push to Build**: Changes are validated by pushing to GitHub and checking Actions
3. **Shield vs Config**: Shield files define hardware; config files define behavior
4. **Split Keyboard**: Left and right halves build separately - changes may need both
5. **OLED Code is External**: Animation logic is in `zmk-nice-oled` module, not this repo
6. **ZMK Studio**: This keyboard has Studio enabled for live configuration

---

## References

- [ZMK Documentation](https://zmk.dev/docs)
- [ZMK Keycodes](https://zmk.dev/docs/keymaps/list-of-keycodes)
- [Devicetree Specification](https://www.devicetree.org/)
- [Original Temper Design](https://github.com/raeedcho/temper)
