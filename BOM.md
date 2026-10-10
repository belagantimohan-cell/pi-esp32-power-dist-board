# Bill of Materials (Preliminary)

This BOM is a practical starting point for the prototype. Values and part numbers may change after final schematic simulation and PCB thermal testing.

## 1. Input protection

| Qty | Reference | Part | Value / Rating | Notes |
|-----|-----------|------|----------------|-------|
| 1 | F1 | Automotive fuse | 20A slow-blow | 24V vehicle input protection |
| 1 | F2 | USB-C inline fuse | 5A fast-blow or resettable | USB-C input protection |
| 1 | D1 | Schottky diode | 60V, 30A | Reverse polarity protection for 24V |
| 1 | TVS1 | TVS diode | SMDJ30CA | Clamp vehicle transients |
| 1 | USB_TVS | ESD/TVS array | PRTR5V0U2X | Protect USB-C VBUS and data lines |
| 2 | C_IN_24V | Bulk capacitor | 100uF, 50V | Input smoothing |
| 2 | C_IN_USB | Ceramic cap | 10uF, 10V | USB-C input filtering |

## 2. Power conversion

| Qty | Reference | Part | Value / Rating | Notes |
|-----|-----------|------|----------------|-------|
| 1 | U1 | Buck regulator | 24V -> 5V, 8A | Raspberry Pi rail |
| 1 | U2 | Buck regulator | 24V -> 3.3V, 4-5A | ESP32 rail |
| 1 | U3 | Optional LDO | 3.3V, 1A | Backup / secondary generation |
| 4 | U4-U7 | LDO per ESP32 | 3.3V, 1A | Recommended for isolation |
| 1 | L1 | Inductor | 4.7uH | Buck regulator output |
| 1 | L2 | Inductor | 4.7uH | Buck regulator output |
| 2 | FB1, FB2 | Ferrite bead | 600 ohm @ 100MHz | EMI suppression |

## 3. 5V rail to Raspberry Pi

| Qty | Reference | Part | Value / Rating | Notes |
|-----|-----------|------|----------------|-------|
| 1 | F3 | Resettable fuse | 10A PTC | 5V rail protection |
| 1 | C5V1 | Bulk cap | 100uF, 10V | Output capacitance |
| 2 | C5V2, C5V3 | Ceramic cap | 10uF, 10V | Local bus decoupling |
| 1 | J_RPI | Output connector | 5V header / barrel / XT30 | To Raspberry Pi |

## 4. 3.3V rail to ESP32-S3

| Qty | Reference | Part | Value / Rating | Notes |
|-----|-----------|------|----------------|-------|
| 1 | F4 | Resettable fuse | 5A PTC | 3.3V rail protection |
| 4 | F5-F8 | Optional per-module fuse | 1A PTC | Fault isolation per ESP32 |
| 1 | C3V3_1 | Bulk cap | 47uF, 10V | 3.3V converter output |
| 4 | C3V3_2 | Bulk cap | 10uF, 10V | Local filtering per module |
| 4 | C3V3_3 | Ceramic cap | 1uF, 10V | HF decoupling |
| 4 | J_ESP32_1-4 | Output connector | JST / header | To ESP32 modules |

## 5. Monitoring and control (optional)

| Qty | Reference | Part | Value / Rating | Notes |
|-----|-----------|------|----------------|-------|
| 1 | U_MON1 | Current monitor | INA219 | 5V or 3.3V current measurement |
| 1 | U_TEMP | Temp sensor | TMP36 | thermal monitoring |
| 1 | COMP1 | Voltage comparator | LM393 | under-voltage alarm |

## 6. Example manufacturer suggestions

- TI: TPS54160, LMZM33602, LM1117, TL1963A
- Infineon: MOSFET reverse polarity solutions
- Littelfuse: automotive fuse, TVS, PPTC resettable fuses
- Diodes Inc.: PRTR5V0U2X, Schottky diodes
- Murata / TDK / Samsung: MLCC capacitors
- Vishay / Bourns: ferrite beads and resistors

## 7. Recommended prototype BOM priority

### Must-have for first prototype
- 24V main fuse
- USB-C input fuse
- 24V reverse polarity protection diode or MOSFET
- 30V TVS diode
- 5V buck converter
- 3.3V buck converter
- 4x ESP32-S3 3.3V output filtering capacitors
- 5V Pi output fuse
- 3.3V rail fuse

### Optional but strongly recommended
- ESD array on USB-C
- per-module 3.3V LDOs
- current sense IC
- thermal sensor

## 8. Important BOM notes
- Raspberry Pi current can exceed 5V 3A easily during heavy workloads; use 5V rail design margin with 5A+ headroom.
- ESP32-S3 modules may show current spikes during Wi-Fi transmission; design the 3.3V rail with local decoupling and adequate transient headroom.
- Automotive vehicle input should be designed for transient spikes, not only nominal 24V.

## 9. Prototype acceptance checklist

- All fuses installed and labeled
- Reverse polarity protection tested
- TVS tested with pulse injection
- 5V rail stable at Pi load
- 3.3V rail stable at ESP32 peak demand
- No thermal runaway at full board load
- Grounding and return paths verified with continuity testing
