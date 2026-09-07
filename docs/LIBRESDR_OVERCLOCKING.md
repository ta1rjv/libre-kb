# LIBRESDR OVERCLOCKING GUIDE

## Engineering Reference - Overclocking LibreSDR

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | OVERCLOCKING GUIDE |
| **Last Updated** | 2026-09-07 |
| **Source** | hz12opensource/libresdr |

---

## 1. Overview

LibreSDR supports overclocking of both the CPU and DDR memory, enabling higher sample rates. The Zynq-7020 chip is designed for up to 766 MHz but can be safely pushed to 1100 MHz with proper cooling.

### 1.1 Clock Architecture

All clocks are derived from a 25 MHz reference crystal:
- CPU clock = 25 MHz * CPU_MULTIPLIER
- DDR clock = 25 MHz * DDR_MULTIPLIER

---

## 2. Overclock Configurations

### 2.1 Available Profiles

| Profile | CPU Multiplier | DDR Multiplier | CPU Speed | DDR Speed | Status |
|---------|---------------|----------------|-----------|-----------|--------|
| Base | 30 | 21 | 750 MHz | 525 MHz | Default |
| Medium | 38 | 24 | 950 MHz | 600 MHz | Tested |
| High | 44 | 30 | 1100 MHz | 750 MHz | Maximum |

### 2.2 Prebuilt FSBL Binaries

The `board/tezuka/libre/bitstream/overclock/` directory contains prebuilt FSBL binaries:

```
overclock/
|-- fsbl_CPU750_DDR525.elf   # Default (base)
|-- fsbl_CPU950_DDR600.elf   # Medium overclock
|-- fsbl_CPU1100_DDR750.elf  # Maximum overclock
```

---

## 3. Clock Multipliers

### 3.1 CPU Clock Calculation

```
CPU Clock = 25 MHz * CPU_MULTIPLIER

Examples:
- 750 MHz: 25 * 30 = 750 MHz
- 950 MHz: 25 * 38 = 950 MHz
- 1100 MHz: 25 * 44 = 1100 MHz
```

### 3.2 DDR Clock Calculation

```
DDR Clock = 25 MHz * DDR_MULTIPLIER

Examples:
- 525 MHz: 25 * 21 = 525 MHz
- 600 MHz: 25 * 24 = 600 MHz
- 750 MHz: 25 * 30 = 750 MHz
```

---

## 4. Building Overclocked Firmware

### 4.1 Using tezuka_fw

```bash
# Set environment
source sourceme.first

# Build with default settings
./build.sh libre

# For overclocking, modify the build:
OVERCLOCK_CPU_MULT=44 OVERCLOCK_DDR_MULT=30 ./build.sh libre
```

### 4.2 Using hz12opensource/libresdr

```bash
# Clone original firmware
git clone --branch v0.38 --recursive https://github.com/analogdevicesinc/plutosdr-fw.git

# Apply patches
./apply.sh

# Set Vivado path
export VIVADO_SETTINGS=/opt/Xilinx/Vivado/2022.2/settings64.sh
export TARGET=libre

# Build with overclock
OVERCLOCK_CPU_MULT=44 OVERCLOCK_DDR_MULT=30 make overclock
make sdimg
```

---

## 5. DDR Timing Parameters

### 5.1 Location

DDR timing parameters are defined in the HDL design:

```
hdl/projects/libre/system_bd.tcl
```

### 5.2 Key Parameters

| Parameter | Description | Base Value | OC Value |
|-----------|-------------|------------|----------|
| PCW_UIPARAM_DDR_CL | CAS Latency | 7 | 7 |
| PCW_UIPARAM_DDR_tRCD | RAS to CAS delay | - | - |
| PCW_UIPARAM_DDR_tRP | Precharge time | - | - |
| PCW_UIPARAM_DDR_tRFC | Refresh cycle | - | - |

### 5.3 Example Timing Configuration

For 750 MHz DDR operation with 9-7-9-9 timing:

```tcl
# CAS Latency 7
# tRCD 9
# tRP 9
# tRFC (depends on density)
```

---

## 6. Performance Impact

### 6.1 Sample Rate vs Configuration

| Configuration | CPU | DDR | Sample Rate |
|---------------|-----|-----|-------------|
| Stock PlutoSDR | 666 MHz | 525 MHz | ~10-12 MSPS |
| LibreSDR Base | 750 MHz | 525 MHz | 20 MSPS |
| LibreSDR OC | 950 MHz | 600 MHz | ~25 MSPS |
| LibreSDR Max OC | 1100 MHz | 750 MHz | 27.5 MSPS |

### 6.2 Host Computer Limitations

The limiting factor for high sample rates is often the host computer:
- Gigabit Ethernet: ~900 Mbps maximum
- USB 2.0: ~40 MB/s maximum
- CPU processing power for demodulation

---

## 7. Stability Considerations

### 7.1 Factors Affecting Stability

1. **CPU Voltage**: Higher clocks require stable voltage
2. **Thermal Management**: Heat dissipation increases with clock speed
3. **DDR Signal Integrity**: Higher speeds require tighter timing
4. **Power Supply**: Quality of onboard power regulators

### 7.2 Testing Recommendations

1. Start with base configuration
2. Test each overclock level incrementally
3. Monitor device temperature during operation
4. Run extended streaming tests
5. Verify data integrity at target sample rate

### 7.3 Failure Symptoms

- Intermittent data errors
- Dropped samples
- System hangs
- Kernel panics

---

## 8. Software Configuration

### 8.1 IIO Buffer Size

For high sample rates, increase the IIO buffer size in your SDR application:

```cpp
// SDR++ example - increase from default
// In source_modules/plutosdr_source/src/main.cpp
const int BUFFER_SIZE = 1000000;  // Increased from default
```

### 8.2 Sample Rate Limits

SDR++ has a sample rate limit for PlutoSDR:

```cpp
// In SDR++ source
// Change limit from 20 MSPS to higher value
const double MAX_SAMPLE_RATE = 30000000;
```

---

## 9. Known Limitations

1. **Cooling**: May require additional heat sinking for maximum overclock
2. **Variance**: Not all boards can achieve maximum overclock
3. **Power**: Higher clocks increase power consumption
4. **Warranty**: Overclocking may void warranty

---

## 10. Reference Commands

### 10.1 Checking Current Clocks

```bash
# Check CPU info
cat /proc/cpuinfo | grep "model name"

# Check available frequencies
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies

# Monitor temperature
cat /sys/class/thermal/thermal_zone0/temp
```

### 10.2 Network Performance Test

```bash
# iperf3 test
iperf3 -c 192.168.1.10 -p 5201 -t 60 -R
```

---

## 11. Future Enhancements

1. **Automatic Overclock Profiles**: Implement user-selectable OC levels
2. **Temperature Monitoring**: Add thermal throttling
3. **Voltage Control**: Implement dynamic voltage scaling
4. **DDR Training**: Improved signal integrity at high speeds
