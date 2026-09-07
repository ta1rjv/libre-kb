# LIBRESDR BOOT FLOW

## Boot Sequence and Initialization

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | BOOT FLOW |
| **Last Updated** | 2026-09-07 |

---

## 1. Boot Overview

LibreSDR supports multiple boot modes with SD card as the primary method:

1. **SD Card Boot** (Primary) - FAT32 formatted SD card
2. **QSPI Flash Boot** (Fallback) - Pre-programmed flash

---

## 2. Boot Sequence

```
+---------------------------+
|        Power On            |
+---------------------------+
            |
            v
+---------------------------+
|     Boot Mode Select       |
|  (MIO pins / DIP switch)  |
+---------------------------+
            |
            v
+---------------------------+
|        Boot ROM            |
|   (Internal to Zynq)       |
+---------------------------+
            |
            v
+---------------------------+
|   Load FSBL from boot      |
|         source              |
+---------------------------+
            |
            v
+---------------------------+
|   FSBL Execution           |
|  - Initialize clocks       |
|  - Initialize DDR          |
|  - Initialize MIO/EMIO     |
|  - Load FPGA bitstream     |
+---------------------------+
            |
            v
+---------------------------+
|   Load U-Boot              |
+---------------------------+
            |
            v
+---------------------------+
|   U-Boot Execution         |
|  - Load kernel/DTB         |
|  - Set boot arguments      |
+---------------------------+
            |
            v
+---------------------------+
|   Load Linux Kernel        |
+---------------------------+
            |
            v
+---------------------------+
|   Kernel Initialization    |
|  - Parse device tree       |
|  - Mount root filesystem   |
+---------------------------+
            |
            v
+---------------------------+
|   Run Init Scripts         |
|  - Network setup           |
|  - IIO daemon              |
|  - Web server              |
+---------------------------+
            |
            v
+---------------------------+
|      Runtime Ready         |
+---------------------------+
```

---

## 3. SD Card Boot

### 3.1 SD Card Structure

```
/SD Card (FAT32)/
|-- BOOT.BIN          # Zynq boot image (FSBL + bitstream + U-Boot)
|-- image.ub           # FIT image (kernel + dtb + rootfs)
|-- system_top.bit     # FPGA bitstream (optional, in BOOT.BIN)
|-- devicetree.dtb     # Device tree (optional, in image.ub)
|-- boot.scr          # U-Boot script
|-- uEnv.txt          # Environment variables
```

### 3.2 BOOT.BIN Structure

```
+------------------+
| Zynq Boot Header |
+------------------+
| FSBL (1st stage) |
+------------------+
| FPGA bitstream   |
+------------------+
| U-Boot (2nd stage)|
+------------------+
```

### 3.3 image.ub Structure (FIT Image)

```
+------------------+
|    FIT Header    |
+------------------+
|  Device Tree    |
+------------------+
|   FPGA bitstream |
+------------------+
|   Linux Kernel  |
+------------------+
|   Rootfs        |
+------------------+
|   Hash Table     |
+------------------+
```

---

## 4. QSPI Flash Boot

### 4.1 Flash Layout

| Offset | Size | Content |
|--------|------|---------|
| 0x000000 | 1 MB | FSBL + U-Boot |
| 0x100000 | 128 KB | U-Boot environment |
| 0x120000 | 896 KB | JFFS2 filesystem |
| 0x200000 | 30 MB | Linux image |

### 4.2 Fallback Mechanism

If SD card is not present or boot fails:
1. Boot ROM attempts QSPI boot
2. Loads FSBL from flash offset 0
3. FSBL loads U-Boot
4. U-Boot loads kernel from flash

---

## 5. FSBL Details

### 5.1 Initialization Sequence

1. **Clock Configuration**
   - Set CPU clock (default 750 MHz)
   - Set DDR clock (default 525 MHz)
   - Configure PLLs

2. **DDR Initialization**
   - Configure DDR controller
   - Memory training
   - Calibration

3. **MIO Configuration**
   - Ethernet pins
   - SD card pins
   - USB pins
   - UART pins

4. **FPGA Loading**
   - Load bitstream from boot source
   - Configure programmable logic

### 5.2 Clock Settings

| Clock | Default | Overclock |
|-------|---------|-----------|
| CPU | 750 MHz | 1100 MHz |
| DDR | 525 MHz | 750 MHz |
| PL | 100 MHz | 100 MHz |

---

## 6. U-Boot Details

### 6.1 U-Boot Responsibilities

1. Load kernel image
2. Set kernel arguments
3. Pass device tree
4. Start kernel

### 6.2 Environment Variables

| Variable | Default Value | Description |
|----------|--------------|-------------|
| ipaddr | 192.168.1.10 | Device IP |
| serverip | 192.168.1.1 | Host IP |
| netmask | 255.255.255.0 | Network mask |
| bootcmd | load image.ub... | Boot command |
| bootargs | console=... | Kernel args |

### 6.3 Boot Commands

```bash
# Load kernel from SD card
load mmc 0:1 0x3000000 image.ub

# Boot from FIT image
bootm 0x3000000
```

---

## 7. Kernel Boot

### 7.1 Kernel Arguments

```bash
console=ttyPS0,115200 root=/dev/ram rw ip=192.168.1.10::192.168.1.1:255.255.255.0
```

### 7.2 Device Tree

The device tree configures:
- CPU configuration
- Memory layout
- Peripherals
- RF chip (AD9363)
- Network interfaces

---

## 8. Init Scripts

### 8.1 Startup Sequence

| Order | Script | Purpose |
|-------|--------|---------|
| S01 | syslogd | System logging |
| S02 | klogd | Kernel logging |
| S10 | network | Network setup |
| S20 | iiod | IIO daemon |
| S50 | dropbear | SSH server |
| S98 | autostart | User startup |

### 8.2 Network Configuration

```bash
# /etc/init.d/S10network
ifconfig eth0 192.168.1.10 netmask 255.255.255.0 up
```

---

## 9. Recovery

### 9.1 Boot Recovery

1. Power off device
2. Insert SD card with known-good firmware
3. Power on
4. Device boots from SD card
5. Can reflash QSPI if needed

### 9.2 Serial Console

Connect via serial at 115200 8N1:
- /dev/ttyUSB2 (typically)
- Can access U-Boot and recovery options

---

## 10. Boot Time

| Phase | Duration |
|-------|----------|
| Boot ROM | ~100ms |
| FSBL | ~1s |
| U-Boot | ~1s |
| Kernel | ~3s |
| Init | ~2s |
| **Total** | **~7s** |
