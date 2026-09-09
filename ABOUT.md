# LibreSDR-KB: LibreSDR/ZynqSDR Engineering Knowledge Base

## About

LibreSDR-KB is a comprehensive engineering knowledge base for the LibreSDR (also known as ZynqSDR) SDR platform, a PlutoSDR-compatible board with enhanced specifications. It documents the LibreSDR hardware architecture, firmware, boot process, FPGA design, and software stack, with comparisons to PlutoSDR where applicable.

### What Is LibreSDR?

The LibreSDR is a third-party PlutoSDR clone built around the Xilinx Zynq-7020 SoC (XC7Z020, larger than PlutoSDR's Z-7010) and the Analog Devices AD9363 RF transceiver. It features 1 GB DDR3 memory, Gigabit Ethernet, SD card boot, and support for overclocking up to 1100 MHz CPU / 750 MHz DDR, enabling continuous sample rates up to 27.5 MSPS.

### What This Project Contains

- **Source repositories**: Three upstream projects are cloned into the project root:
  - `libresdr/` (hz12opensource/libresdr) - Original LibreSDR firmware with overclock patches
  - `LibreSDR/` (lucasmellon-ops/LibreSDR) - Firmware collection and documentation
  - `tezuka_fw/` (F5OEO/tezuka_fw) - Universal firmware builder with LibreSDR support
- **Documentation**: 20 engineering documents covering system architecture, hardware specifications, boot flow, FPGA, AD936x configuration, overclocking, performance, board comparison, flash layout, device tree, and more.
- **Firmware analysis**: Extracted firmware components and artifact analysis.

### Key Hardware Parameters

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

### Performance

| Configuration | CPU Clock | DDR Clock | Continuous Sample Rate |
|---------------|----------|----------|----------------------|
| Base (default) | 750 MHz | 525 MHz | 20 MSPS |
| Overclocked | 1100 MHz | 750 MHz | 27.5 MSPS |
| Stock PlutoSDR | 666 MHz | 525 MHz | ~10-12 MSPS |

> Note: Sample rate limitations are often due to host computer Ethernet/USB bandwidth, not the device itself.

### Documentation

See `docs/` for the full set of engineering documents. The master index is `docs/LIBRESDR_KNOWLEDGE_BASE.md`.

### Sources

- [LibreSDR GitHub](https://github.com/hz12opensource/libresdr)
- [LibreSDR Documentation](https://github.com/lucasmellon-ops/LibreSDR)
- [Tezuka Firmware](https://github.com/F5OEO/tezuka_fw)
- [Analog Devices Wiki - PlutoSDR](https://wiki.analog.com/university/tools/pluto)
- [SDR++](https://github.com/AlexandreRouma/SDRPlusPlus)

### License

This documentation is provided for educational and engineering reference purposes. Hardware designs and firmware are property of their respective owners.