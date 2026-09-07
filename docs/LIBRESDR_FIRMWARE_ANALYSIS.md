# LIBRESDR FIRMWARE ANALYSIS

## Engineering Reference - Firmware Binary Analysis

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | FIRMWARE_ANALYSIS |
| **Last Updated** | 2026-09-07 |
| **Source** | tezuka_fw, LibreSDR working-fw |

---

## 1. Firmware Sources

### 1.1 Available Firmware Files

From `LibreSDR/Firmware&deviceFiles/Working-fw/`:

| File | Size | Description |
|------|------|-------------|
| BOOT.bin | 2.8 MB | Zynq boot image |
| boot.frm | 618 KB | Boot update package |
| boot.dfu | 486 KB | Boot DFU update |
| pluto.frm | 12.2 MB | System update package |
| pluto.dfu | 12.2 MB | System DFU update |
| uboot-env.dfu | 131 KB | U-Boot env update |
| uImage | 5.6 MB | Linux kernel |
| uramdisk.image.gz | 7.7 MB | Compressed rootfs |
| system_top.bit | 2.3 MB | FPGA bitstream |
| devicetree.dtb | 22 KB | Device tree |
| uEnv.txt | 7.6 KB | U-Boot environment |
| boot.bif | 250 B | Boot image description |

### 1.2 tezuka_fw Build Output

Generated in `output/libre/images/`:

| File | Size | Description |
|------|------|-------------|
| BOOT.BIN | ~2-3 MB | Boot image (FSBL+bitstream+U-Boot) |
| image.ub | ~15 MB | FIT image (kernel+dtb+rootfs) |
| pluto.frm | ~12 MB | Flash update package |
| boot.frm | ~600 KB | Boot update |
| sdimg/ | - | SD card contents |

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

### 2.2 boot.bif Content

```bif
/* boot.bif for LibreSDR */
the_ROM_image:
{
    [bootloader]fsbl.elf
    system_top.bit
    u-boot.elf
}
```

### 2.3 Build Commands

```bash
# With tezuka_fw
./build.sh libre

# Or manually with bootgen
bootgen -image boot.bif -w -o BOOT.BIN
```

---

## 3. Firmware Update Packages

### 3.1 boot.frm Structure

```
+------------------+
|   boot.bin       |  (~618 KB)
+------------------+
|   uboot-env.bin  |  (128 KB)
+------------------+
|   MD5 checksum   |  (32 bytes)
+------------------+
```

### 3.2 pluto.frm Structure

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
|   DTB            |
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
/SD Card (FAT32)/
|-- BOOT.BIN          # FSBL + bitstream + U-Boot
|-- image.ub          # Kernel + DTB + rootfs (FIT)
|-- uEnv.txt          # U-Boot environment
|-- boot.scr          # U-Boot script (optional)
```

### 4.2 image.ub Creation

```bash
# Create FIT image
mkimage -f image.its image.ub

# image.its content
/dts-v1/;
/ {
    description = "LibreSDR Linux";
    images {
        kernel@1 {
            data = /incbin/("zImage");
            type = "kernel";
            arch = "arm";
            os = "linux";
            load = <0x8000>;
            entry = <0x8000>;
        };
        fdt@1 {
            data = /incbin/("devicetree.dtb");
            type = "flat_dt";
            arch = "arm";
            load = <0x2000000>;
        };
        ramdisk@1 {
            data = /incbin/("rootfs.cpio.gz");
            type = "ramdisk";
            arch = "arm";
            os = "linux";
            load = <0x4000000>;
        };
    };
    configurations {
        default = "conf@1";
        conf@1 {
            kernel = "kernel@1";
            fdt = "fdt@1";
            ramdisk = "ramdisk@1";
        };
    };
};
```

---

## 5. File Sizes Comparison

### 5.1 LibreSDR vs PlutoSDR

| File | LibreSDR | PlutoSDR | Difference |
|------|----------|----------|------------|
| BOOT.BIN | ~2.8 MB | ~572 KB | Larger FPGA |
| system_top.bit | ~2.3 MB | ~943 KB | Larger FPGA |
| pluto.frm | ~12 MB | ~14 MB | Similar |
| uImage | ~5.6 MB | ~4.1 MB | Kernel differences |

---

## 6. Checksums

### 6.1 MD5 Verification

The `.frm` files include MD5 checksums:
- Last 32 bytes of file
- Format: `md5:<64-hex-digits>`

### 6.2 Known Checksums

```
plutodvb-libresdr-boot.frm: md5:...
plutodvb-libresdr.frm: md5:...
```

---

## 7. FPGA Bitstream Details

### 7.1 Bitstream Properties

| Property | Value |
|----------|-------|
| File | system_top.bit |
| Size | ~2.3 MB |
| Target | 7z020clg400 (Zynq-7020) |
| Build | tezuka_fw |

### 7.2 FPGA Resources

| Resource | LibreSDR | PlutoSDR |
|----------|-----------|----------|
| Device | XC7Z020 | XC7Z010 |
| LUTs | 53,200 | 17,600 |
| BRAM | 4.9 Mb | 2.1 Mb |

---

## 8. U-Boot Environment

### 8.1 Key Variables

```bash
# Network
ipaddr=192.168.1.10
serverip=192.168.1.1
gatewayip=192.168.1.1
netmask=255.255.255.0

# Boot
bootcmd=run sdboot
mode=1r1t

# FIT
fit_load_address=0x2080000
fit_size=0x900000
```

### 8.2 U-Boot Environment File

```bash
# uEnv.txt for SD card boot
ipaddr=192.168.1.10
serverip=192.168.1.1
gatewayip=192.168.1.1
netmask=255.255.255.0
hostname=libre
```

---

## 9. tezuka_fw Build Process

### 9.1 Build Flow

```bash
# 1. Configure Buildroot
make libre_maiasdr_defconfig

# 2. Build rootfs
make

# 3. Post-image script generates firmware
# - Creates pluto.frm, boot.frm
# - Creates SD card image
```

### 9.2 Output Structure

```
output/libre/images/
|-- BOOT.BIN           # For SD card / QSPI
|-- image.ub           # FIT image
|-- pluto.frm         # Flash update
|-- boot.frm          # Boot update
|-- *.dfu             # DFU files
|-- sdimg/             # SD card contents
|   |-- BOOT.BIN
|   |-- image.ub
|   |-- uEnv.txt
|-- build/libre.zip   # Release package
```

---

## 10. Firmware Update Methods

### 10.1 SD Card Update (Recommended)

1. Download firmware package
2. Extract sdimg contents
3. Copy to FAT32 SD card
4. Insert and power on
5. System boots from new firmware

### 10.2 QSPI Flash Update

1. Boot with SD card
2. Device appears as USB drive
3. Copy .frm files to drive
4. Device updates and reboots

### 10.3 DFU Update

```bash
# Using dfu-util
dfu-util -D boot.frm -a boot
dfu-util -D pluto.frm -a rootfs
```

---

## 11. Related Documents

- `LIBRESDR_FIRMWARE_FORMATS.md` - General firmware formats
- `LIBRESDR_BOOT_FLOW.md` - Boot sequence
- `LIBRESDR_DEVICE_TREE.md` - Device tree
- `LIBRESDR_FLASH_LAYOUT.md` - Flash layout
- `LIBRESDR_BUILD_SYSTEM.md` - Build system
