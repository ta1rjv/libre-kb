# LIBRESDR BOARD COMPARISON

## Comparison of LibreSDR vs PlutoSDR and Other Platforms

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | COMPARISON |
| **Last Updated** | 2026-09-07 |

---

## 1. Overview

This document compares LibreSDR with other PlutoSDR-compatible platforms to help users choose the right hardware for their needs.

---

## 2. LibreSDR vs PlutoSDR

### 2.1 Hardware Comparison

| Feature | LibreSDR | PlutoSDR | Winner |
|---------|----------|----------|--------|
| **SoC** | Zynq-7020 | Zynq-7010 | LibreSDR |
| **FPGA LUTs** | 85K | 28K | LibreSDR |
| **DDR Memory** | 1 GB | 512 MB | LibreSDR |
| **Ethernet** | Gigabit | 100 Mbps | LibreSDR |
| **Boot Mode** | SD + QSPI | QSPI only | LibreSDR |
| **RF Chip** | AD9363 | AD9363 | Tie |
| **USB** | USB 2.0 OTG | USB 2.0 OTG | Tie |
| **Price** | Similar | Similar | Tie |

### 2.2 Performance Comparison

| Metric | LibreSDR | PlutoSDR | Improvement |
|--------|----------|----------|-------------|
| Max Sample Rate | 27.5 MSPS | ~12 MSPS | 2.3x |
| Network Throughput | 900 Mbps | ~94 Mbps | 9.5x |
| Memory for Buffers | 1 GB | 512 MB | 2x |
| FPGA Resources | 85K LUTs | 28K LUTs | 3x |

### 2.3 Use Case Comparison

| Use Case | LibreSDR | PlutoSDR | Recommendation |
|----------|-----------|----------|----------------|
| Basic SDR | Both work | Both work | Either |
| High BW streaming | LibreSDR | Limited | LibreSDR |
| FPGA development | LibreSDR | Limited | LibreSDR |
| Custom bitstreams | LibreSDR | Limited | LibreSDR |
| DATV/DVB-S2 TX | LibreSDR | PlutoSDR+ | LibreSDR |
| Portable projects | Either | Either | Either |

---

## 3. LibreSDR vs Tezuka Firmware Boards

Tezuka firmware (`F5OEO/tezuka_fw`) supports multiple boards. LibreSDR is one of them.

### 3.1 Board Matrix from tezuka_fw

| Board | SoC | SD | GbE | DATV | USB | Notes |
|-------|-----|-----|-----|------|-----|-------|
| **LibreSDR** | 7020 | Yes | Yes | Yes | USB-C | This document |
| PlutoSDR | 7010 | No | No | No | micro | Original |
| PlutoPlus | 7010 | Yes | Yes | No | micro | Enhanced Pluto |
| Antsdr E200 | 7020 | Yes | Yes | Yes | USB-C | Similar to LibreSDR |
| Antsdr E310 | 7020 | Yes | Yes | Yes | USB-C | Dual AD9364 |
| Fishball | 7010/7020 | Yes | Yes | No | USB-C | Budget option |
| SignalSDRPro | 7020 | Yes | Yes | Yes | USB-3 | High-end |

### 3.2 LibreSDR vs Antsdr E310

| Feature | LibreSDR | Antsdr E310 | Notes |
|---------|----------|-------------|-------|
| SoC | Zynq-7020 | Zynq-7020 | Same |
| RF Chip | AD9363 | AD9364 (2x) | E310 has dual RX/TX |
| Channels | 1T1R | 2T2R | E310 advantage |
| Price | Similar | Higher | |
| Firmware | tezuka_fw | tezuka_fw | Same |

---

## 4. Platform Selection Guide

### 4.1 Choose LibreSDR if:

- Budget is limited but want better specs
- Need higher sample rates
- Want Gigabit Ethernet for streaming
- Interested in FPGA development
- Prefer SD card boot for easy updates

### 4.2 Choose PlutoSDR if:

- Need official ADI support
- Simple SDR experiments
- Lower power consumption needed
- Already own a PlutoSDR

### 4.3 Choose Other Boards if:

- Need 2T2R operation (Antsdr E310)
- Need USB 3.0 (SignalSDRPro)
- Need specific form factor
- Need DATV support with official firmware

---

## 5. Firmware Compatibility

### 5.1 Firmware Sources

| Source | LibreSDR | PlutoSDR | Other Boards |
|--------|----------|----------|-------------|
| Official ADI | No | Yes | No |
| hz12opensource/libresdr | Yes | No | No |
| tezuka_fw | Yes | Yes | Yes |
| Custom builds | Yes | Yes | Yes |

### 5.2 Software Compatibility

| Software | LibreSDR | PlutoSDR | Notes |
|----------|----------|----------|-------|
| libiio | Yes | Yes | API compatible |
| pyadi-iio | Yes | Yes | API compatible |
| SDR++ | Yes | Yes | Use F5OEO fork for features |
| SDRangel | Yes | Yes | |
| GQRX | Yes | Yes | |
| GNU Radio | Yes | Yes | |
| SatDump | Yes | Yes | Use F5OEO fork |

---

## 6. Hardware Expansion

### 6.1 LibreSDR Advantages

| Feature | Benefit |
|---------|---------|
| Larger FPGA | Room for custom IP cores |
| More DDR | Larger capture buffers |
| SD Card | Easy firmware updates |
| GbE | High-speed data transfer |

### 6.2 Potential Expansions

1. **Custom FPGA Logic**: Use extra LUTs for signal processing
2. **Large Buffer Capture**: 1GB RAM for long recordings
3. **Network Streaming**: GbE for multi-device setups
4. **Storage**: Large SD cards for local recording

---

## 7. Price/Performance Analysis

### 7.1 Cost per Performance

| Board | Approx. Price | Max Sample Rate | $ per MSPS |
|-------|---------------|-----------------|-------------|
| PlutoSDR | $149 | 12 MSPS | $12.42 |
| LibreSDR | $149 | 27.5 MSPS | $5.42 |
| Antsdr E310 | $299 | 61.44 MSPS | $4.87 |

### 7.2 Value Assessment

- **LibreSDR** offers the best value for money if 27.5 MSPS is sufficient
- **Antsdr E310** offers more channels and bandwidth at higher price
- **PlutoSDR** is best for official support and simplicity

---

## 8. Future Considerations

### 8.1 LibreSDR Roadmap

1. Full 2T2R mode implementation
2. Improved LVDS mode support
3. Custom bitstream releases
4. Enhanced cooling solutions

### 8.2 Community Support

- GitHub repositories active
- tezuka_fw has CI/CD for LibreSDR
- Active development community

---

## 9. Summary

| Criteria | Winner |
|----------|--------|
| Hardware specs | LibreSDR |
| Price/performance | LibreSDR |
| Official support | PlutoSDR |
| Community activity | Tie (tezuka_fw) |
| Ease of use | PlutoSDR |
| FPGA development | LibreSDR |

**Recommendation**: LibreSDR offers the best value for users who want enhanced specifications at a similar price to PlutoSDR. For those needing official support or simpler operation, PlutoSDR remains a valid choice.
