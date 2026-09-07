# LIBRESDR PERFORMANCE ANALYSIS

## Performance Benchmarks and Analysis

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | PERFORMANCE ANALYSIS |
| **Last Updated** | 2026-09-07 |
| **Source** | hz12opensource/libresdr benchmarks |

---

## 1. Sample Rate Performance

### 1.1 Maximum Sample Rates

| Configuration | CPU | DDR | Continuous Sample Rate | Notes |
|---------------|-----|-----|------------------------|-------|
| Stock PlutoSDR | 666 MHz | 525 MHz | 10-12 MSPS | Baseline |
| LibreSDR Base | 750 MHz | 525 MHz | 20 MSPS | Default OC |
| LibreSDR Medium | 950 MHz | 600 MHz | ~25 MSPS | Stable |
| LibreSDR Max | 1100 MHz | 750 MHz | 27.5 MSPS | Maximum tested |

### 1.2 Sample Rate Limitations

The achievable sample rate is limited by:

1. **CPU Processing**: Higher rates require faster CPU
2. **DDR Bandwidth**: Memory bandwidth for data transfer
3. **Network Speed**: GbE can handle ~900 Mbps
4. **USB Speed**: USB 2.0 limited to ~40 MB/s
5. **Host Computer**: Processing power for demodulation

---

## 2. Network Performance

### 2.1 Ethernet Throughput

| Metric | Value |
|--------|-------|
| Maximum Speed | 1 Gbps |
| Practical Speed | ~900 Mbps |
| Protocol | TCP/UDP |

### 2.2 Latency

| Metric | Value |
|--------|-------|
| Round-trip | ~1-2 ms |
| UDP jitter | <1 ms |

---

## 3. CPU Performance

### 3.1 Clock Speed Comparison

| Configuration | Clock | Relative Performance |
|---------------|-------|---------------------|
| Stock | 666 MHz | 1.0x |
| Base | 750 MHz | 1.13x |
| Medium | 950 MHz | 1.43x |
| Max | 1100 MHz | 1.65x |

### 3.2 Power Consumption

| Configuration | Relative Power |
|---------------|----------------|
| Stock | 1.0x |
| Base | 1.1x |
| Medium | 1.3x |
| Max | 1.5x |

---

## 4. Memory Performance

### 4.1 DDR Bandwidth

| Configuration | Speed | Bandwidth |
|---------------|-------|-----------|
| Base | 525 MHz | 4.2 GB/s |
| Medium | 600 MHz | 4.8 GB/s |
| Max | 750 MHz | 6.0 GB/s |

### 4.2 Memory Available

| Board | Total | Typical Usage | Available |
|-------|-------|---------------|-----------|
| LibreSDR | 1 GB | ~200 MB | ~800 MB |
| PlutoSDR | 512 MB | ~200 MB | ~312 MB |

---

## 5. FPGA Resource Usage

### 5.1 Zynq-7020 Resources

| Resource | Total | LibreSDR Base | Percentage |
|----------|-------|----------------|------------|
| LUTs | 53,200 | ~25,000 | ~47% |
| Flip-Flops | 106,400 | ~35,000 | ~33% |
| BRAM | 4.9 Mb | ~2 Mb | ~41% |
| DSP Slices | 220 | ~40 | ~18% |

### 5.2 Available for Custom Logic

| Resource | Available |
|----------|-----------|
| LUTs | ~28,000 |
| Flip-Flops | ~71,000 |
| BRAM | ~2.9 Mb |
| DSP Slices | ~180 |

---

## 6. Thermal Performance

### 6.1 Temperature Ranges

| State | Temperature |
|-------|-------------|
| Idle | 40-50C |
| Normal Operation | 50-60C |
| Heavy Load | 60-75C |
| Max Safe | 85C |

### 6.2 Thermal Considerations

- Default clock speeds maintain safe temperatures
- Overclocking may require additional cooling
- Heatsink recommended for maximum overclock

---

## 7. Comparison with Other Platforms

### 7.1 Sample Rate Comparison

| Platform | Max Sample Rate | Notes |
|----------|-----------------|-------|
| RTL-SDR | 3.2 MSPS | USB 2.0 |
| HackRF | 20 MSPS | Full duplex |
| PlutoSDR | 12 MSPS | Stock |
| **LibreSDR** | **27.5 MSPS** | **Overclocked** |
| USRP B210 | 61.44 MSPS | USB 3.0 |

### 7.2 Value Performance

| Platform | Price | $/MSPS |
|----------|-------|--------|
| RTL-SDR | $25 | $7.81 |
| HackRF | $300 | $15.00 |
| PlutoSDR | $149 | $12.42 |
| **LibreSDR** | **$149** | **$5.42** |
| USRP B210 | $1,100 | $17.90 |

---

## 8. Real-World Benchmarks

### 8.1 SDR++ Testing

Test conditions:
- Gigabit Ethernet connection
- Local loopback for minimal latency
- SDR++ with increased buffer size

| Configuration | Stable Sample Rate |
|---------------|-------------------|
| Base (750/525) | 20 MSPS continuous |
| Overclocked (1100/750) | 27.5 MSPS continuous |

### 8.2 Network Streaming

Test with iperf3:
- TCP window size: 256 KB
- Duration: 60 seconds

| Configuration | Throughput |
|---------------|------------|
| LibreSDR | ~940 Mbps |
| PlutoSDR | ~94 Mbps |

---

## 9. Optimization Tips

### 9.1 Host-Side Optimizations

1. **Use Gigabit Ethernet**: Essential for high sample rates
2. **Dedicated NIC**: Separate NIC for SDR traffic
3. **Large Buffer Sizes**: Increase IIO buffer size
4. **UDP over TCP**: Lower overhead
5. **Disable CPU throttling**: Consistent performance

### 9.2 Device-Side Optimizations

1. **Use LVDS Mode**: For max sample rates
2. **Enable jumbo frames**: Larger MTU on Ethernet
3. **Disable unused services**: Reduce CPU load

---

## 10. Future Performance Potential

### 10.1 Theoretical Limits

| Metric | Theoretical Max | Current Achievement |
|--------|-----------------|---------------------|
| Sample Rate | 61.44 MSPS | 27.5 MSPS (45%) |
| Ethernet | 1 Gbps | 940 Mbps (94%) |
| CPU Utilization | 100% | ~60% at max |

### 10.2 Potential Improvements

1. **Custom FPGA bitstream**: Optimize data paths
3. **Improved DDR timing**: Better memory margins
4. **Voltage adjustments**: May enable higher clocks
