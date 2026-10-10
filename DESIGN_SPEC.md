# Power Distribution Board Design Specification
## Raspberry Pi + 4x ESP32-S3 from 24V Vehicle/Power Bank Input

---

## 1. System Overview

**Purpose:** Centralized power distribution for:
- 1x Raspberry Pi (5V, ~5A)
- 4x ESP32-S3 modules (3.3V, ~500mA each, peak 1.5A per module)
- Input: 24V vehicle battery / 5V USB-C power bank / hybrid support

**Total Power Budget:**
- 5V rail: ~7-8A (Pi + ESP32 overhead)
- 3.3V rail: ~2-4A (4x ESP32-S3 @ 500mA nominal + spikes)
- Protection overhead: fuses, converters, margin

---

## 2. Input Sources & Fusing Strategy

### 2.1 Vehicle 24V Input
- **Voltage Range:** 18V - 32V (cold crank to load-dump)
- **Connector:** XT60 or Anderson Power Pole (automotive standard)
- **Protection:**
  - **Fuse 1 (Main 24V):** 20A automotive fuse (Littelfuse ATO or equivalent)
    - Location: Immediately at 24V input connector
    - Type: Slow-blow / automotive rated
    - Purpose: Protect 24V line and all downstream converters
  - **Reverse Polarity Protection Diode:** Schottky 60V, 30A (e.g., MBR3060PT)
    - Alternative: N-channel MOSFET reverse polarity circuit if ultra-low drop needed
  - **TVS Diode (Transient Suppression):** 30V bidirectional TVS (e.g., SMDJ30CA)
    - Location: After fuse, before buck converter
    - Purpose: Suppress ISO 7637 automotive transients (load-dump, field collapse)
  - **Bulk Capacitor:** 100µF, 50V ceramic or film
    - Location: Across 24V input after fuse
    - Purpose: Smooth input ripple and transients

### 2.2 USB-C Power Bank Input
- **Voltage Range:** 5V (standard), 9V/12V (if PD capable)
- **Connector:** USB Type-C
- **Protection:**
  - **Fuse 2 (USB-C Input):** 5A automotive or cartridge fuse
    - Location: Immediately after USB-C connector
    - Type: Fast-blow (USB-C standard current limit)
    - Purpose: Protect against short circuit on USB-C line
  - **Polarity Protection:** Integrated into USB-C connector (mechanical key)
    - Optional: Add series Schottky diode (1N5819, 1A) if belt-and-suspenders needed
  - **ESD Protection:** TVS array (e.g., PRTR5V0U2X)
    - Location: On USB-C data lines (D+, D-, VBUS)
    - Purpose: Protect against static discharge

### 2.3 Power Mux / Input Selection Logic
- **Strategy:** Prioritize 24V input if present; fall back to USB-C if 24V absent
- **Implementation:**
  - **Option A (Diode-OR):** 
    - Schottky diodes on both 24V and 5V rails
    - Lower voltage path automatically blocks via reverse-bias
    - Simple, no active logic needed
    - Drawback: ~0.3V drop per diode
  - **Option B (P-channel MOSFET Mux):**
    - Sense 24V presence
    - Switch P-FET on USB-C when 24V absent
    - Cleaner, but adds complexity
  - **Recommendation:** Use Diode-OR for simplicity in a vehicle context

### 2.4 Input Fuse Summary Table

| Input Source | Fuse Rating | Type | Location | Purpose |
|--------------|-------------|------|----------|---------|
| **24V Vehicle** | 20A | Slow-blow automotive | At 24V connector | Main protection |
| **USB-C Input** | 5A | Fast-blow / PTC | At USB-C connector | USB spec compliance |
| **5V Rail (Pi)** | 10A | Resettable (PTC) | After 5V buck converter | Secondary protection |
| **3.3V Rail (ESP32)** | 5A | Resettable (PTC) | After 3.3V buck converter | Secondary protection |

---

## 3. Power Conversion Architecture

### 3.1 24V → 5V Buck Converter (Primary)
- **Part Suggestion:** TPS54160 or LMZM33602
- **Output:** 5V, 8A
- **Input Range:** 18V - 32V (automotive range)
- **Protection:**
  - Input fuse: 20A (handled above)
  - TVS on input (handled above)
  - Soft-start to avoid inrush current
  - Current limit: 12A (overcurrent shutdown)
- **Output:**
  - Bulk cap: 100µF, 10V ceramic
  - Ferrite bead + RC snubber on output for noise filtering
  - Resettable fuse (PTC) 10A on output

### 3.2 24V → 3.3V Buck Converter (Secondary)
- **Part Suggestion:** TPS54160 (same as above) or LMZM23610
- **Output:** 3.3V, 4-5A
- **Input Range:** 18V - 32V
- **Protection:**
  - Input fuse: shared with 5V buck from main 20A fuse
  - Output resettable fuse: 5A PTC
- **Output:**
  - Bulk cap: 47µF, 10V ceramic
  - Power distribution to 4x ESP32-S3 modules

### 3.3 5V → 3.3V LDO (Optional Secondary Path)
- **Part Suggestion:** LM1117-3.3 or TL1963A-33
- **Output:** 3.3V, 1A
- **Purpose:** Redundant 3.3V source if USB-C only input
- **Protection:**
  - Inline resistor + ferrite bead on input

---

## 4. Raspberry Pi Power Rail (5V)

### 4.1 Output Specifications
- **Voltage:** 5V ±5%
- **Max Current:** 7-8A
- **Ripple:** <100mV peak-peak

### 4.2 Distribution
- **Connector:** Anderson Power Pole or XT60 to Pi board
- **Filtering:** 
  - 100µF bulk cap at converter output
  - 2x 10µF ceramic caps near Pi connector
  - Ferrite bead on each trace to Pi
- **Current Monitoring (Optional):**
  - INA219 I2C current sense (Vshunt on Pi rail)
  - Allows real-time power monitoring via I2C to Raspberry Pi

### 4.3 Fuses & Protection
- **Output Fuse:** 10A resettable (PTC) PPTC0805P
- **Hot-swap Protection:** Schottky catch diode to prevent backfeed

---

## 5. ESP32-S3 Power Rail (3.3V)

### 5.1 Output Specifications
- **Voltage:** 3.3V ±10%
- **Max Current:** 2-4A (1.5A peak per ESP32 module)
- **Ripple:** <150mV peak-peak

### 5.2 Distribution Strategy

**Option A: Shared 3.3V Bus**
- Single 3.3V buck converter feeds all 4 ESP32-S3 modules
- Pros: Simpler, lower cost
- Cons: Crosstalk risk if one module draws heavy current (Bluetooth/Wi-Fi)

**Option B: Dedicated LDO per ESP32**
- 3.3V buck converter → 4x LDO (one per ESP32)
- Each LDO: TL1963A-33, 1A output
- Pros: Isolation, less crosstalk, better transient response
- Cons: More components, more board area
- **Recommendation:** Use Option B for robustness

### 5.3 Per-Module Filtering (If Option B)
- Each LDO output:
  - 10µF ceramic cap (bulk)
  - 1µF ceramic cap (fast transient response)
  - Ferrite bead on output

### 5.4 Fuses & Protection
- **Main 3.3V Fuse:** 5A resettable PTC
- **Per-module Fuse (Optional):** 1A resettable PTC on each LDO output
  - Purpose: Isolate faulty ESP32 module without shutting down others

---

## 6. Protection & Control Circuits

### 6.1 Reverse Polarity Protection (24V Input)
- **Schottky Diode:** MBR3060PT (60V, 30A)
  - Anode: 24V ground reference
  - Cathode: 24V input
  - Voltage drop: ~0.3V at 20A
- **Alternative:** P-channel MOSFET with sense circuit (if ultra-low drop needed)

### 6.2 Transient Suppression (24V Input)
- **TVS Diode:** SMDJ30CA (30V bidirectional)
  - Location: After 20A fuse, before buck converter input
  - Purpose: Clamp load-dump spikes (up to 87V pulses in vehicles)
  - Clamping voltage: ~35V (protects buck converter)

### 6.3 ESD & EMI Filtering
- **USB-C Lines:** PRTR5V0U2X TVS array
- **Power Delivery:** 
  - Ferrite bead on each input line
  - RC low-pass filter before buck converter (R=10Ω, C=100nF)

### 6.4 Soft-Start & Inrush Limiting
- **Buck Converter Soft-Start:** Enable pin with RC delay (1-2 second ramp)
  - Capacitor on SS pin: 10µF
  - Resistor: 100kΩ
  - Purpose: Prevent inrush current surge at power-on

### 6.5 Over-Temperature Shutdown
- **Thermal Sense:** Place temperature sensor near buck converters
  - Part: NTC thermistor or digital sensor (TMP36)
  - Threshold: Disable converters if >80°C
  - Purpose: Prevent thermal runaway in enclosed vehicle

### 6.6 Power OK / Voltage Monitor
- **Comparator Circuit:** LM393 or TL331
  - Monitor 5V and 3.3V rails
  - Pull GPIO low on under/overvoltage
  - Send alert to Raspberry Pi (I2C or GPIO)

---

## 7. Connectors & Mechanical

### 7.1 Input Connectors
- **24V Vehicle Input:** 
  - Anderson Power Pole 45A (or XT60)
  - Keyed to prevent reverse insertion
  - Include panel-mount receptacle with fuse holder inline
  
- **USB-C Power Input:**
  - USB Type-C receptacle (USB 3.1 Gen 2, 100W rated)
  - Include ferrite bead and EMI filter on D+/D-/VBUS
  - Mounted on edge or front panel

### 7.2 Output Connectors
- **To Raspberry Pi:** 
  - 2x 5.5mm x 2.1mm barrel jack + screw terminal backup
  - Or direct XT30 connector
  
- **To ESP32 Modules (x4):**
  - 3-pin header (5V, GND, 3.3V) per module
  - Or JST PH 2.0 connector
  - Keyed to prevent misalignment

### 7.3 Debug / Monitoring Headers
- **I2C Header:** SDA, SCL, GND (for INA219 current sense, TMP36 temp)
- **UART Header:** TX, RX, GND (for debug logs from Pi)
- **Power Monitor Header:** Access to 5V/3.3V rails with sense lines

---

## 8. PCB Layout Guidelines

### 8.1 Layer Stackup
- **4-layer board (recommended):**
  - Layer 1: Signal + 24V input traces
  - Layer 2: GND plane (entire layer)
  - Layer 3: 5V and 3.3V distribution
  - Layer 4: Signal returns + test points

### 8.2 Power Plane Strategy
- **24V Bus:** Thick traces, min 0.5mm width (or power plane zone)
- **5V Bus:** Dedicated area on Layer 3, >0.3mm traces, wide
- **3.3V Bus:** Separate zone from 5V to minimize coupling
- **GND Plane:** Continuous Layer 2, star-point return to 24V input

### 8.3 Thermal Management
- **Buck Converters:** Via-stitch thermal vias under power devices
  - Via pattern: 0.3mm vias on 0.5mm pitch
  - Connect to internal GND plane for heat dissipation
- **Heatsink:** Optional adhesive heatsink on top of TPS54160 if >2W dissipation
- **Airflow:** Keep converters away from enclosed edges; add ventilation holes if needed

### 8.4 Noise & EMI
- **Input Filtering:**
  - 100µF bulk cap + 10µF ceramic cap close to 24V input
  - Ferrite bead on 24V trace before buck converter
  - RC snubber (10Ω + 100nF) on converter input
  
- **Output Filtering:**
  - 100µF + 10µF on 5V output (bulk + fast response)
  - 47µF + 10µF on 3.3V output
  - Separate caps per ESP32 module (10µF + 1µF near each LDO)

- **Routing:**
  - Keep high-current traces wide (>0.5mm)
  - Separate analog/digital ground return paths
  - Use ground vias liberally under converters and filters
  - Minimize loop area on input/output traces

### 8.5 Component Placement
- **Fuse holders:** Immediately after input connectors
- **Buck converters:** Central location, away from high-noise circuits
- **ESP32 modules:** Clustered near their respective LDOs
- **Raspberry Pi connector:** Far side of board, minimize Pi return current through main GND

---

## 9. Bill of Materials (BOM) - Preliminary

### Power Conversion & Regulation
| Qty | Part | Value | Package | Purpose |
|-----|------|-------|---------|---------|
| 1 | TPS54160 | 5V/8A buck | LQFP48 | 24V → 5V converter |
| 1 | TPS54160 | 3.3V/5A buck | LQFP48 | 24V → 3.3V converter |
| 4 | TL1963A-33 | 3.3V/1A LDO | SOT223 | Per ESP32 regulator |
| 1 | LM1117-3.3 | 3.3V/1A LDO | SOT223 | Redundant 3.3V (optional) |

### Protection & Filtering
| Qty | Part | Value | Package | Purpose |
|-----|------|-------|---------|---------|
| 1 | MBR3060PT | 60V/30A Schottky | TO-220 | Reverse polarity protection |
| 1 | SMDJ30CA | 30V TVS | SMB | Transient suppression |
| 1 | PRTR5V0U2X | 5V TVS array | SOT23-6 | USB-C ESD protection |
| 1 | PTC fuse | 20A (24V) | ATO | Main 24V fuse |
| 1 | PTC fuse | 5A (USB-C) | Cartridge | USB input fuse |
| 4 | PPTC0805P | 1A resettable | 0805 | Per ESP32 output fuse |

### Capacitors
| Qty | Part | Value | Voltage | Package | Purpose |
|-----|------|-------|---------|---------|---------|
| 2 | Ceramic cap | 100µF | 50V | 1210 | 24V input bulk |
| 4 | Ceramic cap | 100µF | 10V | 1206 | 5V output bulk |
| 4 | Ceramic cap | 47µF | 10V | 1206 | 3.3V output bulk |
| 8 | Ceramic cap | 10µF | 10V | 0805 | Local filtering |
| 8 | Ceramic cap | 1µF | 10V | 0603 | Fast transient response |
| 4 | Ceramic cap | 100nF | 10V | 0603 | Decoupling |

### Other Components
| Qty | Part | Value | Package | Purpose |
|-----|------|-------|---------|---------|
| 2 | Ferrite bead | 600Ω @ 100MHz | 0805 | Input/output noise filtering |
| 2 | Resistor | 10Ω | 0805 | RC snubber |
| 4 | Inductor | 4.7µH | 1210 | Buck converter output choke |
| 1 | USB Type-C | Receptacle | SMD | Power bank input |
| 1 | Anderson PP | 45A | Chassis mount | Vehicle 24V input |
| 1 | INA219 | I2C current sense | SOT23-8 | Power monitoring (optional) |
| 1 | TMP36 | Temp sensor | TO-92 / SMD | Thermal monitoring (optional) |

---

## 10. Testing & Validation Plan

### 10.1 Pre-Power Tests
- Continuity check on all traces (especially 24V and GND)
- Resistance check: 24V to GND should be >100kΩ (no shorts)
- Capacitor ESR measurement (verify no shorted caps)

### 10.2 Power-On Tests
- Apply 24V via current-limited supply (500mA limit)
  - Measure 5V rail: should reach 5.0V ±5%
  - Measure 3.3V rail: should reach 3.3V ±10%
  - No smoke or excessive heat
  
- Apply USB-C 5V via current-limited supply
  - Measure 5V rail
  - Measure 3.3V rail (via LDO from 5V)
  - Current draw should be <100mA idle

### 10.3 Load Tests
- Connect Raspberry Pi: 
  - Load current to 2A, verify 5V ±5%
  - Monitor noise with oscilloscope
  
- Connect 4x ESP32-S3 modules:
  - Simulate Wi-Fi transmit (1.5A peak per module)
  - Verify 3.3V stays within ±10%
  - Measure power transient response

### 10.4 Fuse & Protection Tests
- **24V Fuse:** Apply 24V through 50A current source, verify 20A fuse blows
- **USB-C Fuse:** Short USB-C VBUS to GND, verify 5A fuse blows or PTC resets
- **Reverse Polarity:** Connect 24V backwards, verify Schottky blocks (no current flow)
- **TVS Suppression:** Inject 60V pulse on 24V input, verify TVS clamps

### 10.5 Thermal Tests
- Run full load (Pi + 4x ESP32 Wi-Fi active) for 30 minutes
- Measure buck converter heatsink temperature: <80°C ambient
- Check for thermal runaway or shutdown

### 10.6 Long-Term Reliability
- Soak test: 24V input for 8 hours at full load
- USB-C power-cycle test: 100 cycles on/off every 30 seconds
- Measure output voltage drift over time

---

## 11. Design Decision Matrix

| Decision | Option A | Option B | Chosen | Rationale |
|----------|----------|----------|--------|-----------|
| **3.3V Distribution** | Shared buck | Per-module LDO | LDO | Better isolation, less crosstalk |
| **24V/USB Priority** | Diode-OR | MOSFET Mux | Diode-OR | Simpler, proven automotive |
| **Fusing Strategy** | Main only | Multi-stage | Multi-stage | Better fault isolation |
| **Monitoring** | None | I2C (INA219) | I2C (optional) | Allows runtime telemetry |
| **Reverse Polarity** | Diode | MOSFET | Diode | Lower cost, acceptable drop |

---

## 12. Deliverables

- [ ] Schematic (KiCad or Altium)
- [ ] PCB layout (4-layer, 10cm x 10cm target)
- [ ] Gerber files + drill files
- [ ] BOM with supplier links and datasheets
- [ ] Assembly drawings and silkscreen labels
- [ ] Power simulation (SPICE model for transient response)
- [ ] Thermal FEA / heatsink sizing
- [ ] Test protocol and acceptance criteria
- [ ] Firmware (Raspberry Pi I2C driver for INA219 current monitoring)

---

## 13. Future Enhancements

- Add CAN bus isolation for vehicle network integration
- GPIO control for per-module power sequencing
- Wireless remote monitoring (MQTT over Wi-Fi)
- Redundant buck converters with failover logic
- Configurable output voltage via I2C DAC

---

**Document Version:** 1.0  
**Last Updated:** 2026-10-10  
**Status:** Design Phase - Ready for Schematic
