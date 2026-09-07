# LIBRESDR HARDWARE VARIANTS

## Engineering Reference - Board Variants and Differences

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | HARDWARE VARIANTS |
| **Last Updated** | 2026-09-07 |
| **Source** | tezuka_fw boards.json, LibreSDR schematics |

---

## 1. LibreSDR Overview

LibreSDR is a third-party PlutoSDR-compatible board, not an official Analog Devices product. It may come from different manufacturers with varying specifications.

### 1.1 Common Characteristics

All LibreSDR variants share:
- **SoC**: Xilinx Zynq-7020
- **RF**: AD9363
- **Memory**: 1 GB DDR3
- **Flash**: 32 MB QSPI
- **Boot**: SD card + QSPI

---

## 2. Manufacturer Variants

### 2.1 Known Suppliers

| Supplier | Notes |
|----------|-------|
| Various Aliexpress sellers | Quality may vary |
| OpenSDRLab | Possible source |
| Others | Limited documentation |

### 2.2 Hardware Variations

| Feature | Variation | Notes |
|---------|----------|-------|
| PCB revision | rev1 - rev5 | Varies by manufacturer |
| Clock | TCXO quality | May differ |
| Power circuit | Design variations | Affects stability |
| Connectors | RF type varies | SMA, UFL |
| GPIO | May vary | Check schematic |

---

## 3. Schematic Analysis

### 3.1 Available Schematics

The `hz12opensource/libresdr` repository contains `zynqsdr_rev5.pdf` schematic.

### 3.2 Key Components

| Component | Part | Notes |
|----------|------|-------|
| SoC | XC7Z020CLG400 | Zynq-7020 |
| RF | AD9363 | Same as PlutoSDR |
| DDR | MT41K256M16 | 1GB DDR3L |
| Flash | S25FL256S | 32MB QSPI |
| ETH PHY | RTL8211E | Gigabit |
| USB | USB3320 | USB 2.0 OTG |

---

## 4. FPGA Configuration

### 4.1 Zynq-7020 Resources

| Resource | Total | LibreSDR | Available |
|----------|-------|----------|-----------|
| LUTs | 53,200 | ~25,000 | ~28,000 |
| Flip-Flops | 106,400 | ~35,000 | ~71,000 |
| BRAM | 4.9 Mb | ~2 Mb | ~2.9 Mb |
| DSP | 220 | ~40 | ~180 |

### 4.2 FPGA Design

- Based on PlutoSDR HDL design
- Supports LVDS mode for high sample rates
- May include 2T2R support

---

## 5. Clock Variations

### 5.1 TCXO Quality

| Quality | Tolerance | Impact |
|--------|----------|--------|
| High | +/- 2 ppm | Best frequency accuracy |
| Standard | +/- 10 ppm | Acceptable for most uses |
| Budget | +/- 25 ppm | May affect stability |

### 5.2 Clock Configuration

| Clock | Frequency | Source |
|-------|-----------|--------|
| Reference | 40 MHz | TCXO |
| CPU | 750 MHz | ARM PLL |
| DDR | 525 MHz | DDR PLL |

---

## 6. Power Supply

### 6.1 Power Rails

| Rail | Voltage | Description |
|------|---------|-------------|
| VCCPINT | 1.0V | PS internal |
| VCCPAUX | 1.8V | PS auxiliary |
| VCCO_DDR | 1.5V | DDR memory |
| VCCO_MIO | 3.3V | MIO pins |
| VCCINT | 1.0V | PL internal |
| VCCAUX | 1.8V | PL auxiliary |

### 6.2 Power Concerns

- Quality of voltage regulators varies
- Overclocking may stress power delivery
- Thermal management may be inadequate

---

## 7. RF Frontend

### 7.1 AD9363 Configuration

| Parameter | Value |
|-----------|-------|
| Frequency Range | 325 MHz - 3.8 GHz |
| RX Channels | 1 or 2 |
| TX Channels | 1 or 2 |
| Max Sample Rate | 61.44 MSPS |
| Interface | LVDS |

### 7.2 RF Port Configuration

| Port | Function |
|------|----------|
| RX1 | Receive input 1 |
| RX2 | Receive input 2 (if 2T2R) |
| TX1 | Transmit output 1 |
| TX2 | Transmit output 2 (if 2T2R) |

---

## 8. Memory Configuration

### 8.1 DDR3 Specification

| Parameter | Value |
|-----------|-------|
| Size | 1 GB |
| Organization | 32-bit |
| Speed | 525 MHz |
| Type | DDR3L |
| Manufacturer | Various |

### 8.2 DDR Timing

| Parameter | Base | Overclock |
|-----------|------|-----------|
| CAS Latency | 7 | 7 |
| tRCD | 9 | 9 |
| tRP | 9 | 9 |
| Speed | 525 MHz | 750 MHz |

---

## 9. Connectivity Options

### 9.1 USB

| Feature | Specification |
|---------|---------------|
| Standard | USB 2.0 OTG |
| Speed | 480 Mbps |
| Connector | USB-C |

### 9.2 Ethernet

| Feature | Specification |
|---------|---------------|
| Standard | Gigabit Ethernet |
| PHY | Realtek RTL8211E |
| Speed | 10/100/1000 Mbps |
| Interface | RGMII |

### 9.3 SD Card

| Feature | Specification |
|---------|---------------|
| Interface | SD 2.0 |
| Speed | Up to 25 MB/s |
| Boot | Primary boot source |

---

## 10. GPIO Configuration

### 10.1 Common GPIO Assignments

| GPIO | Function |
|------|----------|
| 14 | Button (if present) |
| 15 | LED |
| 52 | USB PHY reset |
| 66 | EN_AGC |
| 67 | AD9361 reset |

### 10.2 LED Behavior

| State | LED |
|-------|-----|
| Heartbeat | Boot OK |
| Fast blink | Updating |
| Off | Error/No power |

---

## 11. Firmware Selection

### 11.1 Compatible Firmware

| Firmware | Source | Notes |
|----------|--------|-------|
| tezuka_fw | F5OEO | Recommended |
| libresdr | hz12opensource | Original OC patches |
| Custom | Build yourself | Requires Vivado |

### 11.2 Board Selection

In tezuka_fw:
```bash
./build.sh libre
```

---

## 12. Quality Variations

### 12.1 What to Check

When receiving a LibreSDR:
1. Verify board markings match expected specs
2. Check for physical damage
3. Test basic functionality before overclocking
4. Verify RF connector quality

### 12.2 Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| No boot | Power supply | Check 5V adapter |
| Unstable OC | Poor cooling | Add heatsink |
| RX noise | Antenna | Check VSWR |
| Eth issues | PHY config | Verify PHY driver |

---

## 13. Comparison with PlutoSDR Variants

### 13.1 Hardware Comparison

| Feature | LibreSDR | PlutoSDR |
|---------|----------|----------|
| SoC | Zynq-7020 | Zynq-7010 |
| FPGA LUTs | 53K | 28K |
| Memory | 1 GB | 512 MB |
| Ethernet | GbE | 100M |
| Boot | SD + QSPI | QSPI |
| Price | Similar | Similar |

### 13.2 Feature Comparison

| Feature | LibreSDR | PlutoSDR |
|---------|----------|----------|
| Overclocking | Yes | No |
| LVDS mode | Yes | Limited |
| Sample rate | 27.5 MSPS | 12 MSPS |
| DATV | Yes | No (unless upgraded) |

---

## 14. Related Documents

- `LIBRESDR_BOARD_COMPARISON.md` - Detailed comparison
- `LIBRESDR_SYSTEM_ARCHITECTURE.md` - System overview
- `LIBRESDR_OVERCLOCKING.md` - Overclocking guide
- `LIBRESDR_HARDWARE_SPECIFICATIONS.md` - Specifications
