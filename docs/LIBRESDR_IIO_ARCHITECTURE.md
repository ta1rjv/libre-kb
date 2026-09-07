# LIBRESDR IIO ARCHITECTURE

## Engineering Reference - Industrial I/O Subsystem

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | IIO SUBSYSTEM |
| **Last Updated** | 2026-09-07 |
| **Source** | linux/drivers/iio/, tezuka_fw |

---

## 1. IIO Overview

### 1.1 Industrial I/O Subsystem

The Linux IIO subsystem provides:
- Hardware sensor/attribute access
- Buffered data transfer
- Kernel-userspace interface
- Device attribute management

### 1.2 LibreSDR IIO Devices

| Device | Driver | IIO Device Name |
|--------|--------|-----------------|
| AD9363 | ad9361 | ad9361-phy |
| ADC Interface | axi_ad9361 | cf-ad9361-lpc |
| DAC/DDS Interface | axi_ad9361 | cf-ad9361-dds |
| Current Limiter | adm1177 | adm1177-iio |

**Note**: LibreSDR uses the same IIO architecture as PlutoSDR.

---

## 2. Device Hierarchy

### 2.1 Device Tree Structure

```dts
fpga-axi@0 {
    rx_dma: dma-controller@7c400000 { ... };
    tx_dma: dma-controller@7c420000 { ... };
    cf_ad9361-lpc@79020000 { ... };
    cf_ad9361-dds-core@79024000 { ... };
};

spi0 {
    ad9361-phy@0 { ... };
};
```

### 2.2 Driver Binding

```
ad9361-phy (ad9361 driver)
    |
    +-- Probes SPI device
    +-- Registers with IIO
    +-- Provides sampling clocks
    |
    v
cf-ad9361-lpc (axi_ad9361 driver)
    |
    +-- Links to ad9361-phy
    +-- Registers DMA channels
    +-- Creates buffered IIO device
    |
    v
DMA (axi_dmac driver)
    |
    +-- Handles sample transfer
    +-- Interrupt-driven
```

---

## 3. AD9361 Driver Architecture

### 3.1 Driver Source

- **File**: `linux/drivers/iio/adc/ad9361.c`
- **Size**: ~9700 lines
- **License**: GPL

### 3.2 Core Functions

```c
// Initialization
ad9361_probe()      // SPI device registration
ad9361_remove()     // Cleanup

// Configuration
ad9361_set_rx_freq()        // Set RX LO
ad9361_set_tx_freq()        // Set TX LO
ad9361_set_bb_rate()        // Set baseband rate
ad9361_set_gain_ctrl_mode() // AGC mode
ad9361_set_rx_gain()        // Manual gain

// Calibration
ad9361_rf_port_setup()      // RF port config
ad9361_calibrate()          // Run calibration
```

---

## 4. AXI AD9361 Driver

### 4.1 ADC Core

```dts
cf_ad9361-lpc@79020000 {
    compatible = "adi,axi-ad9361-6.00.a";
    dmas = <&rx_dma 0>;
    dma-names = "rx";
};
```

**Driver**: `linux/drivers/iio/adc/axi_ad9361_core.c`

### 4.2 DAC/DDS Core

```dts
cf_ad9361-dds-core@79024000 {
    compatible = "adi,axi-ad9364-dds-6.00.a";
    dmas = <&tx_dma 0>;
    dma-names = "tx";
};
```

**Driver**: `linux/drivers/iio/frequency/cf_axi_dds.c`

---

## 5. DMA Architecture

### 5.1 AXI DMAC

```dts
rx_dma: dma-controller@7c400000 {
    compatible = "adi,axi-dmac-1.00.a";
    interrupts = <0 57 IRQ_TYPE_LEVEL_HIGH>;
};

tx_dma: dma-controller@7c420000 {
    compatible = "adi,axi-dmac-1.00.a";
    interrupts = <0 56 IRQ_TYPE_LEVEL_HIGH>;
};
```

**Driver**: `linux/drivers/dma/dw-axi-dmac/`

### 5.2 DMA Operations

```
RX Path:
  AD9363 ADC --> FPGA Buffer --> DMA --> DDR Buffer --> Userspace

TX Path:
  Userspace --> DDR Buffer --> DMA --> FPGA Buffer --> AD9363 DAC
```

### 5.3 High-Bandwidth Mode

LibreSDR supports higher sample rates due to:
- Larger DDR bandwidth (1 GB @ 525-750 MHz)
- Gigabit Ethernet for network streaming
- LVDS mode for max data rates

---

## 6. IIO Device Attributes

### 6.1 ad9361-phy Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| out_altvoltage0_RX_LO_frequency | RW | RX LO (Hz) |
| out_altvoltage0_TX_LO_frequency | RW | TX LO (Hz) |
| out_voltage0_rf_bandwidth | RW | RF bandwidth (Hz) |
| out_voltage0_sampling_frequency | RW | Sample rate (Hz) |
| in_voltage0_gain_control_mode | RW | AGC mode |
| in_voltage0_hardwaregain | RW | RX gain (dB) |
| in_temp0_raw | R | Temperature |
| xo_correction | RW | XO correction value |

### 6.2 ADC Core Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| buffer/length | RW | Buffer size |
| buffer/enable | RW | Enable/disable buffer |
| in_voltage0_i | R | I channel data |
| in_voltage1_i | R | Q channel data |

### 6.3 DAC Core Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| buffer/length | RW | Buffer size |
| buffer/enable | RW | Enable/disable buffer |
| out_voltage0 | RW | TX data (I) |
| out_voltage1 | RW | TX data (Q) |

---

## 7. libiio Integration

### 7.1 libiio Architecture

```
Userspace Application
        |
        v
    libiio
        |
        +-- USB backend
        +-- Network backend
        +-- Local backend
        |
        v
    IIO Kernel
        |
        v
    Hardware
```

### 7.2 libiio Functions

```c
// Context
iio_create_context()       // Create IIO context
iio_context_destroy()       // Destroy context

// Device
iio_context_get_device_by_id()  // Get device by ID
iio_device_get_channel()        // Get channel

// Buffer
iio_device_create_buffer()   // Create buffer
iio_device_enable_buffer()   // Enable buffer
iio_device_dequeue_buffer()  // Get samples

// Attributes
iio_device_attr_read()      // Read attribute
iio_device_attr_write()     // Write attribute
```

---

## 8. iiod Daemon

### 8.1 IIO Daemon Purpose

The iiod daemon exposes IIO devices over network:
- Network backend for remote access
- USB backend for USB-connected devices
- Context sharing

### 8.2 iiod Startup

```bash
# Via tezuka init script
/usr/bin/iiod -F

# Options:
# -F: Foreground mode
# -n <uri>: Network URI
# -u <uri>: USB URI
```

### 8.3 Connection

```bash
# List available contexts
iio_info -s

# Connect via IP
iio_info -u ip:192.168.1.10

# Connect via USB
iio_info -u usb:1.2.3
```

---

## 9. Python Bindings (pyadi-iio)

### 9.1 adi.Pluto Class

```python
import adi

# Create device
sdr = adi.Pluto()

# Configure
sdr.rx_lo = 2400000000
sdr.tx_lo = 2450000000
sdr.rx_rf_bandwidth = 18000000
sdr.sample_rate = 30720000

# Receive
rx_data = sdr.rx()

# Transmit
sdr.tx(tx_data)
```

### 9.2 LibreSDR Compatibility

LibreSDR is fully compatible with PlutoSDR API via libiio/pyadi-iio.

---

## 10. Buffer Data Format

### 10.1 Sample Format

Complex samples (I/Q format):
- Each sample: 2 values (I, Q)
- Each value: 12-bit ADC (stored as 16-bit)
- Total: 4 bytes per sample

### 10.2 Data Layout

```
Buffer: [I0, Q0, I1, Q1, I2, Q2, ...]
         4B   4B   4B   4B   4B   4B
```

### 10.3 Buffer Sizes for High Sample Rates

```bash
# For 20+ MSPS, increase buffer size
echo 65536 > /sys/bus/iio/devices/iio:device1/buffer/length

# Available sizes: 2^n (512, 1024, 2048, 4096, 8192, 16384, 32768, 65536)
```

---

## 11. Debug Interface

### 11.1 IIO DebugFS

```bash
# List IIO devices
ls /sys/bus/iio/devices/

# Device directories
/sys/bus/iio/devices/iio:device0/  # ad9361-phy
/sys/bus/iio/devices/iio:device1/  # cf-ad9361-lpc
/sys/bus/iio/devices/iio:device2/  # cf-ad9361-dds

# Debug attributes
ls /sys/kernel/debug/iio/
```

### 11.2 Register Access

```bash
# Read register
cat /sys/bus/iio/devices/iio:device0/regions/registers
echo 0x009 > /sys/bus/iio/devices/iio:device0/regions/registers

# Via iio_reg tool
iio_reg ad9361-phy 0x009 1
```

---

## 12. Performance

### 12.1 Latency

| Path | Typical Latency |
|------|----------------|
| RF -> ADC -> DMA -> DDR | ~1-2 us |
| DDR -> DMA -> DAC -> RF | ~1-2 us |
| Network roundtrip (GbE) | ~50-200 us |

### 12.2 Throughput

| Operation | Rate |
|-----------|------|
| Max sample rate | 61.44 MSPS |
| Max bandwidth | 56 MHz |
| Gigabit Ethernet | ~900 Mbps |
| USB 2.0 throughput | ~40 MB/s |

### 12.3 LibreSDR Advantage

Due to Gigabit Ethernet and larger memory:
- Sustained 20-27.5 MSPS streaming
- Larger buffer sizes supported
- Less packet loss at high rates

---

## 13. Related Documents

- `LIBRESDR_AD936X_CONFIG.md` - AD936x configuration
- `LIBRESDR_DEVICE_TREE.md` - Device tree mapping
- `LIBRESDR_FPGA_ARCHITECTURE.md` - FPGA datapath
- `LIBRESDR_SYSTEM_ARCHITECTURE.md` - System overview
- `LIBRESDR_PERFORMANCE.md` - Performance analysis
