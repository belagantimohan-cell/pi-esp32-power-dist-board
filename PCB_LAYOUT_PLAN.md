# PCB Layout Plan

## 1. Recommended board stackup

Use a 4-layer PCB for best performance and easier grounding.

### Suggested stackup
- Layer 1: Top — power input, fuses, high-current traces, some signal routing
- Layer 2: Internal GND plane — continuous ground reference
- Layer 3: Internal power plane — 5V and 3.3V distribution
- Layer 4: Bottom — remaining signal routing, test points, connectors

## 2. Placement strategy

### 2.1 Input section
Place all input protection parts as close as possible to the connectors.
Recommended order:
1. Input connector
2. Fuse holder or panel fuse
3. Reverse polarity element
4. TVS clamp
5. Bulk capacitor
6. Buck converter input

### 2.2 Power conversion section
Arrange the two DC-DC converters near the center of the board, with clear thermal paths.
- Keep the 24V input + converter loop as short as possible
- Use wide copper pours for input and output power paths
- Place an output capacitor right at each converter output

### 2.3 Raspberry Pi section
- Place the Pi 5V connector near the 5V output and fuse
- Keep the Pi load current path short and direct
- Avoid routing noisy 3.3V or RF traces near the Pi 5V input path

### 2.4 ESP32 section
- Place the 4 ESP32-S3 modules close to the 3.3V regulators and capacitors
- Keep each module's local bypass capacitor close to its power pins
- Route 3.3V and GND paired together to minimize loop area
- Keep Wi-Fi antennas away from the converter switching nodes

## 3. Routing rules

### 3.1 High-current traces
- Use wide traces or copper pours for:
  - 24V input path
  - 5V Pi rail
  - 3.3V ESP32 rail
- Minimum recommended:
  - 24V path: 0.5 mm to 1.0 mm depending on current
  - 5V path: 1.0 mm or wider for 5A+
  - 3.3V path: 0.8 mm or wider with copper pours

### 3.2 GND strategy
- Use a continuous ground plane on Layer 2
- Connect all grounds to the same plane with short returns
- Add vias near converter grounds and connector grounds
- Do not route return current through long narrow traces

### 3.3 Decoupling placement
Place decoupling capacitors physically close to:
- buck converter VIN and VOUT pins
- Pi 5V input
- ESP32-S3 power pins
- LDO outputs

## 4. Thermal design

### 4.1 Buck converters
- Use large copper pours on both top and bottom layers
- Add multiple vias under the thermal pad
- Keep switching nodes away from sensitive analog or RF areas

### 4.2 Power dissipation
- Validate converter heating under worst-case 24V input and full load
- Include an airflow path or heatsink if needed
- Measure surface temperature at 30 minutes under full load

## 5. EMI and noise control

- Place the input TVS and bulk caps close to the connector
- Route the converter switching nodes short and compact
- Use an RC snubber if needed at the converter input or output
- Use ferrite beads on power rails feeding sensitive blocks
- Keep the Raspberry Pi and ESP32 analog reference grounds clean

## 6. Connector placement options

### Option A: Front panel layout
- 24V vehicle inlet on one side
- USB-C port on another side
- Pi and ESP32 output connectors on the opposite edge
- Good for lab prototype and serviceability

### Option B: Compact board layout
- 24V input on left
- USB-C at top
- Pi and ESP32 connectors clustered in center/right side
- Better for compact enclosure, but more difficult for thermal management

## 7. Suggested silkscreen labels

Add visible labels:
- VIN_24V
- VIN_USB_C
- FUSE_24V_20A
- FUSE_USB_5A
- +5V_PI
- +3V3_ESP32
- GND
- POWER_OK
- TEMP_WARNING

## 8. Testpoint placement

Add test points for:
- 24V input after fuse
- USB-C VBUS before OR-ing
- 5V Pi rail
- 3.3V ESP32 rail
- GND reference
- converter enable / power-good signal

## 9. Layout checklist

- [ ] Input protection close to connector
- [ ] Main fuse at source input
- [ ] Flooded ground plane on internal layer
- [ ] 5V rail kept away from RF-sensitive traces
- [ ] 3.3V rail routed with local decoupling near each ESP32
- [ ] High-current paths short and wide
- [ ] Thermal vias under buck regulators
- [ ] Test points available for bench validation
- [ ] Board edges clear for enclosure mounting

## 10. Recommended manufacturing notes

- Use ENIG finish for better assembly reliability
- Request stencil for SMD power components
- Confirm copper thickness (2 oz recommended for high-current paths)
- Verify board connector mounting holes before final fabrication

## 11. Final layout recommendation

For a robust first revision:
- Put the 24V input protection on the left edge
- Keep the two buck converters in the central power region
- Put the Raspberry Pi 5V output on the right side
- Put the ESP32-S3 3.3V distribution on the lower section of the board
- Add a ground ring around power components for thermal and EMI control

This layout gives a clean separation between:
- input protection
- power conversion
- Pi load
- ESP32 load
- ground reference and filtering
