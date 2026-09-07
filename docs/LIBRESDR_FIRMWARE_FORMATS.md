# LIBRESDR FIRMWARE FORMATS

## Firmware File Structures and Analysis

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | FIRMWARE FORMATS |
| **Last Updated** | 2026-09-07 |
| **Source** | lucasmellon-ops/LibreSDR, F5OEO/tezuka_fw |

---

## 1. Firmware Sources

### 1.1 Available Firmware Files

From `lucasmellon-ops/LibreSDR/Firmware&deviceFiles/Working-fw/`:

| File | Size | Purpose |
|------|------|---------|
| BOOT.bin | 2.8 MB | Zynq boot image |
| boot.frm | 618 KB | Boot update package |
| boot.dfu | 486 KB | Boot DFU update |
| pluto.frm | 12.2 MB | System update package |
| pluto.dfu | 12.2 MB | System DFU update |
| uboot-env.dfu | 131 KB | U-Boot env update |
| uImage | 5.6 MB | Linux kernel |
| uramdisk.image.gz | 7.7 MB | Compressed rootfs |
| ramdisk.image.gz | 7.7 MB | Uncompressed rootfs |
| system_top.bit | 2.3 MB | FPGA bitstream |
| devicetree.dtb | 22 KB | Device tree blob |
| uEnv.txt | 7.6 KB | U-Boot environment |
| boot.bif | 250 B | Boot image description |

---

## 2. BOOT.BIN Structure

### 2.1 Components

```
+------------------+
| Zynq Boot Header |
+------------------+
| FSBL (ELF)       |
+------------------+
| FPGA Bitstream   |
+------------------+
| U-Boot (ELF)     |
+------------------+
```

### 2.2 Boot Image (BIF)

```bif
/* boot.bif */
the_ROM_image:
{
    [bootloader]fsbl.elf
    system_top.bit
    u-boot.elf
}
```

---

## 3. Firmware Update Packages

### 3.1 boot.frm

Structure:
```
+------------------+
|   boot.bin       |  (~618 KB)
+------------------+
|   uboot-env.bin  |  (128 KB)
+------------------+
|   MD5 checksum   |  (32 bytes)
+------------------+
```

### 3.2 pluto.frm

Structure:
```
+------------------+
|   pluto.itb      |  (~12 MB)
+------------------+
|   MD5 checksum   |  (32 bytes)
+------------------+
```

### 3.3 pluto.itb (FIT Image)

```
+------------------+
|   FIT Header     |
+------------------+
|   DTB (RevA)     |
+------------------+
|   DTB (RevB)     |
+------------------+
|   DTB (RevC)     |
+------------------+
|   FPGA bitstream |
+------------------+
|   zImage         |
+------------------+
|   rootfs.cpio   |
+------------------+
|   Hash Table     |
+------------------+
```

---

## 4. SD Card Image

### 4.1 Required Files

```
/SD Card/
|-- BOOT.BIN          # FSBL + bitstream + U-Boot
|-- image.ub          # Kernel + DTB + rootfs (FIT)
|-- system_top.bit    # FPGA bitstream (optional)
|-- devicetree.dtb    # Device tree
|-- boot.scr          # U-Boot script
|-- uEnv.txt          # U-Boot environment
```

### 4.2 image.ub (Combined FIT)

```bash
# Create FIT image
mkimage -f image.its image.ub
```

---

## 5. File Sizes

### 5.1 LibreSDR vs PlutoSDR

| File | LibreSDR | PlutoSDR | Difference |
|------|----------|----------|------------|
| BOOT.BIN | 2.8 MB | 572 KB | Larger FPGA |
| system_top.bit | 2.3 MB | 943 KB | Larger FPGA |
| pluto.frm | 12.2 MB | ~14 MB | Similar |

---

## 6. Checksums

### 6.1 MD5 Verification

The `.frm` files include MD5 checksums:
- Last 32 bytes of file
- Format: `md5:<64-hex-digits>`

### 6.2 Verification Process

```bash
# Verify boot.frm
md5sum boot.frm
# Compare with embedded checksum
```

---

## 7. Firmware Build

### 7.1 tezuka_fw Build Output

```bash
./build.sh libre
# Output in: output/libre/images/
```

### 7.2 Generated Files

```
output/libre/images/
|-- BOOT.BIN
|-- image.ub
|-- uEnv.txt
|-- boot.scr
|-- system_top.bit
|-- pluto.frm
|-- boot.frm
|-- *.dfu
```

---

## 8. Update Methods

### 8.1 SD Card Update (Recommended)

1. Download firmware package
2. Extract sdimg contents
3. Copy to FAT32 SD card
4. Insert and boot

### 8.2 USB MSD Update

1. Boot device with SD card
2. Device appears as USB drive
3. Copy .frm files to drive
4. Device updates and reboots

### 8.3 DFU Update

```bash
# Using dfu-util
dfu-util -D boot.frm -a boot
dfu-util -D pluto.frm -a rootfs
```
