# libres-kb Project Index

## Project Overview

- **Name**: libres-kb
- **Status**: Active
- **Type**: LibreSDR/ZynqSDR engineering knowledge base
- **Last Updated**: 2026-09-09
- **Primary Focus**: Comprehensive LibreSDR/ZynqSDR SDR platform engineering reference

## Project Structure

```
libres-kb/
├── README.md                    # Project overview (121 lines)
├── .claude/
│   ├── CLAUDE.md                # Project orchestration instructions
│   └── MEMORY.md                # Project memory and knowledge tracking
├── docs/                        # Engineering documentation (22 files)
│   ├── LIBRESDR_SYSTEM_ARCHITECTURE.md  # System overview
│   ├── LIBRESDR_hARDWARE_SPECIFICATIONS.md # Detailed hardware specs
│   ├── LIBRESDR_BOOT_FLOW.md              # Boot sequence
│   ├── LIBRESDR_FIRMWARE_FORMATS.md      # Firmware artifact analysis
│   ├── LIBRESDR_FLASH_LAYOUT.md          # QSPI flash memory map
│   ├── LIBRESDR_FPGA_ARCHITECTURE.md     # FPGA design and register map
│   ├── LIBRESDR_AD936X_CONFIG.md         # AD9363 configuration
│   ├── LIBRESDR_OVERCLOCKING.md          # Overclocking guide
│   ├── LIBRESDR_PERFORMANCE.md           # Performance benchmarks
│   ├── LIBRESDR_BOARD_COMPARISON.md      # PlutoSDR vs LibreSDR
│   ├── LIBRESDR_BUILD_SYSTEM.md          # Build system analysis
│   ├── LIBRESDR_KNOWLEDGE_BASE.md        # Master index
│   ├── LIBRESDR_DEVICE_TREE.md           # Device tree mapping
│   ├── LIBRESDR_FIRMWARE_ANALYSIS.md     # Firmware binary analysis
│   ├── LIBRESDR_FPGA_BITSTREAM.md        # FPGA bitstream analysis
│   ├── LIBRESDR_IIO_ARCHITECTURE.md      # IIO subsystem
│   ├── LIBRESDR_JTAG_BOOTSTRAP.md        # JTAG bootstrap mechanism
│   ├── LIBRESDR_ROOTFS_ANALYSIS.md       # Root filesystem analysis
│   ├── LIBRESDR_UBOOT_BOOT_CUSTOMIZATION.md # U-Boot customization
│   ├── LIBRESDR_WEB_ARCHITECTURE.md      # Web interface architecture
│   ├── LIBRESDR_REVISION_DIFFERENCES.md   # Hardware variants
│   └── SOURCES.md                        # Source repository index
├── libresdr/                   # hz12opensource/libresdr (overclock patches)
├── LibreSDR/                  # lucasmellon-ops/LibreSDR (firmware collection)
├── tezuka_fw/                 # F5OEO/tezuka_fw (universal firmware builder)
└── changes/                    # Change tracking
```

## Documentation Completeness

All 22 documentation files are present:

| Document | Status | Description |
|----------|--------|-------------|
| `LIBRESDR_SYSTEM_ARCHITECTURE.md` | [X] Complete | System overview and block diagrams |
| `LIBRESDR_hARDWARE_SPECIFICATIONS.md` | [X] Complete | Detailed hardware specifications |
| `LIBRESDR_BOOT_FLOW.md` | [X] Complete | Boot sequence and initialization |
| `LIBRESDR_FIRMWARE_FORMATS.md` | [X] Complete | Firmware file structure analysis |
| `LIBRESDR_FLASH_LAYOUT.md` | [X] Complete | QSPI flash memory map |
| `LIBRESDR_FPGA_ARCHITECTURE.md` | [X] Complete | FPGA design and register map |
| `LIBRESDR_AD936X_CONFIG.md` | [X] Complete | AD9363 configuration |
| `LIBRESDR_OVERCLOCKING.md` | [X] Complete | Overclocking guide |
| `LIBRESDR_PERFORMANCE.md` | [X] Complete | Performance benchmarks |
| `LIBRESDR_BOARD_COMPARISON.md` | [X] Complete | PlutoSDR vs LibreSDR comparison |
| `LIBRESDR_BUILD_SYSTEM.md` | [X] Complete | Build system analysis |
| `LIBRESDR_KNOWLEDGE_BASE.md` | [X] Complete | Master index and cross-reference |
| `LIBRESDR_DEVICE_TREE.md` | [X] Complete | Device tree mapping |
| `LIBRESDR_FIRMWARE_ANALYSIS.md` | [X] Complete | Firmware binary analysis |
| `LIBRESDR_FPGA_BITSTREAM.md` | [X] Complete | FPGA bitstream analysis |
| `LIBRESDR_IIO_ARCHITECTURE.md` | [X] Complete | IIO subsystem architecture |
| `LIBRESDR_JTAG_BOOTSTRAP.md` | [X] Complete | JTAG bootstrap mechanism |
| `LIBRESDR_ROOTFS_ANALYSIS.md` | [X] Complete | Root filesystem analysis |
| `LIBRESDR_UBOOT_BOOT_CUSTOMIZATION.md` | [X] Complete | U-Boot boot customization |
| `LIBRESDR_WEB_ARCHITECTURE.md` | [X] Complete | Web interface architecture |
| `LIBRESDR_REVISION_DIFFERENCES.md` | [X] Complete | Hardware variants and differences |
| `SOURCES.md` | [X] Complete | Source repository index |

## Firmware Sources

| Source | Description |
|--------|-------------|
| [hz12opensource/libresdr](https://github.com/hz12opensource/libresdr) | Overclock patches and prebuilt firmware |
| [lucasmellon-ops/LibreSDR](https://github.com/lucasmellon-ops/LibreSDR) | Firmware collection and documentation |
| [F5OEO/tezuka_fw](https://github.com/F5OEO/tezuka_fw) | Universal firmware builder |

## Hardware Comparison

| Parameter | LibreSDR | PlutoSDR |
|-----------|----------|----------|
| SoC | Xilinx Zynq-7020 (XC7Z020, 85K LUTs) | Zynq-7010 (XC7Z010, 28K LUTs) |
| CPU Clock | 750 MHz (default), 1100 MHz (OC) | 666 MHz |
| DDR | 1 GB @ 525-750 MHz | 512 MB @ 525 MHz |
| Ethernet | Gigabit | 100 Mbps |
| FPGA | Larger (85K LUTs) | Smaller (28K LUTs) |
| Boot Mode | SD Card primary | QSPI primary |
| RF Transceiver | AD9363 (1Rx-1Tx or 2Rx-2Tx) | AD9363 (1Rx-1Tx) |
| Frequency Range | 325 MHz - 3800 MHz | 325 MHz - 3800 MHz |

## Performance Comparison

| Configuration | CPU Clock | DDR Clock | Continuous Sample Rate |
|---------------|----------|----------|----------------------|
| LibreSDR Base | 750 MHz | 525 MHz | 20 MSPS |
| LibreSDR OC | 1100 MHz | 750 MHz | 27.5 MSPS |
| PlutoSDR Stock | 666 MHz | 525 MHz | ~10-12 MSPS |

## Key Features

- **SD Card Boot**: Primary boot mode, zero-risk firmware updates
- **Overclocking**: Up to 1100 MHz CPU / 750 MHz DDR (27.5 MSPS)
- **Gigabit Ethernet**: Enables high sample rate streaming
- **LVDS Mode**: AD9361 LVDS mode for max sampling rates in 2T2R mode
- **DVB Support**: DATV/DVB-S2 TX capability (via tezuka_fw)
- **Maia-SDR**: Web-based spectrum analyzer (via tezuka_fw)
- **Platform Detection**: Device-tree model string (not GPIO probing)

## Quick Start

### Prebuilt Firmware

1. Download from [tezuka_fw releases](https://github.com/F5OEO/tezuka_fw/releases)
2. Extract the `libre` package
3. Copy contents of `sdimg/` to FAT32-formatted SD card
4. Insert SD card and power on

### Build from Source

```bash
git clone https://github.com/F5OEO/tezuka_fw.git
cd tezuka_fw
./getbuildroot.sh
source sourceme.first
./build.sh libre
```

## Research Status

All documentation is marked as PLANNED in the internal status tables. The research has been conducted and documented in the engineering reference files. See individual document status tables for verification levels (VERIFIED, STRONGLY INFERRED, INFERRED, UNKNOWN).

## Quick Navigation

### Core Documentation
- `README.md` - Project overview and hardware comparison
- `docs/LIBRESDR_SYSTEM_ARCHITECTURE.md` - Complete system architecture
- `docs/LIBRESDR_KNOWLEDGE_BASE.md` - Master index with cross-references
- `docs/SOURCES.md` - Source repository index and cloning instructions

### Hardware Reference
- `docs/LIBRESDR_HARDWARE_SPECIFICATIONS.md` - Detailed hardware specs
- `docs/LIBRESDR_BOARD_COMPARISON.md` - PlutoSDR vs LibreSDR vs other platforms
- `docs/LIBRESDR_AD936X_CONFIG.md` - AD9363 configuration
- `docs/LIBRESDR_OVERCLOCKING.md` - Overclocking guide

### Boot & Firmware
- `docs/LIBRESDR_BOOT_FLOW.md` - Boot sequence and initialization
- `docs/LIBRESDR_FIRMWARE_FORMATS.md` - Firmware file structures
- `docs/LIBRESDR_FLASH_LAYOUT.md` - QSPI flash memory map
- `docs/LIBRESDR_UBOOT_BOOT_CUSTOMIZATION.md` - U-Boot customization

### Development Reference
- `docs/LIBRESDR_BUILD_SYSTEM.md` - Build system and buildroot
- `docs/LIBRESDR_FPGA_ARCHITECTURE.md` - FPGA design analysis
- `docs/LIBRESDR_PERFORMANCE.md` - Performance benchmarks
- `docs/LIBRESDR_JTAG_BOOTSTRAP.md` - JTAG recovery procedures

### Reference Materials
- See `.claude/README.md` for global skills and agents
- See `USER.md` for user expertise and preferences
- See master index at `/root/claude-code/INDEX.md` for workspace overview
- See `pluto-kb/INDEX.md` for ADALM-PLUTO engineering knowledge base

## Related Projects

- **pluto-kb**: ADALM-PLUTO engineering reference (PlutoSDR v0.40, Zynq-7010)
- **plutodvb-main**: DATV transmission over QO-100 (uses both PlutoSDR and LibreSDR)

## Operating Principles

### For This Project
1. Evidence-based documentation (prefer source code over documentation)
2. Label inferences explicitly (VERIFIED/INFERRED/UNKNOWN)
3. Use Turkish for user-facing communication
4. English for technical documentation
5. ASCII-only for generated files

### For LibreSDR Development
1. Use `double` for frequency, phase, timing, and RF calculations
2. Units explicitly stated (Hz, kHz, MHz, GHz)
3. No silent assumptions about hardware parameters
4. Compare with PlutoSDR where applicable
5. Document overclocking procedures carefully