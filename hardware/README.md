# Hardware Design

This directory contains the physical hardware designs for the HDDC system.

## Directory Structure

- **head-assembly/**: GMR sensor arrays and magnetic head designs
- **main-board/**: Processor boards, ADCs, and analog frontend
- **mechanical/**: 3D printable parts and mechanical assemblies

## Development Phases

### Phase 1: 8-16 Track Prototype ($200-400)
- **Capacity**: 25-75 MB (Type I/II cassettes)
- 1-2x ADS131M08 ADCs
- 8-16 GMR sensors (NVE AA004-02E)
- ESP32-S3 processor
- Modified cassette mechanism

### Phase 2: 16-32 Track System ($400-800)
- **Capacity**: 75-200 MB (Type II cassettes)
- 2-4x ADS131M08 ADCs
- Custom 16-32 sensor head assembly
- STM32H7 processor
- PCB motor integration

### Phase 3: 32-48 Track System ($800-2000)
- **Capacity**: 200-600 MB (Type IV Metal cassettes)
- Full GMR sensor array
- Revolutionary PCB motor integration
- Professional-grade components

**Reality Check**: Magnetic grain size fundamentally limits practical implementations to Phase 1-2 range. See [MAGNETIC-MEDIA-CONSTRAINTS.md](../MAGNETIC-MEDIA-CONSTRAINTS.md) for detailed analysis.

## Key Technical Challenges

1. **Magnetic Media Limits**: Particle size constrains track density and bit resolution
2. **Print-Through Crosstalk**: Layer-to-layer magnetic interference in wound tape
3. **Track Alignment**: ±25μm lateral positioning required
4. **Azimuth Calibration**: ±2 arc minutes tolerance
5. **PCB Motor Design**: Custom coil patterns and magnetic simulation

**Most Critical**: Magnetic grain resolution limits practical track count to 16-32 tracks, not the originally envisioned 50-80 tracks.

See individual directories for detailed specifications and designs.