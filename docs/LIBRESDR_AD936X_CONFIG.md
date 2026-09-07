# LIBRESDR AD936X CONFIGURATION

## AD9363 RF Transceiver Configuration

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | AD936X CONFIGURATION |
| **Last Updated** | 2026-09-07 |

---

## 1. Overview

LibreSDR uses the Analog Devices AD9363 RF transceiver, the same chip used in PlutoSDR.

### 1.1 Key Features

| Parameter | Value |
|-----------|-------|
| Chip | AD9363 |
| Frequency Range | 325 MHz - 3.8 GHz |
| RX Channels | 1 or 2 |
| TX Channels | 1 or 2 |
| ADC Resolution | 12-bit |
| Max Sample Rate | 61.44 MSPS |
| Interface | LVDS or CMOS |
| Output Power | +7 dBm max |

---

## 2. Clock Architecture

### 2.1 Reference Clock

| Parameter | Value |
|-----------|-------|
| Source | External TCXO |
| Frequency | 40 MHz |
| Tolerance | +/- 2 ppm typical |

### 2.2 PLL Configuration

| Clock | Multiplier | Frequency |
|-------|------------|-----------|
| Reference | 1x | 40 MHz |
| BBPLL | 24.576x | 983.04 MHz |
| ADC | 6.144x | 245.76 MHz |
| DAC | 3.072x | 122.88 MHz |

### 2.3 Derived Clocks

| Clock | Frequency | Usage |
|-------|-----------|-------|
| DATA_CLK | 245.76 MHz | LVDS data clock |
| ADC Clock | 245.76 MHz | ADC sampling |
| DAC Clock | 122.88 MHz | DAC update |
| Rx Clock | 122.88 MHz | RX processing |

---

## 3. Register Configuration

### 3.1 Initialization Sequence

1. Reset chip
2. Configure reference clock
3. Setup PLLs
4. Configure filters
5. Enable RX/TX paths
6. Set LO frequencies

### 3.2 Key Registers

| Register | Address | Description |
|----------|---------|-------------|
| CLK_DET | 0x002 | Clock detection |
| BBPLL | 0x003-0x00E | BBPLL configuration |
| RX_REF | 0x0x0A | RX reference |
| RX_Nyquist | 0x101 | RX Nyquist filter |
| TX_Nyquist | 0x104 | TX Nyquist filter |
| RX_GAIN | 0x1F | RX gain control |
| TX_ATTEN | 0x073 | TX attenuation |
| RX_LO | 0x123 | RX LO frequency |
| TX_LO | 0x124 | TX LO frequency |

---

## 4. RX Configuration

### 4.1 RX Signal Chain

```
RF Input --> LNA --> Mixer --> BBF --> ADC --> LVDS Interface
```

### 4.2 Gain Control Modes

| Mode | Description | Typical Use |
|------|-------------|-------------|
| Manual | Fixed gain | Steady signals |
| Slow AGC | Automatic gain | Voice/data |
| Fast AGC | Rapid gain | Pulsed signals |
| Hybrid | Combined | General use |

### 4.3 Default RX Settings

| Parameter | Value |
|-----------|-------|
| LO Frequency | 2.4 GHz |
| Bandwidth | 18 MHz |
| Gain Mode | Slow AGC |
| AGC Set Point | -10 dBFS |
| Manual Gain | 50 dB |

---

## 5. TX Configuration

### 5.1 TX Signal Chain

```
LVDS Interface --> DAC --> Mixer --> PAF --> RF Output
```

### 5.2 Default TX Settings

| Parameter | Value |
|-----------|-------|
| LO Frequency | 2.45 GHz |
| Bandwidth | 18 MHz |
| Attenuation | 0 dB |
| TX Power | +7 dBm max |

### 5.3 Attenuation Control

Range: 0 to -89.75 dB in 0.25 dB steps

---

## 6. Filter Configuration

### 6.1 Baseband Filters

| Filter | Cutoff | Purpose |
|--------|--------|---------|
| BBF | Programmable | Anti-aliasing |
| RCF | Fixed | Reconstruction |
| TXFIR | Programmable | TX shaping |
| RXFIR | Programmable | RX filtering |

### 6.2 FIR Filters

| Filter | Taps | Bandwidth Options |
|--------|------|-------------------|
| RX FIR | 64 | 1.92 - 56 MHz |
| TX FIR | 64 | 1.92 - 56 MHz |

---

## 7. Interface Configuration

### 7.1 LVDS Mode

For maximum sample rates (up to 61.44 MSPS):

| Parameter | Value |
|-----------|-------|
| Interface | LVDS |
| DDR Mode | Enabled |
| Data Clock | 245.76 MHz |
| LVDS Levels | 1.8V |

### 7.2 Signal Mapping

```
P/N Pairs:
- DATA[5:0]P/N: I data
- DATA[11:6]P/N: Q data
- FB_CLK: Feedback clock
- RX_FRAME: Frame sync
- TX_DATA: Output enable
```

---

## 8. Frequency Planning

### 8.1 LO Frequency Range

| Parameter | Min | Max |
|-----------|-----|-----|
| RX LO | 325 MHz | 3.8 GHz |
| TX LO | 325 MHz | 3.8 GHz |

### 8.2 Sample Rate Options

| Sample Rate | BBPLL Div | ADC Div |
|-------------|------------|---------|
| 30.72 MHz | 32 | 8 |
| 20.48 MHz | 48 | 12 |
| 10.24 MHz | 96 | 24 |
| 3.84 MHz | 256 | 64 |

---

## 9. Power Management

### 9.1 Power Modes

| Mode | Power | Use Case |
|------|-------|----------|
| Full Power | 1.5W | Active RX/TX |
| Sleep | 50 mW | Standby |
| Alert | 200 mW | Quick wake |
| MIMO | 2W | 2T2R mode |

### 9.2 Power Supplies

| Rail | Voltage | Current |
|------|---------|---------|
| DVDD | 1.3V | 300 mA |
| AVDD | 3.3V | 450 mA |
| VIO | 1.8V | 50 mA |

---

## 10. Calibration

### 10.1 Auto Calibration

| Calibration | Trigger | Duration |
|-------------|---------|----------|
| DCXO | On init | 10 ms |
| TX MON | On init | 5 ms |
| RX GDC | Manual | 20 ms |
| RX QEC | Manual | 50 ms |
| TX QEC | Manual | 50 ms |

### 10.2 Manual Calibration Commands

```bash
# Via libiio
iio_attr -C ad9361-phy voltage0 rf_port_select "A_BALANCED"
iio_attr -C ad9361-phy voltage0 filter_fir_en 1
```

---

## 11. Device Tree Configuration

### 11.1 AD9363 Node

```dts
&ad9361_phandle {
    compatible = "adi,ad9363";
    reg = <0 0>;
    clocks = <&clkc 0>;
    clock-names = "ref_clk";

    adi,tx-lo-clk-free-run;
    adi,rx-synth-lon-freq-hz = <2400000000>;
    adi,tx-synth-lon-freq-hz = <2450000000>;
};
```

---

## 12. Troubleshooting

### 12.1 Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| No RX | LO not set | Set LO frequency |
| No TX | Power down | Enable TX |
| Low SNR | Gain too low | Increase gain |
| spurs | DC offset | Run calibration |
