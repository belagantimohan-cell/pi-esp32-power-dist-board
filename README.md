# Pi + 4x ESP32-S3 Power Distribution Board

This repository contains the design foundation for a power distribution board that powers:
- 1x Raspberry Pi
- 4x ESP32-S3 modules
- Input sources: 24V vehicle power and USB-C power bank / PD input
- Fuse protection for each input source and major outputs

## Project files
- [DESIGN_SPEC.md](DESIGN_SPEC.md) — full design specification
- [SCHEMATIC_BLOCK_DIAGRAM.md](SCHEMATIC_BLOCK_DIAGRAM.md) — functional block-level diagram and design notes
- [BOM.md](BOM.md) — preliminary bill of materials with example components
- [PCB_LAYOUT_PLAN.md](PCB_LAYOUT_PLAN.md) — placement, routing, stackup, and manufacturing guidance

## System summary
- 24V vehicle input: protected by main fuse and surge protection
- USB-C input: fused and ESD-protected
- 5V rail for Raspberry Pi
- 3.3V rail for ESP32-S3 modules
- Separate power rails for lower noise and better fault isolation
- Optional per-ESP32 LDOs for improved stability

## Next steps
1. Convert the block diagram into a KiCad or Altium schematic
2. Place the buck converters and fuses in the PCB layout
3. Validate the 5V and 3.3V rail load calculations
4. Run transient testing against automotive conditions
