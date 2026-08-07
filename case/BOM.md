# BOM — Temper with Custom Case

Bill of materials for building the custom temper split keyboard with the case in
[`case/`](case/readme.md). Quantities are for the **full build (both halves)**.

> **Case hardware sizing note:** Sizes below are based on the [issue thread](https://github.com/AdrianBonpin/temper-case-and-config/issues/1) — the original build used M2, and a builder confirmed that **M2 is too wide and 6 mm is too short**, successfully using **M1.6 × 7 mm** instead. Always test-fit one screw on a test print, since FDM printers shrink holes slightly.

## 1. 3D-printed case (2× of each, one per half)

| Part | File | Qty | Notes |
|------|------|-----|-------|
| Bottom case | `case/stl/bottom.stl` | 2 | 116 × 95 × 11.9 mm; fits 350 mAh battery (40 × 20 × 6 mm ±2 mm), built-in tilt, rubber-feet space |
| Top plate (tested) | `case/stl/top_old.stl` | 2 | 4.8 mm thick, type-C cutout |
| Top plate (switch version, **untested**) | `case/stl/top_w-switch.stl` | 2 | thicker (13.4 mm), dedicated power-switch cutout |

Recommended print settings: 0.2 mm layers, ~20–30 % infill, 3–4 perimeters.
Screw holes may need a quick ream after printing.

## 2. Case hardware — 5 screws + 5 nuts per half (10 + 10 total)

| Part | Spec | Qty | Notes |
|------|------|-----|-------|
| Screws | **M1.6 × 7 mm machine screws** | 10 | Verified by a builder (M2 was too wide, 6 mm was too short). Use a pan/button head — the holes are small and flush mounting is tight. |
| Nuts | **M1.6 hex nuts** (3.2 mm across flats) | 10 | Drop into the pockets in the bottom case. |
| Alt. screws | M2 × 8 mm | 10 | What the original build used — only fits if your print keeps the holes loose enough. |

## 3. Electronics

| Part | Qty | Notes |
|------|-----|-------|
| nice!nano v2 | 2 | One per half (central = left) |
| 0.91" 128×32 SSD1306 OLED (I2C, 4-pin) | 2 | Wired to the Pro Micro I2C pins per the `oled_adapter_pro_micro_128x32` shield |
| LiPo battery 350 mAh (40 × 20 × 6 mm ±2 mm) | 2 | With JST-PH 2.0 connector |
| Temper PCBs (36-key halves) | 2 | From the original temper design |

## 4. Keys & switchgear (36 keys per half)

| Part | Qty | Notes |
|------|-----|-------|
| Kailh Choc v1 switches | 72 | Low-profile |
| Choc hotswap sockets | 72 | Kailh CPG1350 |
| 1N4148 diodes | 72 | Required by the `col2row` diode matrix (SMD pre-soldered if using the official PCB) |
| Choc keycaps (1U) | 72 | |

## 5. Tools / consumables

- Soldering iron + flux, flush cutters, tweezers
- 4-pin wires (or JST-SH) to connect OLEDs to the controllers
- 2.0 mm hex key or driver for M4 screws (or a Philips driver if using socket-head)
- USB-C cables for flashing (`.uf2` from the [latest release](https://github.com/AdrianBonpin/temper-case-and-config/releases))

## Build notes

- Left half is the central (BT host); flash `LEFT…uf2` to it and `RIGHT…uf2` to the right half.
- Flash `RESET…uf2` to either half to clear BLE pairings/settings.
- ZMK Studio is enabled (left half): edit your keymap live from the web.
- The top-with-switch variant is still untested — see `case/readme.md`.