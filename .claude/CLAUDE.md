# LIBRES-KB Project - Claude Code Orchestration

## Project Overview

This is an engineering knowledge base project for the LibreSDR (ZynqSDR) SDR platform, a PlutoSDR-compatible board based on Xilinx Zynq-7020 with enhanced specifications.

### Project Goals

1. Build a comprehensive engineering model of the LibreSDR platform
2. Document hardware architecture, FPGA, Linux, boot chain, and software stack
3. Analyze firmware formats, update mechanisms, and flash layout
4. Create a persistent reference for future LibreSDR development
5. Compare with PlutoSDR and document differences

### Source Repositories (cloned in project root)

| Repository | Local Path | Purpose |
|------------|------------|---------|
| hz12opensource/libresdr | `/libresdr/` | Original LibreSDR firmware with overclock |
| lucasmellon-ops/LibreSDR | `/LibreSDR/` | Firmware collection and documentation |
| F5OEO/tezuka_fw | `/tezuka_fw/` | Universal firmware builder with LibreSDR support |

### LibreSDR Board Specifics

- **SoC**: Xilinx Zynq-7020 (XC7Z020) - larger than PlutoSDR's Z-7010
- **Memory**: 1 GB DDR3 (vs PlutoSDR's 512 MB)
- **Flash**: QSPI NOR (similar to PlutoSDR)
- **Boot**: SD card primary, USB MSD fallback
- **Network**: Gigabit Ethernet (vs PlutoSDR's 100M)
- **Clock**: Default 750 MHz CPU / 525 MHz DDR, supports overclock to 1100 MHz / 750 MHz

---

## Project Structure

```
libres-kb/
|-- README.md                    # Project overview
|-- .claude/
|   `-- CLAUDE.md                # This file
|-- docs/                        # Engineering documentation
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
|-- libresdr/                   # hz12opensource/libresdr
|-- LibreSDR/                  # lucasmellon-ops/LibreSDR
|-- tezuka_fw/                 # F5OEO/tezuka_fw
`-- changes/                    # Change tracking
```

---

## Key Findings Summary

### Hardware Architecture (vs PlutoSDR)

| Parameter | LibreSDR | PlutoSDR |
|-----------|----------|----------|
| SoC | Zynq-7020 (85K LUTs) | Zynq-7010 (28K LUTs) |
| CPU Clock | 750 MHz (default) | 666 MHz |
| DDR | 1 GB @ 525 MHz | 512 MB @ 525 MHz |
| Ethernet | Gigabit | 100 Mbps |
| FPGA | Larger (more resources) | Smaller |
| Boot Mode | SD Card primary | QSPI primary |

### Overclocking Achievements

| Configuration | CPU | DDR | Max Sample Rate |
|---------------|-----|-----|-----------------|
| Base (default) | 750 MHz | 525 MHz | 20 MSPS |
| Overclocked | 1100 MHz | 750 MHz | 27.5 MSPS |

### Supported Firmware Builders

1. **hz12opensource/libresdr**: Based on PlutoSDR v0.38, with overclock patches
2. **F5OEO/tezuka_fw**: Universal builder, supports LibreSDR (`libre` board)
3. **Manual Build**: Vivado 2022.2 + custom patches

---

## Documentation Status

| Document | Status | Quality |
|----------|--------|---------|
| System Architecture | PLANNED | - |
| Hardware Specifications | PLANNED | - |
| Boot Flow | PLANNED | - |
| Firmware Formats | PLANNED | - |
| Flash Layout | PLANNED | - |
| FPGA Architecture | PLANNED | - |
| AD936x Configuration | PLANNED | - |
| Overclocking Guide | PLANNED | - |
| Performance Analysis | PLANNED | - |
| Board Comparison | PLANNED | - |
| Build System | PLANNED | - |
| Knowledge Base | PLANNED | - |

---

## Missing Information

### Critical
1. **FPGA HDL Source**: Full HDL project not yet analyzed
2. **Runtime Observations**: Actual hardware testing data
3. **Detailed Schematics**: Only partial schematic available (zynqsdr_rev5.pdf)

### Nice to Have
1. Complete flash dump analysis
2. Performance benchmarking data
3. Comparison with other PlutoSDR clones

---

## Research Conducted

### Sources Analyzed
- hz12opensource/libresdr/README.md (confirmed)
- hz12opensource/libresdr/patches/*.diff (confirmed)
- lucasmellon-ops/LibreSDR/README.md (confirmed)
- F5OEO/tezuka_fw/boards.json (confirmed)
- F5OEO/tezuka_fw/board/tezuka/libre/ structure (confirmed)
- LibreSDR firmware files in LibreSDR/Firmware&deviceFiles/Working-fw/ (confirmed)

### Evidence Levels
- **CONFIRMED**: Directly verified from source
- **STRONGLY INFERRED**: High confidence from indirect evidence
- **INFERRED**: Based on typical patterns
- **UNKNOWN**: Not yet verified

---

## Operating Principles

### For This Project
1. Evidence-based documentation (prefer source code over documentation)
2. Label inferences explicitly (CONFIRMED/INFERRED/UNKNOWN)
3. Use Turkish for user-visible communication
4. English for technical documentation
5. ASCII-only for generated files

### For LibreSDR Development
1. RF calculations use double precision
2. Units explicitly stated (Hz, kHz, MHz, GHz)
3. No silent assumptions about hardware parameters
4. Compare with PlutoSDR where applicable
5. Document overclocking procedures carefully

---

## Future Work

1. Analyze FPGA HDL source code
2. Document complete boot process
3. Create performance benchmarks
4. Document firmware update procedures
5. Compare with tezuka_fw implementation

## GitHub Commit Automation

### BEFORE any commit to GitHub:
1. Run documentation-sync skill to verify README.md reflects actual directory structure.
2. Verify no new files are missing from README.md repository structure section.
3. Verify all docs/ files mentioned in README.md actually exist.

### Author rules for GitHub commits:
- **Always** use author: `ta1rjv <amateurta1rjv@gmail.com>` (user) or `claude <claude@anthropic.com>` (Claude code)
- Never commit as root or any other author.
- For sub-agent commits, use author: `claude <claude@anthropic.com>` only when explicitly doing Claude-code-only work.
- Before committing, verify the author name matches one of the two allowed values above.

### Commit message rules:
- Commit messages must be descriptive: `type(scope): description`
- Never push empty, "wip", or single-word commits.
- Always push to `main` branch only after README verification.
