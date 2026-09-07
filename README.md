# PS Access controller web tool

`ps-access-config` is a browser-based configuration tool for the PlayStation Access controller. It connects directly over USB through WebHID, loads all three on-device profiles, lets you edit supported button and expansion-port mappings, and writes changes back while preserving settings the editor does not expose.

Use Chrome or Edge on desktop. Bluetooth is intentionally unsupported because profile feature reports are available only over USB.

## Local testing

Run the app from a local web server, then open the listed localhost address in a WebHID-capable browser:

```sh
npx --yes serve -l 4173 .
```

Connect the controller by USB and select it through the browser permission dialog.

## Mapping parameters

The UI stores a canonical PlayStation action name. The **Action style** selector only changes the displayed label; it does not change the byte written to the controller.

| Canonical action | Controller value |
| --- | ---: |
| Nothing | 0 |
| Circle | 1 |
| Cross | 2 |
| Triangle | 3 |
| Square | 4 |
| D-pad up/down/left/right | 5–8 |
| L1 / R1 | 9 / 10 |
| L2 / R2 | 11 / 12 |
| L3 / R3 | 13 / 14 |
| Options / Create | 15 / 16 |
| PS button | 17 |
| Touch pad | 18 |
| Left stick / Right stick | 101 / 102 |

## Android action style mapping

This display style is a label translation only. It never changes the controller value in the table above.

| PlayStation action | Android label |
| --- | --- |
| Triangle | X |
| Circle | A |
| Cross | B |
| L1 | Y |
| L2 | LB |
| R2 | RB |
| L3 | Select |
| R3 | Select |
| Options | RT |
| Create | LT |
| PS button | Home |
| Touch pad | L3 |

The controller retains its own profile format. Before saving, the app requires a successful load and modifies only the fields exposed by the UI. Other controller settings are kept from the loaded profile.

## Physical-control mapping

Every profile exposes these controls. `B` values identify the controller’s button slot; the primary assigned action is stored at `100 + (B - 1) × 5`. Its toggle flag is bit `B - 1` in the two-byte toggle field beginning at byte `150`.

| UI control | Controller slot | Primary-action byte | Toggle bit |
| --- | --- | ---: | ---: |
| Stick press in | B10 | 145 | 9 |
| Center button | B9 | 140 | 8 |
| Button 1–8 | B1–B8 | 100, 105, 110, 115, 120, 125, 130, 135 | 0–7 |

The analog stick and expansion ports use 45-byte records, beginning at `152 + port × 45`:

| UI control | Port | Record start | Toggle bit | Notes |
| --- | ---: | ---: | ---: | --- |
| Stick | E0 | 152 | 9 | Left/right stick assignment and orientation |
| Expansion port 1 | E1 | 197 | 10 | Button or analog input |
| Expansion port 2 | E2 | 242 | 11 | Button or analog input |
| Expansion port 3 | E3 | 287 | 12 | Button or analog input |
| Expansion port 4 | E4 | 332 | 13 | Button or analog input |

For an expansion-port record, byte `0` is the type (`0x01` stick, `0x02` analog button, `0x03` digital button); byte `2` is the primary button action. The app presents toggles for all exposed controls and analog mode for expansion ports 1–4.

## Hard-coded controller protocol IDs

| Constant | Value | Purpose |
| --- | ---: | --- |
| Sony vendor ID | `0x054c` | WebHID device filter |
| Access controller product ID | `0x0e5f` | WebHID device filter |
| Feature report (send) | `0x60` | Profile read/write request packets |
| Feature report (receive) | `0x61` | Profile read response packets |
| Bluetooth report indicator | `0x63` | Used to reject Bluetooth connections |
| Profile payload size | `956` bytes | Complete stored profile record |
| Profile packet size | `56` bytes | Data portion in each report packet |
| Packets per profile | `18` | Packets required for a complete profile |
| Read command | `0x10 + profileIndex` | Requests profile 1–3 (`profileIndex` is 0–2) |
| Save command | `0x09 + profileIndex` | Writes profile 1–3 (`profileIndex` is 0–2) |

The save operation writes a standard IEEE CRC-32 checksum in the final packet. Profile button mappings begin at byte `100`, toggle flags at byte `150`, expansion-port settings at byte `152`, and the saved timestamp at byte `948`.
