# LIBRESDR U-BOOT BOOT CUSTOMIZATION

## Engineering Reference - Boot Loader Configuration

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | BOOT LOADER ANALYSIS |
| **Last Updated** | 2026-09-07 |
| **Source** | tezuka_fw board/tezuka/libre/ |

---

## 1. U-Boot Configuration

### 1.1 Configuration File

- **Config**: `libre_maiasdr_defconfig`
- **Board Config**: `board/tezuka/libre/u-boot-config/`

### 1.2 Key Configuration Options

```c
CONFIG_TEXT_BASE=0x4000000      // Load address
CONFIG_SF_DEFAULT_SPEED=104000000 // QSPI speed (104 MHz)
CONFIG_ENV_OFFSET=0x100000       // Environment offset
CONFIG_DEFAULT_DEVICE_TREE="zynq-libre"
CONFIG_BOOTDELAY=0              // No boot delay
CONFIG_BOOTCOMMAND="run $modeboot"
CONFIG_SYS_PROMPT="libre> "
```

---

## 2. Memory Layout

### 2.1 Address Map

| Address | Size | Purpose |
|---------|------|---------|
| 0x04000000 | - | U-Boot load address |
| 0x100000 | 1 MB | QSPI FSBL+U-Boot offset |
| 0x200000 | 2 MB | FIT image load address |
| 0x10000000 | - | SPL load address |
| 0x2000000 | 32 MB | DTB load address |
| 0x2080000 | - | FIT load address |

---

## 3. Environment Variables

### 3.1 Default Environment

```c
"ethaddr=00:0a:35:00:01:23\0"     // MAC address
"ipaddr=192.168.1.10\0"             // LibreSDR IP
"ipaddr_host=192.168.1.1\0"        // Host IP
"netmask=255.255.255.0\0"          // Netmask
"kernel_image=uImage\0"              // Kernel image name
"fit_load_address=0x2080000\0"      // FIT address
"fit_config=config@0\0"            // Default FIT config
"devicetree_image=devicetree.dtb\0" // DTB name
"boot_image=BOOT.bin\0"             // Boot image
```

### 3.2 Boot Mode Variables

```c
"mode=1r1t\0"           // RF mode: 1R1T or 2R2T
"refclk_source=internal\0" // Reference clock source
"attr_val=ad9363a\0"     // RF transceiver type
```

---

## 4. Boot Commands

### 4.1 Boot Flow

```c
"modeboot=sdboot\0"    // Default boot mode (SD card)
```

### 4.2 SD Card Boot Sequence

```c
"sdboot=" \
    "load mmc 0:1 0x3000000 image.ub && "  // Load FIT from SD
    "bootm 0x3000000"                        // Boot from memory
```

### 4.3 QSPI Boot Sequence

```c
"qspiboot=" \
    "sf probe && "                    // Initialize QSPI
    "sf read ${fit_load_address} 0x200000 ${fit_size} && " // Read FIT
    "bootm ${fit_load_address}#${fit_config}"
```

---

## 5. Clock Configuration

### 5.1 LibreSDR Default Clocks

| Clock | Default | Overclock |
|-------|---------|-----------|
| CPU | 750 MHz | 1100 MHz |
| DDR | 525 MHz | 750 MHz |

### 5.2 Clock Multipliers

```c
"OVERCLOCK_CPU_MULT=30\0"    // Default: 30 * 25 = 750 MHz
"OVERCLOCK_DDR_MULT=21\0"    // Default: 21 * 25 = 525 MHz
```

---

## 6. DFU Configuration

### 6.1 DFU ALT Info

```c
#define DFU_ALT_INFO_SF
    "dfu_sf_info=" \
    "boot.dfu raw 0x0 0x100000\\;"      // Boot image (1 MB)
    "firmware.dfu raw 0x200000 0x1E00000\\;"  // Firmware (30 MB)
    "uboot-env.dfu raw 0x100000 0x20000\\;"  // Environment
    "spare.dfu raw 0x120000 0xE0000"       // Spare (JFFS2)
```

### 6.2 DFU Flash Addresses

| DFU Target | Flash Offset | Size |
|------------|--------------|------|
| boot.dfu | 0x0 | 1 MB |
| firmware.dfu | 0x200000 | 30 MB |
| uboot-env.dfu | 0x100000 | 128 KB |
| spare.dfu | 0x120000 | 896 KB |

---

## 7. Network Configuration

### 7.1 Default Network Settings

```bash
ipaddr=192.168.1.10        # LibreSDR IP
serverip=192.168.1.1      # Host IP
netmask=255.255.255.0
gatewayip=192.168.1.1
```

### 7.2 Gigabit Ethernet

LibreSDR supports Gigabit Ethernet, unlike PlutoSDR's 100M.

---

## 8. Boot Flow Diagram

```
BootROM
    |
    v
Check boot mode (MIO pins)
    |
    +-- SD Card detected -> SD boot
    |
    +-- QSPI fallback -> QSPI boot
    |
    v
FSBL (loads U-Boot to 0x04000000)
    |
    v
U-Boot Start
    |
    v
Load image.ub from boot source
    |
    v
bootm (load kernel + DTB + rootfs)
    |
    v
Boot Linux
```

---

## 9. Customization Variables

### 9.1 User-Settable Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `ipaddr` | 192.168.1.10 | Device IP address |
| `ipaddr_host` | 192.168.1.1 | Host IP address |
| `netmask` | 255.255.255.0 | Network mask |
| `mode` | 1r1t | RF mode |
| `refclk_source` | internal | Clock source |

### 9.2 Overclock Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `OVERCLOCK_CPU_MULT` | 30 | CPU multiplier |
| `OVERCLOCK_DDR_MULT` | 21 | DDR multiplier |

---

## 10. Recovery Boot

### 10.1 SD Card Recovery

1. Power off
2. Insert SD card with firmware
3. Power on
4. Device boots from SD card

### 10.2 DFU Recovery

```bash
# Via serial console
run dfu_sf
```

---

## 11. uEnv.txt Configuration

### 11.1 Sample uEnv.txt

```
ipaddr=192.168.1.10
serverip=192.168.1.1
gatewayip=192.168.1.1
netmask=255.255.255.0
hostname=libre
```

### 11.2 Boot Override

```
bootcmd=run sdboot
```

---

## 12. Comparison with PlutoSDR

| Feature | LibreSDR | PlutoSDR |
|---------|----------|----------|
| Default Boot | SD Card | QSPI |
| Default IP | 192.168.1.10 | 192.168.2.1 |
| Ethernet | Gigabit | 100 Mbps |
| CPU Speed | 750 MHz | 666 MHz |
| Overclock | Supported | Not supported |

---

## 13. Related Documents

- `LIBRESDR_BOOT_FLOW.md` - Boot sequence overview
- `LIBRESDR_FIRMWARE_FORMATS.md` - Firmware formats
- `LIBRESDR_FLASH_LAYOUT.md` - Flash layout
- `LIBRESDR_DEVICE_TREE.md` - Device tree
- `LIBRESDR_OVERCLOCKING.md` - Overclocking guide
