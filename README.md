# LibreSDR-KB: LibreSDR/ZynqSDR Engineering Knowledge Base

Comprehensive engineering documentation and analysis for the LibreSDR (also known as ZynqSDR) SDR platform - a PlutoSDR-compatible board with enhanced specifications.

## Overview

This repository contains detailed technical documentation about the LibreSDR hardware, firmware, boot process, FPGA architecture, and software stack. LibreSDR is a third-party PlutoSDR clone that features a larger Zynq-7020 FPGA and 1GB DDR memory, with support for higher sample rates through overclocking.

## Repository Structure

```
libres-kb/
|-- README.md                    # This file
|-- .claude/
|   `-- CLAUDE.md                # Project orchestration instructions
|-- docs/                        # Engineering documentation (12 files)
|   |-- LIBRESDR_SYSTEM_ARCHITECTURE.md
|   |-- LIBRESDR_HARDWARE_SPECIFICATIONS.md
|   |-- LIBRESDR_BOOT_FLOW.md
|   |-- LIBRESDR_FIRMWARE_FORMATS.md
|   |-- LIBRESDR_FLASH_LAYOUT.md
|   |-- LIBRESDR_FPGA_ARCHITECTURE.md
|   |-- LIBRESDR_AD936X_CONFIG.md
|   |-- LIBRESDR_OVERCLOCKING.md
|   |-- LIBRESDR_PERFORMANCE.md
|   |-- LIBRESDR_BOARD_COMPARISON.md
|   |-- LIBRESDR_BUILD_SYSTEM.md
|   `-- LIBRESDR_KNOWLEDGE_BASE.md
|-- libresdr/                   # hz12opensource/libresdr (Patches & overclock)
|-- LibreSDR/                  # lucasmellon-ops/LibreSDR (Firmware collection)
|-- tezuka_fw/                 # F5OEO/tezuka_fw (Universal firmware builder)
`-- changes/                    # Change tracking
```

## Hardware Summary

| Parameter | LibreSDR | PlutoSDR |
|-----------|----------|----------|
| SoC | Xilinx Zynq-7020 (XC7Z020) | Zynq-7010 (XC7Z010) |
| FPGA Logic | 85K LUTs, 53K Slices | 28K LUTs, 17K Slices |
| CPU | ARM Cortex-A9 @ 750 MHz | ARM Cortex-A9 @ 666 MHz |
| Memory | 1 GB DDR3 | 512 MB DDR3L |
| Flash | QSPI NOR (similar) | QSPI NOR |
| Ethernet | Gigabit (GbE) | 100 Mbps |
| USB | USB 2.0 OTG | USB 2.0 OTG |
| Boot | SD Card + QSPI | QSPI + USB MSD |
| RF Transceiver | AD9363 (same as PlutoSDR) | AD9363 |

## Performance

| Configuration | CPU Clock | DDR Clock | Continuous Sample Rate |
|---------------|----------|----------|----------------------|
| Base (default) | 750 MHz | 525 MHz | 20 MSPS |
| Overclocked | 1100 MHz | 750 MHz | 27.5 MSPS |
| Stock PlutoSDR | 666 MHz | 525 MHz | ~10-12 MSPS |

> Note: Sample rate limitations are often due to host computer Ethernet/USB bandwidth, not the device itself.

## Key Features

- **SD Card Boot**: Primary boot mode, zero-risk firmware updates
- **Overclocking**: Up to 1100 MHz CPU / 750 MHz DDR supported
- **Gigabit Ethernet**: Enables high sample rate streaming over network
- **LVDS Mode**: AD9361 LVDS mode for max sampling rates in 2T2R mode
- **DVB Support**: DATV/DVB-S2 TX capability (via tezuka_fw)
- **Maia-SDR**: Web-based spectrum analyzer (via tezuka_fw)

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

## Firmware Sources

| Source | Description |
|--------|-------------|
| [hz12opensource/libresdr](https://github.com/hz12opensource/libresdr) | Overclock patches and prebuilt firmware |
| [lucasmellon-ops/LibreSDR](https://github.com/lucasmellon-ops/LibreSDR) | Firmware collection and documentation |
| [F5OEO/tezuka_fw](https://github.com/F5OEO/tezuka_fw) | Universal firmware builder |

## Documentation

| Document | Description |
|----------|-------------|
| `LIBRESDR_SYSTEM_ARCHITECTURE.md` | Complete system overview |
| `LIBRESDR_HARDWARE_SPECIFICATIONS.md` | Detailed hardware specs |
| `LIBRESDR_BOOT_FLOW.md` | Boot sequence |
| `LIBRESDR_FIRMWARE_FORMATS.md` | Firmware artifact analysis |
| `LIBRESDR_OVERCLOCKING.md` | Overclocking guide |
| `LIBRESDR_PERFORMANCE.md` | Performance benchmarks |
| `LIBRESDR_BOARD_COMPARISON.md` | Comparison with PlutoSDR |

## References

- [LibreSDR GitHub](https://github.com/hz12opensource/libresdr)
- [LibreSDR Documentation](https://github.com/lucasmellon-ops/LibreSDR)
- [Tezuka Firmware](https://github.com/F5OEO/tezuka_fw)
- [Analog Devices Wiki - PlutoSDR](https://wiki.analog.com/university/tools/pluto)
- [SDR++](https://github.com/AlexandreRouma/SDRPlusPlus)

## Contributing

Found an error or want to add analysis? Open an issue or submit a pull request.

## License

This documentation is provided for educational and engineering reference purposes. Hardware designs and firmware are property of their respective owners.
