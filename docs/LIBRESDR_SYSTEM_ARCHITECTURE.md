# LIBRESDR SYSTEM ARCHITECTURE

## Engineering Reference - LibreSDR/ZynqSDR Platform

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | SYSTEM ARCHITECTURE |
| **Last Updated** | 2026-09-07 |
| **Source Repository** | hz12opensource/libresdr, F5OEO/tezuka_fw |
| **Base Platform** | PlutoSDR v0.38 |

---

## 1. System Overview

LibreSDR (also known as ZynqSDR) is a third-party PlutoSDR-compatible SDR platform that features enhanced specifications compared to the original Analog Devices PlutoSDR:

- **Larger FPGA**: Xilinx Zynq-7020 instead of Z-7010
- **More Memory**: 1 GB DDR3 instead of 512 MB
- **Gigabit Ethernet**: vs PlutoSDR's 100 Mbps
- **SD Card Boot**: Primary boot mode for zero-risk updates
- **Overclocking Support**: Up to 1100 MHz CPU / 750 MHz DDR

### 1.1 Block Diagram

```
+------------------------------------------------------------------+
|                    Xilinx Zynq-7020 (XC7Z020)                    |
|  +-------------------------------------------------------------+ |
|  |                    Processing System (PS)                     | |
|  |  +---------------------------------------------------+  | |
|  |  |              ARM Cortex-A9 (750 MHz default)        |  | |
|  |  |                   (Up to 1100 MHz OC)             |  | |
|  |  +---------------------------------------------------+  | |
|  |                                                              | |
|  |  +---------------------------------------------------+  | |
|  |  |  Peripherals: UART, USB OTG, QSPI, SD Card, GbE  |  | |
|  |  +---------------------------------------------------+  | |
|  +-------------------------------------------------------------+ |
|==================================================================|
|                     Programmable Logic (PL)                       |
|  +-------------------------------------------------------------+ |
|  |                                                               | |
|  |  +------------------+  +-----------------------+               | |
|  |  | AXI Interconnect |  |   GPIO Controller     |               | |
|  |  +------------------+  +-----------------------+               | |
|  |          |                         |                           | |
|  |  +-------+-------+         +-------+-------+                   | |
|  |  | RX DMA      |         | TX DMA      |                     | |
|  |  +--------------+         +--------------+                   | |
|  |          |                         |                           | |
|  |  +-------+-------+         +-------+-------+                   | |
|  |  | ADC Core     |         | DAC Core    |                     | |
|  |  +--------------+         +--------------+                   | |
|  |                                                               | |
|  +---------------------------------------------------------------+ |
+------------------------------------------------------------------+
                    |                              |
          +---------+----------+         +----------------+
          |   AD9363          |         |   QSPI Flash    |
          | RF Transceiver   |         |   (32 MB)       |
          +------------------+         +----------------+
                    |
          +---------+----------+
          |   1 GB DDR3       |
          |   (525-750 MHz)   |
          +------------------+
```

---

## 2. Hardware Specifications

### 2.1 SoC Comparison

| Parameter | LibreSDR | PlutoSDR | Difference |
|-----------|----------|----------|------------|
| FPGA Part | XC7Z020CLG400-1 | XC7Z010CLG400-1 | Larger |
| Logic Cells | 85K | 28K | 3x more |
| LUTs | 53,200 | 17,600 | 3x more |
| Flip-Flops | 106,400 | 35,200 | 3x more |
| Block RAM | 4.9 Mb | 2.1 Mb | 2.3x more |
| DSP Slices | 220 | 80 | 2.75x more |

### 2.2 Memory

| Parameter | LibreSDR | PlutoSDR |
|-----------|----------|----------|
| DDR Type | DDR3 | DDR3L |
| DDR Size | 1 GB | 512 MB |
| DDR Width | 32-bit | 16-bit |
| Default Speed | 525 MHz | 525 MHz |
| Max OC Speed | 750 MHz | N/A |

### 2.3 Connectivity

| Interface | LibreSDR | PlutoSDR |
|-----------|----------|----------|
| Ethernet | Gigabit (GbE) | 100 Mbps |
| USB | USB 2.0 OTG | USB 2.0 OTG |
| SD Card | Yes (primary boot) | No |
| QSPI Flash | 32 MB | 32 MB |
| UART | Debug console | Debug console |

---

## 3. Clock Architecture

### 3.1 Default Clock Configuration

| Clock | Frequency | Source |
|-------|-----------|--------|
| CPU Clock | 750 MHz | 25 MHz * 30 |
| DDR Clock | 525 MHz | 25 MHz * 21 |
| DDR REF | 533 MHz | DDR divider |
| Reference Clock | 40 MHz | External TCXO |
| Ethernet | 125 MHz | PS clock |

### 3.2 Overclock Configurations

| Configuration | CPU | DDR | Status |
|---------------|-----|-----|--------|
| Base | 750 MHz | 525 MHz | Default |
| Medium | 950 MHz | 600 MHz | Tested |
| High | 1100 MHz | 750 MHz | Maximum tested |

---

## 4. RF Subsystem

### 4.1 AD9363 Configuration

LibreSDR uses the same AD9363 RF transceiver as PlutoSDR:

| Parameter | Value |
|-----------|-------|
| Frequency Range | 325 MHz - 3.8 GHz |
| RX Channels | 1 or 2 (2T2R mode) |
| TX Channels | 1 or 2 |
| ADC Resolution | 12-bit |
| Max Sample Rate | 61.44 MSPS |
| Interface | LVDS (for max rate) |

### 4.2 Data Path

```
                    RX Path (2T2R Mode)
                    ================
AD9363 (LVDS) --> ADC Interface --> Decimation --> DMA --> DDR --> Linux/IIO

                    TX Path
                    =======
Linux/IIO --> DMA --> Interpolation --> DAC Interface --> AD9363 (LVDS)
```

---

## 5. Software Stack

```
+---------------------------+
|   User Space Applications |
|   - SDR++                  |
|   - SatDump                |
|   - libiio applications     |
|   - pyadi-iio Python API   |
+---------------------------+
          |
          v
+---------------------------+
|       libiio              |
|   - IIO context management |
|   - Buffer management      |
+---------------------------+
          |
          v
+---------------------------+
|   IIO Kernel Subsystem     |
+---------------------------+
          |
          v
+---------------------------+
|   Linux Device Drivers     |
|   - ad9361 driver          |
|   - axi_ad9361 driver      |
|   - axi_dmac driver        |
+---------------------------+
          |
          v
+---------------------------+
|   FPGA Fabric (PL)         |
|   - AD9361 interface       |
|   - DMA engines            |
+---------------------------+
          |
          v
+---------------------------+
|   AD9363 RF Transceiver    |
+---------------------------+
```

---

## 6. Boot Process

### 6.1 Boot Flow

```
Power On
    |
    v
SD Card / QSPI Flash (boot.bif)
    |
    v
FSBL (First Stage Boot Loader)
    - PS7 initialization
    - Clock configuration
    - DDR initialization
    - FPGA bitstream loading
    |
    v
U-Boot
    - Load environment variables
    - Load kernel/DTB from SD/flash
    - Start Linux kernel
    |
    v
Linux Kernel
    - Device tree parsing
    - Driver initialization
    - Mount root filesystem (ramdisk or SD)
    |
    v
Init Scripts
    - Network services
    - IIO daemon
    - Web interface
    |
    v
Runtime
```

### 6.2 Boot Mode Selection

| Boot Mode | Description |
|-----------|-------------|
| SD Card | Primary boot mode, FAT32 formatted |
| QSPI Flash | Fallback, populated by SD boot |

---

## 7. Network Configuration

### 7.1 Default IP Settings

| Parameter | LibreSDR | PlutoSDR |
|-----------|----------|----------|
| Device IP | 192.168.1.10 | 192.168.2.1 |
| Host IP | DHCP/Static | 192.168.2.2 |
| Protocol | GbE | 100M |
| Services | SSH, HTTP, IIO | Same |

---

## 8. Key Differences from PlutoSDR

| Feature | LibreSDR | PlutoSDR |
|---------|----------|----------|
| Boot Priority | SD > QSPI | QSPI > USB |
| FPGA Size | Zynq-7020 | Zynq-7010 |
| Memory | 1 GB | 512 MB |
| Ethernet | GbE | 100M |
| Default IP | 192.168.1.10 | 192.168.2.1 |
| Overclocking | Supported | Not supported |
| LVDS Mode | Yes | Limited |

---

## 9. Known Limitations

1. **USB Bandwidth**: USB 2.0 limits to ~40 MB/s
2. **FPGA Bitstream**: Requires Vivado for modifications
3. **Memory Timing**: Overclocking may require DDR timing adjustments
4. **Documentation**: Limited official documentation from manufacturer

---

## 10. Future Enhancements

1. **2T2R Mode**: Full utilization of larger FPGA
2. **Custom Bitstreams**: Leverage extra FPGA resources
3. **DATV Support**: DVB-S2 TX capability
4. **Maia-SDR**: Web-based spectrum analyzer
