# LIBRESDR FLASH LAYOUT

## QSPI Flash Memory Map

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | FLASH LAYOUT |
| **Last Updated** | 2026-09-07 |

---

## 1. Flash Overview

LibreSDR uses a 32 MB QSPI NOR flash for persistent storage.

### 1.1 Flash Parameters

| Parameter | Value |
|-----------|-------|
| Type | QSPI NOR Flash |
| Size | 32 MB |
| Interface | SPI x4 |
| Speed | 104 MHz |

---

## 2. Memory Map

### 2.1 MTD Partition Layout

| MTD | Name | Offset | Size | Purpose |
|-----|------|--------|------|---------|
| mtd0 | qspi-fsbl-uboot | 0x00000000 | 1 MB | FSBL + U-Boot |
| mtd1 | qspi-uboot-env | 0x00100000 | 128 KB | U-Boot environment |
| mtd2 | qspi-nvmfs | 0x00120000 | 896 KB | JFFS2 filesystem |
| mtd3 | qspi-linux | 0x00200000 | 30 MB | Linux FIT image |

### 2.2 Detailed Offsets

```
Flash Address Map (32 MB):
+---------------------------+ 0x00000000
|                           |
|  FSBL + U-Boot           | 1 MB
|  (BOOT.BIN)              |
|                           |
+---------------------------+ 0x00100000
|                           |
|  U-Boot Environment       | 128 KB
|                           |
+---------------------------+ 0x00120000
|                           |
|  JFFS2 Filesystem        | 896 KB
|  (Persistent storage)    |
|                           |
+---------------------------+ 0x00200000
|                           |
|  Linux Image (FIT)        | 30 MB
|  pluto.itb               |
|                           |
+---------------------------+ 0x02000000
|                           |
|  (Reserved/Empty)         |
|                           |
+---------------------------+ 0x02000000 (32 MB)
```

---

## 3. Partition Details

### 3.1 Boot Partition (mtd0)

| Parameter | Value |
|-----------|-------|
| Offset | 0x00000000 |
| Size | 1,048,576 bytes (1 MB) |
| Contents | BOOT.BIN (FSBL + bitstream + U-Boot) |
| Write Size | 4-byte aligned |

### 3.2 Environment Partition (mtd1)

| Parameter | Value |
|-----------|-------|
| Offset | 0x00100000 |
| Size | 131,072 bytes (128 KB) |
| Contents | U-Boot environment variables |
| Format | Key-value pairs |

### 3.3 Persistent Storage (mtd2)

| Parameter | Value |
|-----------|-------|
| Offset | 0x00120000 |
| Size | 917,504 bytes (896 KB) |
| Filesystem | JFFS2 |
| Mount Point | /mnt/jffs2 |
| Contents | config.txt, calibration data |

### 3.4 Linux Partition (mtd3)

| Parameter | Value |
|-----------|-------|
| Offset | 0x00200000 |
| Size | 31,457,280 bytes (30 MB) |
| Contents | pluto.itb (FIT image) |
| Format | FIT (Flat Image Tree) |

---

## 4. FIT Image Contents

### 4.1 pluto.itb Structure

```
pluto.itb (FIT Image):
+------------------+
|    FIT Header     |
+------------------+
|   DTB (RevA)     |
+------------------+
|   DTB (RevB)     |
+------------------+
|   DTB (RevC)     |
+------------------+
|   FPGA bitstream |
+------------------+
|   Linux Kernel   |
|   (zImage)       |
+------------------+
|   Root Filesystem|
|   (rootfs.cpio)  |
+------------------+
|   Hash Table     |
+------------------+
```

---

## 5. Boot Process

### 5.1 Flash Boot Sequence

1. Boot ROM reads mtd0
2. FSBL executes
3. U-Boot loads pluto.itb from mtd3
4. Kernel boots from FIT image

### 5.2 U-Boot Commands

```bash
# Load from flash
sf probe 0
sf read 0x3000000 0x200000 0x1e00000

# Boot from memory
bootm 0x3000000
```

---

## 6. Updating Flash

### 6.1 Update Methods

| Method | Tool | Target MTD |
|--------|------|------------|
| DFU | dfu-util | Any |
| U-Boot | sf | Any |
| Linux | mtd-utils | Any |

### 6.2 DFU Update

```bash
# Update boot partition
dfu-util -D boot.frm -a boot

# Update rootfs
dfu-util -D pluto.frm -a rootfs

# Update U-Boot env
dfu-util -D uboot-env.dfu -a uboot-env
```

### 6.3 U-Boot Update

```bash
# Update boot partition
sf probe 0
sf erase 0 0x100000
sf write 0x100000 0x0 0x100000

# Update Linux partition
sf erase 0x200000 0x1e00000
sf write <addr> 0x200000 <size>
```

---

## 7. Persistent Storage

### 7.1 JFFS2 Filesystem

Mounted at `/mnt/jffs2` with contents:

```
/mnt/jffs2/
|-- config.txt           # Runtime configuration
|-- calibration/        # RF calibration data
|-- wlan/              # WiFi config (if supported)
```

### 7.2 config.txt

```
# LibreSDR Configuration
# Stored in JFFS2 for persistence
```

---

## 8. Comparison with PlutoSDR

| Parameter | LibreSDR | PlutoSDR |
|-----------|----------|----------|
| Flash Size | 32 MB | 32 MB |
| Boot Partition | 1 MB | 1 MB |
| Env Partition | 128 KB | 128 KB |
| Storage | 896 KB | 896 KB |
| Linux | 30 MB | 30 MB |

> Note: Flash layout is nearly identical to PlutoSDR.

---

## 9. Recovery

### 9.1 JTAG Boot

If flash is corrupted:
1. Set boot mode to JTAG
2. Load FSBL via JTAG
3. Reflash boot partition
4. Reflash Linux partition

### 9.2 SD Card Recovery

1. Boot from SD card
2. Use `dd` or `flashcp` to rewrite flash
3. Reboot without SD card
