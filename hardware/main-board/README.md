# Main Board Electronics

*Reference: Technical Specification Sections 4.1-4.2, 4.4, 5.1*

## Overview

The main board provides digital signal processing, analog front-end, and system control for the HDDC system. It must handle real-time processing of 50-80 parallel data channels while maintaining precise timing and low noise operation.

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Main Board Architecture                   │
├─────────────────┬───────────────────────────┬───────────────┤
│ Analog Front-End │    Digital Processing     │ Motor Control │
│                 │                           │               │
│ GMR Head        │ ARM Cortex-M7 @ 480MHz   │ 3-Phase BLDC  │
│ ↓               │ ↓                         │ Drivers       │
│ Preamp (50-80x) │ 1MB SRAM                  │ ↓             │
│ ↓               │ ↓                         │ Capstan +     │
│ Variable Gain   │ Hardware DSP              │ Reel Motors   │
│ ↓               │ ↓                         │               │
│ Anti-alias      │ DMA Controllers (8x)      │ Encoder       │
│ Filter          │ ↓                         │ Feedback      │
│ ↓               │ Real-time OS              │               │
│ ADC (24-bit)    │ (FreeRTOS/Zephyr)        │               │
└─────────────────┴───────────────────────────┴───────────────┘
```

## Processor Selection

### Option 1: STM32H743 (Recommended for Performance)
**Specifications**:
- **Architecture**: ARM Cortex-M7 @ 480MHz
- **RAM**: 1MB SRAM + 32MB external PSRAM
- **Flash**: 2MB for firmware
- **DSP**: Hardware accelerated signal processing
- **DMA**: 16 independent channels
- **Cost**: $15-20

**Advantages**:
- Excellent DSP performance for real-time processing
- Abundant DMA channels for parallel data streaming
- Professional-grade timing accuracy
- Extensive peripheral set

**Use Cases**: Professional implementation, maximum performance

### Option 2: ESP32-S3 (Recommended for Prototyping)
**Specifications**:
- **Architecture**: Dual-core Xtensa @ 240MHz
- **RAM**: 512KB SRAM + 8MB external PSRAM
- **Flash**: 4-16MB for firmware and data
- **Connectivity**: WiFi 802.11b/g/n, Bluetooth 5.0
- **Cost**: $3-8

**Advantages**:
- Built-in wireless connectivity for remote monitoring
- Dual-core allows separation of real-time and application tasks
- Large maker community and development tools
- Low cost for prototyping

**Use Cases**: Development, wireless-enabled implementations, cost-sensitive designs

### Option 3: RP2040 (Minimum Viable)
**Specifications**:
- **Architecture**: Dual Cortex-M0+ @ 133MHz
- **RAM**: 264KB SRAM
- **Flash**: External (2-16MB)
- **PIO**: Programmable I/O for custom timing
- **Cost**: $1-4

**Advantages**:
- Programmable I/O can handle custom protocols
- Very low cost
- Good community support

**Use Cases**: Educational projects, minimal implementations

## Analog Front-End Design

### Read Path Signal Chain
```
GMR Head → Preamp → Variable Gain → Anti-alias Filter → ADC → DSP
   ↓          ↓          ↓                ↓              ↓      ↓
50-80ch    TI INA163  TI PGA280    8th-order        TI ADS131M08
           (per ch)    (8ch mux)    Butterworth      (8ch 24-bit)
```

### Component Specifications

#### Instrumentation Amplifiers (INA163)
- **Gain**: 1-1000 V/V programmable
- **Bandwidth**: 1.3 MHz at G=1
- **Input Noise**: 8 nV/√Hz typical
- **CMRR**: 100 dB at G=100
- **Cost**: $3-5 each (need 50-80)

#### Programmable Gain Amplifiers (PGA280)
- **Gain Range**: 1, 2, 4, 8, 16, 32, 64, 128 V/V
- **Channels**: 8 channels per chip
- **Bandwidth**: 12 MHz at G=1
- **SPI Control**: Digital gain selection
- **Cost**: $8-12 each (need 7-10)

#### Anti-Alias Filters
- **Type**: 8th-order Butterworth
- **Cutoff**: 100 kHz (adjustable)
- **Implementation**: Switched-capacitor or Sallen-Key topology
- **Requirement**: >60dB attenuation at Nyquist frequency

#### ADC (ADS131M08)
- **Resolution**: 24-bit delta-sigma
- **Channels**: 8 simultaneous sampling
- **Sample Rate**: Up to 32 kSPS per channel
- **Built-in PGA**: 1-128x gain
- **Interface**: SPI with data ready interrupt
- **Cost**: $15-20 each (need 7-10)

### Write Path Signal Chain
```
DSP → DAC → Current Driver → Write Head
 ↓      ↓          ↓              ↓
Data   TI DAC53608  DRV134    Custom coil array
       (8ch 10-bit)          (50-80 channels)
```

## Memory Architecture

### Memory Map
```
┌─────────────────┐ 0x24000000
│ External PSRAM  │ 8-32MB - Audio buffer storage
├─────────────────┤ 0x20000000  
│ Internal SRAM   │ 512KB-1MB - Working memory
├─────────────────┤ 0x10000000
│ DMA Buffers     │ 64KB - Real-time I/O buffers
├─────────────────┤ 0x08000000
│ Flash Memory    │ 2MB - Firmware storage
└─────────────────┘
```

### Buffer Management Strategy
```c
// Circular buffer for real-time streaming
typedef struct {
    uint8_t  data[BLOCK_SIZE];      // 4KB per block
    uint64_t position;              // Tape position
    uint32_t timestamp;             // Acquisition time
    uint16_t track_errors;          // Error flags per track
    uint8_t  valid;                 // Block validity
} AudioBlock;

// Multi-buffer system for continuous operation
typedef struct {
    AudioBlock blocks[BUFFER_BLOCKS];  // 30-60 seconds of audio
    uint32_t   write_idx;              // Current write position
    uint32_t   read_idx;               // Current read position
    uint32_t   valid_blocks;           // Number of valid blocks
    uint32_t   overrun_count;          // Buffer overflow counter
} CircularBuffer;
```

## Real-Time Processing Requirements

### Task Priorities (FreeRTOS)
```
Priority 7 (Highest): Motor control ISR
Priority 6: ADC/DAC DMA completion
Priority 5: Track synchronization
Priority 4: Error correction processing
Priority 3: Position tracking
Priority 2: User interface updates
Priority 1 (Lowest): Background maintenance
```

### Timing Constraints
- **ADC Sampling**: 32 kSPS × 80 channels = 2.56 MSPS total
- **Processing Deadline**: <31.25 μs per sample
- **Motor Control**: 1 kHz update rate for smooth operation
- **Buffer Management**: <1ms latency for real-time streaming

## PCB Design Guidelines

### High-Speed Digital Design
- **Layer Stack**: 6-layer PCB minimum
  - Layer 1: Component placement and short traces
  - Layer 2: Ground plane (digital)
  - Layer 3: Power planes (+3.3V, +5V)
  - Layer 4: Signal routing
  - Layer 5: Ground plane (analog)
  - Layer 6: Analog signals and power

### Power Distribution
```
+12V Input → LDO → +5V Analog → LDO → +3.3V Analog
            ↓                     ↓
         +5V Digital ← LDO ← +3.3V Digital
```

### Critical Design Rules
- **Analog/Digital Separation**: Separate ground planes with single connection point
- **Power Decoupling**: 0.1μF + 10μF capacitors at each IC
- **Clock Distribution**: Differential clock signals with matched trace lengths
- **EMI Suppression**: Ground guard traces around sensitive signals

## Motor Control Integration

### PCB Motor Implementation
*Reference: Technical Specification Section 13*

**Capstan Motor (40mm diameter)**:
- **Coil Design**: 9-pole stator in star configuration
- **PCB Layers**: 4-6 layers with 2oz copper
- **Control**: 3-phase BLDC driver (DRV8313)
- **Feedback**: Integrated optical encoder or Hall sensors

**Reel Motors (25mm diameter)**:
- **Coil Design**: 6-pole stator for cost reduction
- **Torque Control**: Current-mode control for tension
- **Bidirectional**: Forward/reverse operation
- **Sensing**: Current measurement for torque feedback

### Control Algorithms
```c
// Dual-spindle control with tension management
typedef struct {
    float supply_radius;        // Calculated from encoder
    float takeup_radius;        // Calculated from encoder
    float target_speed;         // Constant tape speed
    float tension_setpoint;     // Desired tape tension
    float supply_torque;        // Brake torque (negative)
    float takeup_torque;        // Drive torque (positive)
} TapeControl;

void update_tape_control(TapeControl* ctrl) {
    // Calculate required RPM for constant tape speed
    ctrl->takeup_rpm = ctrl->target_speed / (2 * PI * ctrl->takeup_radius);
    ctrl->supply_rpm = ctrl->target_speed / (2 * PI * ctrl->supply_radius);
    
    // Adjust torques for tension control
    ctrl->supply_torque = -ctrl->tension_setpoint * ctrl->supply_radius;
    ctrl->takeup_torque = +ctrl->tension_setpoint * ctrl->takeup_radius;
}
```

## Power Management

### Power Budget Analysis
```
Component                     Current    Power
Processor (STM32H7)          150 mA     0.5W
ADCs (8x ADS131M08)          200 mA     1.0W
Analog front-end             300 mA     1.5W
Motor drivers (3x)           1000 mA    5.0W
Display and UI               50 mA      0.2W
Total                        1700 mA    8.2W
```

### Battery Operation
- **Battery Type**: 3.7V Li-ion, 2200-3000 mAh
- **Operating Time**: 2-3 hours continuous operation
- **Charging**: USB-C PD with 5V/2A input
- **Power Management**: Sleep modes during idle periods

## Testing and Validation

### Functional Tests
1. **Power-up sequence**: Verify all supply voltages within tolerance
2. **ADC calibration**: Offset and gain calibration for each channel
3. **Motor control**: Speed regulation and torque control verification
4. **DMA operation**: Data transfer without CPU intervention
5. **Real-time performance**: Timing analysis under full load

### Performance Measurements
- **Dynamic range**: Signal-to-noise ratio across all channels
- **Crosstalk**: Inter-channel isolation measurement
- **Frequency response**: Verify anti-aliasing filter performance
- **Power consumption**: Measure actual vs. theoretical power usage
- **Thermal performance**: Temperature measurement under load

## Bill of Materials (Estimated)

### Professional Implementation
```
STM32H743 processor:        $20
ADCs (8x ADS131M08):        $160
Analog front-end:           $400
Motor drivers:              $60
PCB (6-layer):              $200
Passive components:         $100
Connectors and misc:        $60
Total:                      $1000
```

### Development Implementation
```
ESP32-S3 module:            $15
ADCs (2x ADS131M08):        $40
Simplified analog:          $100
Motor drivers:              $30
PCB (4-layer):              $80
Passive components:         $50
Development board:          $25
Total:                      $340
```

## Educational Value

This component teaches:

### Digital Signal Processing
- **Real-time constraints**: Meeting hard deadlines in embedded systems
- **DMA operation**: Hardware-accelerated data movement
- **Filter design**: Anti-aliasing and reconstruction filters
- **Error correction**: Hardware implementation of Reed-Solomon codes

### Embedded Systems Design
- **RTOS concepts**: Task scheduling and priority management
- **Memory management**: Efficient buffer allocation and management
- **Hardware abstraction**: Portable code across different processors
- **Interrupt handling**: Low-latency response to real-time events

### Analog Circuit Design
- **Instrumentation amplifiers**: Low-noise signal conditioning
- **ADC selection**: Resolution vs. speed trade-offs
- **Power supply design**: Low-noise analog power distribution
- **PCB layout**: High-speed digital and precision analog integration

## Next Steps

1. **Choose processor** based on performance requirements and budget
2. **Design analog front-end** starting with single-channel prototype
3. **Implement motor control** using existing BLDC controller ICs
4. **Develop firmware** with focus on real-time constraints
5. **Validate performance** with comprehensive test procedures

---

*The main board represents the engineering heart of the HDDC system, requiring expertise in both digital signal processing and precision analog design. Start with a simplified implementation and gradually add complexity as understanding develops.*