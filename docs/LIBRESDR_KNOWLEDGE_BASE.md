# LIBRESDR KNOWLEDGE BASE

## Engineering Reference Document - LibreSDR/ZynqSDR Platform

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | KNOWLEDGE BASE |
| **Last Updated** | 2026-09-07 |
| **Source Repository** | hz12opensource/libresdr, F5OEO/tezuka_fw |
| **Base Platform** | PlutoSDR v0.38 |

---

## 1. System Summary

LibreSDR (ZynqSDR) is a PlutoSDR-compatible SDR platform featuring a larger Xilinx Zynq-7020 FPGA and 1GB DDR memory. It supports overclocking for higher sample rates and boots primarily from SD card.

### 1.1 Key Specifications

| Parameter | Value |
|-----------|-------|
| SoC | Xilinx Zynq-7020 (ARM Cortex-A9 + FPGA) |
| FPGA | XC7Z020CLG400-1 (85K LUTs) |
| CPU Clock | 750 MHz (default), up to 1100 MHz (OC) |
| DDR | 1 GB DDR3 @ 525-750 MHz |
| RF Transceiver | Analog Devices AD9363 (1Rx-1Tx or 2Rx-2Tx) |
| Frequency Range | 325 MHz - 3800 MHz |
| Sample Rate | Up to 61.44 MSPS |
| ADC Resolution | 12-bit |
| Ethernet | Gigabit |
| USB | USB 2.0 OTG |
| Boot | SD Card (primary), QSPI Flash |
| Default IP | 192.168.1.10 |

---

## 2. Hardware Architecture

### 2.1 Block Diagram

```
+----------------------------------------------------------+
|                Xilinx Zynq Z-7020                        |
|  +-----------------------------------------------------+ |
|  |              Processing System (PS)                   | |
|  |  +----------------------------------------------+  | |
|  |  |           ARM Cortex-A9 (750 MHz)             |  | |
|  |  +----------------------------------------------+  | |
|  |                                                      | |
|  |  +----------------------------------------------+  | |
|  |  |   Peripherals: UART, USB, QSPI, SD, GbE     |  | |
|  |  +----------------------------------------------+  | |
|  +-----------------------------------------------------+ |
|==========================================================|
|                Programmable Logic (PL)                     |
|  +-----------------------------------------------------+ |
|  |                                                       | |
|  |  +------------------+  +-----------------------+       | |
|  |  | AXI Interconnect |  |    GPIO Controller     |       | |
|  |  +------------------+  +-----------------------+       | |
|  |         |                       |                       | |
|  |  +------+------+         +------+------+               | |
|  |  | RX DMA     |         | TX DMA      |               | |
|  |  +------+------+         +------+------+               | |
|  |         |                       |                       | |
|  |  +------+------+         +------+------+               | |
|  |  | ADC Core   |         | DAC Core    |               | |
|  |  +------+------+         +------+------+               | |
|  +-------------------------------------------------------+ |
+----------------------------------------------------------+
                              |              |
                    +---------+--------+  +----------------+
                    |    AD9363        |  |   QSPI Flash   |
                    |  RF Transceiver |  |    32 MB       |
                    +-----------------+  +----------------+
                              |
                    +---------+--------+
                    |   1 GB DDR3     |
                    +-----------------+
```

### 2.2 Memory Map

| Region | Address | Size | Description |
|--------|---------|------|-------------|
| DDR | 0x00000000 | 1 GB | Main DDR3 memory |
| QSPI Flash | 0x00000000 | 32 MB | Boot flash |
| RX DMA | 0x7C400000 | 4 KB | AXI DMAC (RX) |
| TX DMA | 0x7C420000 | 4 KB | AXI DMAC (TX) |
| ADC Core | 0x79020000 | 24 KB | AD9361 ADC interface |
| DAC Core | 0x79024000 | 4 KB | AD9361 DAC/DDS interface |

---

## 3. Boot Chain

```
Power On
    |
    v
Boot ROM (ROM)
    |
    v
FSBL (First Stage Boot Loader)
    - PS7 initialization
    - Clock configuration (CPU/DDR)
    - DDR initialization
    - FPGA bitstream loading
    |
    v
U-Boot
    - Load environment variables
    - Load kernel/DTB
    - Start Linux kernel
    |
    v
Linux Kernel
    - Device tree parsing
    - Driver initialization
    - Mount root filesystem
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

---

## 4. Flash Partition Map

| MTD | Name | Size | Purpose |
|-----|------|------|---------|
| mtd0 | qspi-fsbl-uboot | 1 MB | FSBL + U-Boot |
| mtd1 | qspi-uboot-env | 128 KB | U-Boot environment |
| mtd2 | qspi-nvmfs | 896 KB | JFFS2 persistent storage |
| mtd3 | qspi-linux | 30 MB | Linux image |

> Note: LibreSDR primarily boots from SD card, QSPI is fallback.

---

## 5. FPGA Architecture

### 5.1 Data Path

```
                    RX Path
                    =======
AD9363 (LVDS) --> ADC Interface --> Decimation --> DMA --> DDR --> Linux/IIO

                    TX Path
                    =======
Linux/IIO --> DMA --> Interpolation --> DAC Interface --> AD9363 (LVDS)
```

### 5.2 FPGA Register Map

| Address | Module | Description |
|---------|--------|-------------|
| 0x79020000 | ADC Core | AD9361 ADC interface control |
| 0x79024000 | DAC Core | AD9361 DAC/DDS interface |
| 0x7C400000 | RX DMA | Receiver DMA controller |
| 0x7C420000 | TX DMA | Transmitter DMA controller |

---

## 6. AD936x Configuration

### 6.1 Clock Configuration

| Clock | Frequency | Source |
|-------|-----------|--------|
| Reference Clock | 40 MHz | External TCXO |
| BBPLL | 983.04 MHz | x24.576 multiplier |
| ADC Clock | 245.76 MHz | /4 divider |
| DAC Clock | 122.88 MHz | /8 divider |

### 6.2 Default Settings

| Parameter | Value |
|-----------|-------|
| RX LO Frequency | 2.4 GHz |
| TX LO Frequency | 2.45 GHz |
| RF Bandwidth | 18 MHz |
| Sample Rate | 30.72 MHz |
| AGC Mode | Slow Attack |

---

## 7. Overclocking

### 7.1 Clock Multipliers

| Configuration | CPU Mult | DDR Mult | CPU Speed | DDR Speed |
|---------------|----------|----------|-----------|-----------|
| Base | 30 | 21 | 750 MHz | 525 MHz |
| Medium | 38 | 24 | 950 MHz | 600 MHz |
| High | 44 | 30 | 1100 MHz | 750 MHz |

### 7.2 DDR Timing Parameters

Located in `hdl/projects/libre/system_bd.tcl`:
- `PCW_UIPARAM_DDR_CL`: CAS Latency
- Other timing parameters in same file

### 7.3 Sample Rate Achievements

| Configuration | Continuous Sample Rate |
|---------------|------------------------|
| Stock PlutoSDR | ~10-12 MSPS |
| LibreSDR Base | 20 MSPS |
| LibreSDR Overclocked | 27.5 MSPS |

---

## 8. Software Stack

```
+---------------------------+
|   User Space Applications |
|   - libiio applications     |
|   - pyadi-iio Python API   |
|   - SDR++                  |
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
|   - ad9361 driver           |
|   - axi_ad9361 driver      |
|   - axi_dmac driver         |
+---------------------------+
          |
          v
+---------------------------+
|   FPGA Fabric (PL)         |
+---------------------------+
          |
          v
+---------------------------+
|   AD9363 RF Transceiver    |
+---------------------------+
```

---

## 9. Network Configuration

### 9.1 Default Settings

| Parameter | Value |
|-----------|-------|
| Device IP | 192.168.1.10 |
| Netmask | 255.255.255.0 |
| Gateway | 192.168.1.1 |
| MAC Address | Device-specific |

### 9.2 Services

- SSH (dropbear)
- HTTP (web interface)
- IIO daemon (iiod)
- MQTT (optional, tezuka_fw)

---

## 10. Firmware Update

### 10.1 SD Card Update (Recommended)

1. Download firmware package
2. Extract contents of `sdimg/` folder
3. Copy to FAT32-formatted SD card
4. Insert into LibreSDR and power on
5. System boots from SD card

### 10.2 QSPI Update

1. Boot from SD card
2. SD card auto-mounts as USB drive
3. Copy `.frm` files to mounted drive
4. Device updates QSPI flash
5. Can boot without SD card afterwards

---

## 11. Firmware File Structures

### 11.1 SD Card Contents

```
/SD Card (FAT32)/
|-- BOOT.BIN          # Zynq boot image (FSBL + bitstream + U-Boot)
|-- image.ub           # Linux kernel + DTB + rootfs (FIT image)
|-- system_top.bit     # FPGA bitstream
|-- devicetree.dtb     # Device tree blob
|-- boot.scr          # U-Boot boot script
|-- uEnv.txt          # U-Boot environment
```

### 11.2 boot.frm / pluto.frm

Similar to PlutoSDR firmware format:
- MD5 checksum for verification
- DFU update support

---

## 12. Board Comparison

| Feature | LibreSDR | PlutoSDR | Notes |
|---------|----------|----------|-------|
| FPGA | Zynq-7020 | Zynq-7010 | 3x more logic |
| DDR | 1 GB | 512 MB | 2x more |
| Ethernet | GbE | 100M | 10x faster |
| Boot | SD + QSPI | QSPI | SD safer |
| Default IP | 192.168.1.10 | 192.168.2.1 | Different subnet |
| Overclock | Yes | No | LibreSDR advantage |
| Price | Similar | Similar | |

---

## 13. Key Files and Sources

### 13.1 Build

| File | Source | Description |
|------|--------|-------------|
| BOOT.BIN | bootgen | FSBL + bitstream + U-Boot |
| image.ub | mkimage | FIT image |
| zImage | Linux build | Linux kernel |
| *.dtb | DTC | Device tree |

### 13.2 Runtime

| File | Location | Description |
|------|----------|-------------|
| VERSIONS | /opt/ | Build information |
| config.txt | /boot/ or USB drive | Runtime settings |
| index.html | /www/ | Web interface |

---

## 14. Known Limitations

1. **USB 2.0 Bandwidth**: Maximum ~40 MB/s data transfer
2. **FPGA Modifications**: Requires Vivado for bitstream generation
3. **Documentation**: Limited official documentation
4. **Overclocking Risk**: May cause instability on some boards

---

## 15. Verification Commands

```bash
# Device info
cat /proc/cpuinfo
cat /proc/mtd
fw_printenv

# IIO devices
iio_info
iio_attr -C

# Network
ifconfig -a
cat /etc/network/interfaces

# Filesystem
df -h
mount
ls -la /www

# Firmware version
cat /opt/VERSIONS
cat /proc/cmdline
```

---

## 16. Document Cross-References

| Document | Topic |
|----------|-------|
| `LIBRESDR_SYSTEM_ARCHITECTURE.md` | System overview |
| `LIBRESDR_HARDWARE_SPECIFICATIONS.md` | Detailed hardware specs |
| `LIBRESDR_BOOT_FLOW.md` | Boot sequence |
| `LIBRESDR_FIRMWARE_FORMATS.md` | Firmware format analysis |
| `LIBRESDR_FIRMWARE_ANALYSIS.md` | Firmware binary analysis |
| `LIBRESDR_FLASH_LAYOUT.md` | Flash memory layout |
| `LIBRESDR_FPGA_ARCHITECTURE.md` | FPGA HDL architecture |
| `LIBRESDR_FPGA_BITSTREAM.md` | FPGA bitstream analysis |
| `LIBRESDR_AD936X_CONFIG.md` | AD936x configuration |
| `LIBRESDR_DEVICE_TREE.md` | Device tree mapping |
| `LIBRESDR_IIO_ARCHITECTURE.md` | IIO subsystem |
| `LIBRESDR_ROOTFS_ANALYSIS.md` | Root filesystem analysis |
| `LIBRESDR_WEB_ARCHITECTURE.md` | Web interface |
| `LIBRESDR_UBOOT_BOOT_CUSTOMIZATION.md` | U-Boot configuration |
| `LIBRESDR_JTAG_BOOTSTRAP.md` | JTAG bootstrap |
| `LIBRESDR_OVERCLOCKING.md` | Overclocking guide |
| `LIBRESDR_PERFORMANCE.md` | Performance data |
| `LIBRESDR_REVISION_DIFFERENCES.md` | Hardware variants |
| `LIBRESDR_BOARD_COMPARISON.md` | Board comparison |

---

## 17. Research Status

### 17.1 Verified Information

| Area | Status |
|------|--------|
| Hardware specs | VERIFIED |
| Boot sequence | VERIFIED |
| Overclocking | VERIFIED |
| Firmware formats | VERIFIED |
| Software stack | VERIFIED |

### 17.2 Limited Access

| Area | Status | Note |
|------|--------|------|
| FPGA HDL source | PARTIAL | Patches available |
| Complete schematics | LIMITED | Partial PDF only |
| Runtime testing | PENDING | No hardware access |

---

## 18. Important Engineering Questions

### Q: What are the main advantages over PlutoSDR?
**A:**
- Larger FPGA (3x more logic)
- 2x more DDR memory
- 10x faster Ethernet
- SD card boot (safer updates)
- Overclocking support

### Q: What sample rates can be achieved?
**A:**
- Base: 20 MSPS continuous
- Overclocked: 27.5 MSPS continuous
- Limiting factor is often host computer Ethernet/USB

### Q: How does overclocking work?
**A:**
1. Modify CPU/DDR clock multipliers in device tree
2. Adjust DDR timing parameters in HDL
3. Rebuild FSBL with new clock settings
4. Test for stability

### Q: What is the boot priority?
**A:**
1. SD Card (primary, recommended)
2. QSPI Flash (fallback, populated by SD boot)

---

## 19. Version Information

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-09-07 | Initial knowledge base |

---

*This document is prepared as an engineering-level reference for the LibreSDR platform.*
