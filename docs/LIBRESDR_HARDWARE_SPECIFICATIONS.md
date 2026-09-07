# LIBRESDR HARDWARE SPECIFICATIONS

## Detailed Hardware Specifications - LibreSDR/ZynqSDR Platform

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | HARDWARE SPECIFICATIONS |
| **Last Updated** | 2026-09-07 |

---

## 1. SoC Specifications

### 1.1 Xilinx Zynq-7020

| Parameter | Value |
|-----------|-------|
| Part Number | XC7Z020CLG400-1 |
| Logic Cells | 85,000 |
| Slices | 53,200 |
| LUTs | 53,200 |
| Flip-Flops | 106,400 |
| Block RAM | 4.9 Mb |
| DSP Slices | 220 |
| I/O Pins | 200 |
| Package | BGA (400-pin) |

### 1.2 Processing System (PS)

| Parameter | Value |
|-----------|-------|
| Processor | ARM Cortex-A9 (dual-core) |
| Default Clock | 750 MHz |
| Max Clock | 1100 MHz (overclocked) |
| L1 Cache | 32 KB instruction + 32 KB data per core |
| L2 Cache | 512 KB unified |
| NEON SIMD | Yes |
| FPU | VFPv3 |

---

## 2. Memory Specifications

### 2.1 DDR3 Memory

| Parameter | Value |
|-----------|-------|
| Type | DDR3 |
| Size | 1 GB |
| Organization | 32-bit width |
| Default Speed | 525 MHz |
| Overclock Speed | Up to 750 MHz |
| Voltage | 1.5V |

### 2.2 QSPI Flash

| Parameter | Value |
|-----------|-------|
| Size | 32 MB |
| Interface | SPI x4 |
| Speed | 104 MHz |
| Manufacturer | Various |

---

## 3. RF Subsystem

### 3.1 AD9363 RF Transceiver

| Parameter | Value |
|-----------|-------|
| Frequency Range | 325 MHz - 3.8 GHz |
| RX Channels | 1 or 2 (2T2R capable) |
| TX Channels | 1 or 2 |
| ADC Resolution | 12-bit |
| Max Sample Rate | 61.44 MSPS |
| Interface | LVDS or CMOS |
| Output Power | Up to +7 dBm |
| Noise Figure | ~3 dB |

### 3.2 Clock Configuration

| Parameter | Value |
|-----------|-------|
| Reference Clock | 40 MHz (TCXO) |
| BBPLL | 983.04 MHz |
| ADC Clock | 245.76 MHz |
| DAC Clock | 122.88 MHz |

---

## 4. Connectivity

### 4.1 Ethernet

| Parameter | Value |
|-----------|-------|
| Type | Gigabit Ethernet |
| PHY | 10/100/1000 Mbps |
| Interface | RGMII |
| MAC | Zynq GEM |

### 4.2 USB

| Parameter | Value |
|-----------|-------|
| Type | USB 2.0 OTG |
| Speed | 480 Mbps (High Speed) |
| Modes | Device, Host, OTG |

### 4.3 Debug

| Parameter | Value |
|-----------|-------|
| UART | 115200 8N1 |
| Interface | USB-UART (ttyUSB2) |

---

## 5. Power

### 5.1 Power Requirements

| Parameter | Value |
|-----------|-------|
| Input Voltage | 5V DC |
| Input Current | ~1A typical |
| Power Consumption | ~5W typical |

### 5.2 Power Rails

| Rail | Voltage | Description |
|------|---------|-------------|
| VCCPINT | 1.0V | PS internal |
| VCCPAUX | 1.8V | PS auxiliary |
| VCCO_DDR | 1.5V | DDR memory |
| VCCO_MIO | 3.3V | MIO pins |
| VCCO_IO | 3.3V | PL I/O |
| VCCINT | 1.0V | PL internal |
| VCCAUX | 1.8V | PL auxiliary |

---

## 6. Clock Tree

### 6.1 Reference Clocks

| Clock | Source | Frequency |
|-------|--------|-----------|
| System Reference | 25 MHz crystal | 25 MHz |
| RF Reference | 40 MHz TCXO | 40 MHz |
| Ethernet | PS clock | 125 MHz |

### 6.2 Clock Outputs

| Output | Frequency | Usage |
|--------|-----------|-------|
| CPU | 750-1100 MHz | ARM cores |
| DDR | 525-750 MHz | Memory |
| APER | 50 MHz | AXI peripherals |
| GEM | 125 MHz | Ethernet |

---

## 7. Pinout

### 7.1 Expansion Headers

LibreSDR has expansion headers for:
- GPIO
- Analog inputs
- External reference clock
- Additional I2C/SPI devices

### 7.2 RF Connectors

| Connector | Function |
|-----------|----------|
| RX | Receive input |
| TX | Transmit output |

---

## 8. Physical Specifications

| Parameter | Value |
|-----------|-------|
| Form Factor | Custom PCB |
| Dimensions | ~100mm x 60mm |
| Mounting | Holes for standoffs |
| Cooling | Heatsink recommended for OC |

---

## 9. Environmental

| Parameter | Value |
|-----------|-------|
| Operating Temp | 0 to 70C |
| Storage Temp | -40 to 85C |
| Humidity | 5% to 90% non-condensing |

---

## 10. Schematic Reference

The LibreSDR schematics are available as `zynqsdr_rev5.pdf` in the `hz12opensource/libresdr` repository.
