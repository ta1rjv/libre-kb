# LIBRESDR FPGA BITSTREAM ANALYSIS

## Engineering Reference - FPGA Bitstream Deep Analysis

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | FPGA_BITSTREAM_ANALYSIS |
| **Last Updated** | 2026-09-07 |
| **Source** | tezuka_fw board/tezuka/libre/bitstream/ |

---

## 1. Bitstream Overview

### 1.1 File Details

| Property | Value |
|----------|-------|
| File | system_top.bit / tezuka.xsa |
| Target Device | 7z020clg400 (Zynq-7020) |
| Package | clg400 (BGA, 400-pin) |
| Speed Grade | -1 (commercial) |
| Compression | TRUE |

**Note**: LibreSDR uses Zynq-7020 vs PlutoSDR's Zynq-7010, resulting in a larger bitstream.

---

## 2. Zynq-7020 Architecture

### 2.1 Device Overview

| Feature | Zynq-7020 | Zynq-7010 | Difference |
|---------|------------|------------|------------|
| Logic Cells | 85K | 28K | 3x more |
| LUTs | 53,200 | 17,600 | 3x more |
| Flip-Flops | 106,400 | 35,200 | 3x more |
| Block RAM | 4.9 Mb | 2.1 Mb | 2.3x more |
| DSP Slices | 220 | 80 | 2.75x more |

### 2.2 Configuration Frames

The 7z020 has more configuration frames:

| Frame Type | Zynq-7020 | Zynq-7010 |
|------------|------------|------------|
| CLB frames | ~2400 | ~801 |
| BRAM frames | ~60 | ~30 |
| Total | ~2500 | ~831 |

---

## 3. Bitstream Header Format

### 3.1 Header Structure

```
+------------------+
| Text Header     |  Variable length
| (Key-value)     |
+------------------+
| NULL delimiter   |  1 byte
+------------------+
| Sync Word        |  0xAA995566 (4 bytes)
+------------------+
| Binary Data     |  Configuration data
+------------------+
```

### 3.2 Header Fields

| Field | Value | Description |
|-------|-------|-------------|
| Design | system_top | Top-level design name |
| Part | 7z020clg400 | Target FPGA part |
| Compression | TRUE | Bitstream compression enabled |
| UserID | 0xFFFFFFFF | User ID field |

---

## 4. Binary Configuration Data

### 4.1 Sync Word

The **sync word (0xAA995566)** marks the start of configuration data.

### 4.2 Configuration Packets

Type 2 Configuration Data Packets are used for:
- Configuration frame data
- Implicit address increment

### 4.3 Frame Structure

Each configuration frame contains:
- Frame address word
- Configuration data
- Padding words

---

## 5. LibreSDR FPGA Design

### 5.1 IP Cores

The FPGA design includes:

```
system_top
|-- ps7 (Processing System)
|   |-- FCLK_CLK0 -> sys_cpu_clk
|   `-- M_AXI_GP0 -> processing_system7_0_M_AXI_GP0
|-- axi_ad9361
|   |-- base_rst (reset)
|   |-- sys_cpu_clk (clock)
|   `-- axi (AXI4-Lite interface)
|-- axi_ad9361_adc_dma
|   `-- m_dest_axi -> ps7.S_AXI_HP1 (DMA to DDR)
|-- axi_ad9361_dac_dma
|   `-- m_dest_axi -> ps7.S_AXI_HP2 (DMA from DDR)
|-- axi_iic_main
|   `-- iic_main (external I2C)
|-- (Optional) axi_tdd_0
|   `-- tdd (TDD controller)
```

### 5.2 Address Map

| Peripheral | Address | Size |
|------------|---------|------|
| axi_ad9361 | 0x79020000 | 24 KB |
| axi_ad9361_adc_dma | 0x7C400000 | 4 KB |
| axi_ad9361_dac_dma | 0x7C420000 | 4 KB |
| axi_iic_main | 0x41600000 | 64 KB |

---

## 6. LVDS Interface

### 6.1 LVDS Mode Benefits

LibreSDR supports LVDS mode for maximum sample rates:
- 12-bit I data + 12-bit Q data
- Differential signaling
- Up to 61.44 MSPS

### 6.2 Signal Mapping

| Signal | Description |
|--------|-------------|
| DATA[5:0]P/N | I data (LVDS pairs) |
| DATA[11:6]P/N | Q data (LVDS pairs) |
| FB_CLK | Feedback clock |
| RX_FRAME | Frame sync |

---

## 7. Bitstream Compression

### 7.1 Compression Status

**COMPRESS=TRUE** - The bitstream is compressed using BITSWAP.

### 7.2 Compression Benefits

| Metric | Uncompressed | Compressed | Savings |
|--------|--------------|-------------|---------|
| Size | ~2.5 MB | ~1.2 MB | ~52% |
| Config Time | ~250 ms | ~150 ms | ~40% |

---

## 8. FPGA Configuration

### 8.1 Configuration Mode

LibreSDR uses **QSPI flash** for FPGA configuration:
- Boot mode: QSPI (M[2:0] = 001)
- CCLK: 50-104 MHz

### 8.2 Configuration Flow

```
BootROM
    |
    v
Load FSBL from QSPI
    |
    v
FSBL initializes PS
    |
    v
FSBL programs FPGA
    |
    v
FPGA starts up
    |
    v
Load U-Boot
```

---

## 9. Bitstream Security

### 9.1 Bitstream Authentication

LibreSDR does **NOT** use bitstream encryption:
- No eFUSE encryption key
- Bitstream stored in plaintext
- Can be read/modified

### 9.2 Readback Protection

Configuration registers can be locked:
- BITSTREAM.READBACK = FALSE
- BITSTREAM.USR_ACCESS = locked

---

## 10. Custom Bitstream Development

### 10.1 Development Flow

```bash
# 1. Clone maia-sdr HDL
git clone https://github.com/F5OEO/maia-sdr.git
cd maia-sdr

# 2. Set up Vivado
source /opt/Xilinx/Vivado/2023.1/settings64.sh

# 3. Open project
cd maia-hdl/projects/tezuka
make PROJECT_NAME=libre

# 4. Generate bitstream
make system_top.bit

# 5. Export
make system_top.xsa
```

### 10.2 Customization Points

1. **Clock multipliers** in system_bd.tcl
2. **DDR timing** in ps7.tcl
3. **GPIO assignments** in ports.tcl
4. **FPGA resources** - add custom IP

---

## 11. Tools for Bitstream Analysis

### 11.1 Vivado

```bash
# Open bitstream
vivado -bin system_top.bit

# Read configuration
report_config_checksum

# Export to raw bits
write_bitsfile -force system_top.bin
```

### 11.2 Custom Python Script

```python
def parse_bitstream(filename):
    with open(filename, 'rb') as f:
        data = f.read()
    
    # Find sync word
    sync_pos = data.find(b'\xaa\x99\x55\x66')
    
    # Header is before sync word
    header = data[:sync_pos]
    
    # Binary starts after sync word
    binary = data[sync_pos + 4:]
    
    return header, binary
```

---

## 12. Comparison with PlutoSDR

| Property | LibreSDR | PlutoSDR |
|----------|-----------|----------|
| FPGA | Zynq-7020 | Zynq-7010 |
| LUTs | 53,200 | 17,600 |
| BRAM | 4.9 Mb | 2.1 Mb |
| Bitstream size | ~1.2 MB | ~943 KB |
| Frames | ~2500 | ~801 |

---

## 13. Related Documents

- `LIBRESDR_FPGA_ARCHITECTURE.md` - HDL source analysis
- `LIBRESDR_DEVICE_TREE.md` - Device tree details
- `LIBRESDR_SYSTEM_ARCHITECTURE.md` - System overview
