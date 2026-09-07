# LIBRESDR FPGA ARCHITECTURE

## FPGA Design and Register Map

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | FPGA ARCHITECTURE |
| **Last Updated** | 2026-09-07 |

---

## 1. FPGA Overview

LibreSDR uses Xilinx Zynq-7020 FPGA, which is 3x larger than PlutoSDR's Zynq-7010.

### 1.1 FPGA Resources

| Resource | Zynq-7020 | Zynq-7010 | Available for Custom Logic |
|---------|------------|------------|-------------------------|
| Logic Cells | 85K | 28K | ~60K |
| LUTs | 53,200 | 17,600 | ~28,000 |
| Slices | 13,300 | 4,600 | ~9,000 |
| Flip-Flops | 106,400 | 35,200 | ~71,000 |
| Block RAM | 4.9 Mb | 2.1 Mb | ~2.9 Mb |
| DSP Slices | 220 | 80 | ~180 |

---

## 2. Data Path Architecture

### 2.1 RX Path

```
AD9363 (LVDS)
    |
    v
ADC Interface (axi_ad9361)
    |
    v
RX Filter (CIC + FIR)
    |
    v
DMA Controller (axi_dmac)
    |
    v
DDR Memory
    |
    v
Linux Kernel / libiio
```

### 2.2 TX Path

```
Linux Kernel / libiio
    |
    v
DDR Memory
    |
    v
DMA Controller (axi_dmac)
    |
    v
TX Filter (CIC + FIR)
    |
    v
DAC Interface (axi_ad9361)
    |
    v
AD9363 (LVDS)
```

---

## 3. Register Map

### 3.1 Core Registers

| Address | Module | Size | Description |
|---------|--------|------|-------------|
| 0x79020000 | ADC Core Control | 24 KB | AD9361 ADC interface |
| 0x79024000 | DAC Core Control | 4 KB | AD9361 DAC/DDS interface |
| 0x7C400000 | RX DMA | 4 KB | Receiver DMA controller |
| 0x7C420000 | TX DMA | 4 KB | Transmitter DMA controller |

### 3.2 AXI Addresses

| Signal | Address Range | Size |
|--------|--------------|------|
| HP0 (DDR) | 0x0010_0000 - 0x3FFF_FFFF | 1 GB |
| HP2 (DDR) | 0x0010_0000 - 0x3FFF_FFFF | 1 GB |
| GP0 (MIO) | 0x4000_0000 - 0x7FFF_FFFF | 1 GB |

---

## 4. IP Cores

### 4.1 AD9361 Interface (axi_ad9361)

```
Configuration:
- 2R2T mode support
- LVDS interface
- 12-bit samples
- Real/complex modes
```

### 4.2 DMA Controller (axi_dmac)

```
Features:
- Scatter-gather support
- Cycle mode
- Interrupt on completion
- 32-bit addressing
```

### 4.3 Clock Management

```
PLL Configuration:
- Input: 40 MHz reference
- AD9361 CLOCK: 40 MHz
- BBPLL: 983.04 MHz
- ADC Clock: 245.76 MHz
- DAC Clock: 122.88 MHz
```

---

## 5. Memory Interface

### 5.1 DDR Controller

| Parameter | Value |
|-----------|-------|
| Type | DDR3 |
| Size | 1 GB |
| Width | 32-bit |
| Speed | 525-750 MHz |
| ECC | Not used |

### 5.2 Memory Map

| Region | Address | Size | Usage |
|--------|---------|------|-------|
| Linux | 0x0010_0000 | ~900 MB | Kernel + rootfs |
| IIO Buffer | 0x2000_0000 | 64 MB | RX/TX buffers |
| Bitstream | Loaded to fabric | N/A | FPGA config |

---

## 6. Clock Distribution

### 6.1 Clock Tree

```
Reference Clock (40 MHz)
    |
    +---> BBPLL ---> ADC Clock (245.76 MHz)
    |           +---> DAC Clock (122.88 MHz)
    |
    +---> System Clock (50 MHz)
    |
    +---> Ethernet Clock (125 MHz)
```

### 6.2 Clock Outputs

| Clock | Frequency | Destination |
|-------|-----------|-------------|
| sys_clk | 50 MHz | AXI infrastructure |
| eth_clk | 125 MHz | GEM Ethernet |
| ddr_clk | 525-750 MHz | Memory controller |

---

## 7. LVDS Interface

### 7.1 LVDS vs CMOS

LibreSDR uses LVDS mode for maximum sample rates:

| Mode | Max Sample Rate | Notes |
|------|-----------------|-------|
| CMOS | ~20 MSPS | Default |
| LVDS | 61.44 MSPS | For high BW |

### 7.2 LVDS Signaling

```
Data Lines: 12 bits I, 12 bits Q
Frame Clock: DATA_CLK
Output Enable: RX_FRAME
```

---

## 8. Custom Logic Space

### 8.1 Available Resources

With LibreSDR base design:
- ~28,000 LUTs available
- ~180 DSP slices available
- ~2.9 Mb Block RAM available

### 8.2 Potential Expansions

1. **DDC/DUC**: Custom digital down/up converters
2. **FIR Filters**: Programmable filter chains
3. **FFT Engine**: Hardware FFT accelerator
4. **DVB-S2**: MODCOD encoder for DATV

---

## 9. FPGA Configuration

### 9.1 Configuration Memory

| Parameter | Value |
|-----------|-------|
| Type | SPI x4 |
| Size | 32 MB QSPI |
| Speed | 104 MHz |

### 9.2 Partial Reconfiguration

Not typically used - full bitstream loaded on boot.

---

## 10. Comparison with PlutoSDR

| Feature | LibreSDR | PlutoSDR |
|---------|----------|----------|
| FPGA | Zynq-7020 | Zynq-7010 |
| LUTs | 53,200 | 17,600 |
| BRAM | 4.9 Mb | 2.1 Mb |
| DSP | 220 | 80 |
| LVDS | Full | Limited |
| Custom Logic | More space | Limited |
