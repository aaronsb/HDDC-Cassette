# Mechanical Components

Revolutionary PCB motor designs and precision mechanical assemblies.

## Overview

This directory contains the mechanical designs for the HDDC system, including the revolutionary dual-spindle PCB motor approach that eliminates traditional capstan mechanisms.

## Revolutionary Design Philosophy

Instead of traditional cassette mechanisms, HDDC uses:
- **Dual PCB Motors**: Integrated directly into main circuit board
- **Direct Spindle Drive**: No capstan, pinch roller, or belts
- **Electronic Tension Control**: Real-time torque sensing and adjustment
- **Physics-Based Positioning**: Calculate tape position from reel diameters

## PCB Motor Specifications

### Capstan Motor (Eliminated!)
*Traditional capstan is replaced by intelligent dual-spindle control*

### Reel Motors (2x)
- **Type**: PCB stator axial flux motors
- **Diameter**: 25-40mm
- **Torque**: 10-40 mNm (sufficient for full reels)
- **Control**: Current-mode torque control
- **Integration**: Part of main PCB assembly

## Mechanical Design Files

### PCB Motor Designs
- `pcb-motor-stator-6pole.kicad_pro` - 6-pole motor PCB layout
- `pcb-motor-stator-9pole.kicad_pro` - 9-pole smooth rotation design
- `rotor-magnet-assembly.step` - 3D printed rotor with N52 magnets
- `motor-simulation.py` - Electromagnetic simulation and optimization

### Cassette Mechanism Modifications
- `mechanism-base-plate.step` - Precision mounting for reel motors
- `head-positioning-system.step` - Micrometer adjustment mechanism
- `tape-guide-improvements.step` - Reduced friction guides
- `bearing-upgrades.step` - Replace plastic with precision bearings

### Alignment and Calibration Fixtures
- `azimuth-alignment-jig.step` - 2 arc-minute precision fixture
- `head-height-gauge.step` - ±50μm vertical positioning
- `track-alignment-scope.step` - Optical alignment verification

## Torque Requirements Analysis

```
Full reel (25mm radius):
- Required torque: 10 mNm
- With 2x safety: 20 mNm

Empty reel (11mm radius):  
- Required torque: 4.4 mNm
- With 2x safety: 9 mNm

PCB Motor Capability:
- 30mm motor: 10-20 mNm ✓
- 40mm motor: 20-40 mNm ✓✓
```

## Control Algorithm Concept

```python
# Dual-spindle control pseudocode
supply_radius = calculate_from_rotation_count()
takeup_radius = calculate_from_rotation_count()

# Maintain constant tape speed
takeup_rpm = TAPE_SPEED / (2 * π * takeup_radius)
supply_rpm = TAPE_SPEED / (2 * π * supply_radius)

# Maintain tension via torque differential
supply_torque = -TENSION * supply_radius  # Brake
takeup_torque = +TENSION * takeup_radius  # Drive
```

## Manufacturing Considerations

### PCB Motor Fabrication
- **Stator**: 4-6 layer PCB with heavy copper (2oz minimum)
- **Rotor**: 3D printed carrier with press-fit N52 magnets
- **Assembly**: Requires precision bearing press-fit
- **Cost**: $20-50 per motor in prototype quantities

### Precision Requirements
- **Rotor-stator gap**: 0.5-1mm (critical for performance)
- **Bearing alignment**: ±0.1mm concentricity
- **Magnet placement**: ±0.2mm angular positioning

## Advantages of PCB Motor Approach

✓ **Perfect Speed Stability** - No cogging torque
✓ **Integration** - Motors become part of main PCB
✓ **Customization** - Optimized for exact torque needs
✓ **Simplicity** - Eliminates belts, pulleys, gears
✓ **Reliability** - Fewer mechanical failure points
✓ **Cost** - Integrated manufacturing in production

## Getting Started

1. **Design single test motor** - Validate PCB motor concept
2. **Characterize performance** - Measure torque/speed curves  
3. **Create mechanism mock-up** - Test with real cassettes
4. **Develop control algorithms** - Physics-based positioning
5. **Integrate with head assembly** - Complete mechanical system

## References

- Section 13 of main technical specification (PCB Motor Implementation)
- Section 4.3 (Motor Control System)
- Axial flux motor design principles
- Precision tape transport mechanisms

*Note: This represents a fundamental reimagining of cassette tape mechanics using 21st century technology. The dual-spindle approach is theoretical but based on sound engineering principles.*