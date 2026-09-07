# LIBRESDR JTAG BOOTSTRAP

## Engineering Reference - JTAG Bootstrap and Recovery

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | JTAG_BOOTSTRAP |
| **Last Updated** | 2026-09-07 |

---

## 1. JTAG Overview

### 1.1 JTAG Purpose

JTAG (Joint Test Action Group) is used for:
- Initial firmware programming
- Device recovery (bricked devices)
- Debugging
- FPGA bitstream loading

### 1.2 LibreSDR JTAG Support

LibreSDR has a JTAG header for:
- FPGA programming
- CPU debugging
- Initial boot loader loading

---

## 2. JTAG Pinout

### 2.1 Header Pinout (Standard 14-pin)

| Pin | Function | Description |
|-----|----------|-------------|
| 1 | VTREF | Reference voltage |
| 2 | TMS | Test Mode Select |
| 3 | GND | Ground |
| 4 | TDI | Test Data In |
| 5 | GND | Ground |
| 6 | TDO | Test Data Out |
| 7 | GND | Ground |
| 8 | TCK | Test Clock |
| 9 | GND | Ground |
| 10 | RTCK | Return Test Clock |
| 11 | GND | Ground |
| 12 | TRST | Test Reset |
| 13 | GND | Ground |
| 14 | NC | Not Connected |

---

## 3. JTAG Bootstrap Flow

### 3.1 Recovery Process

When the device is bricked or for initial programming:

1. Connect JTAG debugger to LibreSDR
2. Run XMD/Vivado with bootstrap script
3. Load U-Boot into RAM via JTAG
4. Execute U-Boot from RAM
5. Use U-Boot commands to program flash

### 3.2 Bootstrap Script

```tcl
connect arm hw
stop
xreset 64

source ps7_init.tcl
ps7_init
ps7_post_config

dow u-boot.elf
run
disconnect 64
```

---

## 4. ps7_init.tcl Script

### 4.1 Purpose

Initializes Zynq Processing System:
- PLL configuration
- Clock tree setup
- DDR memory controller
- MIO configuration

### 4.2 Procedures

| Procedure | Description |
|-----------|-------------|
| `ps7_pll_init_data_*` | PLL configuration |
| `ps7_clock_init_data_*` | Clock tree |
| `ps7_ddr_init_data_*` | DDR controller |
| `ps7_mio_init_data_*` | MIO pins |
| `ps7_init` | Main initialization |

### 4.3 LibreSDR PLL Configuration

**Main PLL (ARM PLL)**:
```
FCLK0 (CPU):  750 MHz (x30 of 25 MHz reference)
FCLK1 (DDR):  525 MHz (x21 of 25 MHz reference)
FCLK2:        200 MHz
FCLK3:        100 MHz
```

---

## 5. DDR Configuration

### 5.1 LibreSDR DDR

| Parameter | Value |
|-----------|-------|
| Type | DDR3 |
| Size | 1 GB |
| Width | 32-bit |
| Speed | 525 MHz (default) |

### 5.2 DDR Initialization

```tcl
# DDR registers configuration
mask_write 0xF8006000 0xFFFFFFFF 0x00000084  # Control
mask_write 0xF8006004 0xFFFFFFFF 0x00001082  # Status
# ... additional timing registers
```

---

## 6. JTAG Boot Process

### 6.1 Step-by-Step

```
1. Connect JTAG hardware
   |
   v
2. Load ps7_init.tcl
   |
   v
3. Initialize PS7 (clocks, DDR, MIO)
   |
   v
4. Download U-Boot.elf to RAM (0x04000000)
   |
   v
5. Execute U-Boot
   |
   v
6. Use U-Boot to program flash
   |
   v
7. Reboot from flash
```

### 6.2 U-Boot Load Address

| Address | Purpose |
|---------|---------|
| 0x04000000 | U-Boot load address |
| 0x2080000 | FIT image load |

---

## 7. Programming Flash via JTAG

### 7.1 U-Boot Commands

```bash
# Initialize QSPI
sf probe 0

# Erase boot partition
sf erase 0 0x100000

# Write boot image
sf write <addr> 0x0 <size>

# Erase kernel partition
sf erase 0x200000 0x1E00000

# Write FIT image
sf write <addr> 0x200000 <size>
```

### 7.2 DFU via JTAG

```bash
# Start DFU
run dfu_sf

# Then use dfu-util from host
dfu-util -D boot.frm -a boot
dfu-util -D pluto.frm -a rootfs
```

---

## 8. SD Card Recovery

### 8.1 SD Card Boot

LibreSDR can boot from SD card:
1. Prepare SD card with firmware
2. Insert into LibreSDR
3. Set boot mode to SD
4. Power on

### 8.2 Advantages

- No JTAG required
- Safe recovery method
- Can reflash QSPI when running

---

## 9. JTAG Debugging

### 9.1 Debug Connection

```bash
# Connect to ARM via XMD
connect arm hw
stop

# Read registers
reg pc
reg cpsr

# Read memory
mdw 0x04000000 10
```

### 9.2 Breakpoints

```bash
# Set breakpoint
bp 0x04010000

# Continue execution
con

# Remove breakpoint
rbp 0x04010000
```

---

## 10. Vivado Hardware Manager

### 10.1 Open Hardware Manager

```bash
vivado -mode batch -source tcl_script.tcl
```

### 10.2 Program FPGA

```tcl
# Open hardware manager
open_hw_manager

# Connect to hardware
connect_hw_server -url localhost:3121

# Refresh devices
refresh_hw_device [get_hw_devices xc7z020_1]

# Program FPGA
set_property PROGRAM.FILE {system_top.bit} [get_hw_devices xc7z020_1]
program_hw_devices [get_hw_devices xc7z020_1]
```

---

## 11. JTAG Cables

### 11.1 Supported Cables

| Cable | Connection | Speed |
|--------|-----------|-------|
| Digilent HS2 | USB | High |
| Xilinx Platform USB | USB | High |
| JTAG-HS3 | USB | High |
| DAPLink | Debug | Medium |

### 11.2 Connection

```
Host PC
    |
    +-- USB Cable -->
         |
         v
    JTAG Debugger
         |
         +-- JTAG Cable -->
              |
              v
         LibreSDR JTAG Header
```

---

## 12. Troubleshooting

### 12.1 Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| No JTAG connection | Cable not connected | Check cable |
| Device not found | Driver issue | Install drivers |
| DDR init fails | Memory issue | Check hardware |
| U-Boot hang | Bad firmware | Reflash |

### 12.2 Debug Steps

1. Verify JTAG cable connection
2. Check power to LibreSDR
3. Verify reference voltage (VTREF)
4. Try slower JTAG clock
5. Check cable integrity

---

## 13. Related Documents

- `LIBRESDR_BOOT_FLOW.md` - Boot sequence
- `LIBRESDR_UBOOT_BOOT_CUSTOMIZATION.md` - U-Boot configuration
- `LIBRESDR_FLASH_LAYOUT.md` - Flash layout
