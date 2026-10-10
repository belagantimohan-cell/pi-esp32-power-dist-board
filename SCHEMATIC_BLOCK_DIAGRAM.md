# Schematic Block Diagram

This is a block-level schematic concept for the board. It is intentionally simplified and suitable for KiCad or Altium conversion.

## Power architecture overview

```text
                                  ┌──────────────────────┐
                                  │ USB-C Input          │
                                  │ 5V / 9V / 12V PD     │
                                  └──────────┬───────────┘
                                             │
                                             │ 5A fuse
                                             │
                                             v
                                  ┌──────────────────────┐
                                  │ USB-C ESD/TVS Filter │
                                  └──────────┬───────────┘
                                             │
                                             └──────────────┐
                                                            │
                                                            v
                                     ┌──────────────────────────────────┐
                                     │ Input Selection / OR-ing         │
                                     │ (Diode-OR or MOSFET mux)         │
                                     └──────────────┬───────────────────┘
                                                    │
                                                    v
                                     ┌──────────────────────────────────┐
                                     │ Main Power Bus (24V / 5V rail)   │
                                     └───────┬───────────────┬─────────┘
                                             │               │
                                             │               │
                    ┌────────────────────┘               └──────────────────────┐
                    v                                                            v
      ┌──────────────────────────────┐                             ┌──────────────────────────────┐
      │ 24V Vehicle Input            │                             │ USB-C Input                  │
      │ 24V from vehicle battery     │                             │ 5V/9V/12V PD source         │
      └──────────────┬───────────────┘                             └──────────────┬───────────────┘
                     │                                                     │
                     │ 20A fuse                                          │ 5A fuse
                     │                                                     │
                     v                                                     v
      ┌──────────────────────────────┐                            ┌──────────────────────────────┐
      │ Reverse Polarity Protection │                            │ ESD/TVS Protection          │
      │ Schottky or MOSFET          │                            │ USB lines and VBUS          │
      └──────────────┬───────────────┘                            └──────────────┬───────────────┘
                     │                                                     │
                     v                                                     v
      ┌──────────────────────────────┐                            ┌──────────────────────────────┐
      │ TVS Clamp / Bulk Cap        │                            │ Power Selection Mux         │
      │ 30V TVS + 100uF + 10uF      │                            │ Diode-OR / PFET            │
      └──────────────┬───────────────┘                            └──────────────┬───────────────┘
                     │                                                     │
                     └───────────────────────┬─────────────────────────────┘
                                             │
                                             v
                            ┌─────────────────────────────────────┐
                            │ Main regulated input rail           │
                            │ 24V bus to buck converters          │
                            └──────────────┬──────────────────────┘
                                           │
                         ┌───────────────────────────────┐
                         │                               │
                         v                               v
           ┌───────────────────────────┐   ┌─────────────────────────────┐
           │ 24V -> 5V Buck Converter  │   │ 24V -> 3.3V Buck Converter  │
           │ TPS54160 / equivalent     │   │ TPS54160 / equivalent       │
           │ Output: 5V, 8A           │   │ Output: 3.3V, 4-5A         │
           └──────────────┬────────────┘   └──────────────┬──────────────┘
                          │                                  │
                          │                                  │
                          v                                  v
            ┌───────────────────────────┐      ┌──────────────────────────────┐
            │ 5V Rail Protection        │      │ 3.3V Rail Protection        │
            │ Fuse + bulk caps + bead  │      │ Fuse + bulk caps + bead     │
            └──────────────┬────────────┘      └──────────────┬───────────────┘
                           │                                    │
                           v                                    v
               ┌───────────────────┐                ┌───────────────────────────┐
               │ Raspberry Pi Rail │                │ ESP32-S3 Rail            │
               │ 5V to Pi          │                │ 3.3V to 4 ESP32s         │
               └─────────┬─────────┘                └──────────┬──────────────┘
                         │                                            │
                         │                                            │
                         v                                            v
              ┌──────────────────┐                       ┌─────────────────────────┐
              │ Raspberry Pi      │                       │ ESP32-S3 Module 1      │
              │ 5V, 5A max       │                       │ 3.3V, ~500mA + peaks  │
              └──────────────────┘                       └─────────────────────────┘
                                                               │
                                                              ...
                                                               │
                                               ┌─────────────────────────┐
                                               │ ESP32-S3 Module 4      │
                                               │ 3.3V, ~500mA + peaks  │
                                               └─────────────────────────┘
```

## Functional notes

### 1. Input protection
- 24V vehicle input must include:
  - main fuse near connector
  - reverse polarity protection
  - TVS clamp
  - input bulk capacitance
- USB-C input must include:
  - current-limited fuse
  - ESD protection array
  - overvoltage clamp

### 2. Power muxing
- Recommended approach: diode-OR for simplicity and robustness.
- Alternative: PFET-based ideal OR-ing for reduced voltage drop.

### 3. Regulated rails
- 5V rail should directly feed the Raspberry Pi
- 3.3V rail should feed the ESP32-S3 modules
- Use separate local bypass capacitors near each module

### 4. Optional enhancement
- Add a Pi current monitor (INA219) and 3.3V monitor (INA219 or voltage comparator)
- Add a power-good signal to the Pi GPIO or MCU

## Suggested schematic blocks to create in KiCad

### Part blocks
- `24V_INPUT_FUSE`
- `REVERSE_POLARITY_PROTECTION`
- `TVS_CLAMP`
- `USB_C_5V_INPUT`
- `USB_ESD_PROTECTION`
- `POWER_MUX`
- `BUCK_5V`
- `BUCK_3V3`
- `5V_OUTPUT_FUSE`
- `3V3_OUTPUT_FUSE`
- `RPI_5V_LOAD`
- `ESP32_3V3_LOAD_1`
- `ESP32_3V3_LOAD_2`
- `ESP32_3V3_LOAD_3`
- `ESP32_3V3_LOAD_4`

## Recommended next design step
Create the KiCad schematic with four main sections:
1. 24V vehicle input protection
2. USB-C input and OR-ing
3. 5V conversion to Raspberry Pi
4. 3.3V conversion to four ESP32-S3 modules

A final design review should check:
- thermal behavior under full load
- transient suppression under automotive conditions
- input source switching behavior
- voltage drop on the Pi rail
- current spikes on each ESP32 rail
