# LIBRESDR DEVICE TREE ANALYSIS

## Engineering Reference - LibreSDR Device Tree Mapping

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | DEVICE TREE MAPPING |
| **Last Updated** | 2026-09-07 |
| **Source** | F5OEO/tezuka_fw, board/tezuka/libre/dts/ |

---

## 1. Device Tree Files

### 1.1 LibreSDR Device Tree Files

| File | Description |
|------|-------------|
| `zynq-7000.dtsi` | Base Zynq-7000 include |
| `zynq-libre.dtsi` | LibreSDR specific includes |
| `zynq-libre.dts` | LibreSDR main device tree |

**Source Location**: `board/tezuka/libre/dts/`

### 1.2 Build Output

| DTB File | Source | Size |
|----------|--------|------|
| `devicetree.dtb` | zynq-libre.dts | ~22-24 KB |

---

## 2. Base Device Tree Structure

### 2.1 Memory Configuration

```dts
memory {
    device_type = "memory";
    reg = <0x00000000 0x40000000>;  /* 1 GB DDR3 */
};
```

**Key Difference from PlutoSDR**: LibreSDR has 1 GB (0x40000000) vs PlutoSDR's 512 MB (0x20000000)

### 2.2 Clock Configuration

```dts
clocks {
    ad9364_clkin: clock@0 {
        #clock-cells = <0>;
        compatible = "adjustable-clock";
        clock-frequency = <40000000>;  /* 40 MHz TCXO */
        clock-accuracy = <200000>;    /* 200 ppm */
    };
};
```

---

## 3. QSPI Flash Configuration

### 3.1 Flash Partitions

```dts
&qspi {
    status = "okay";
    is-dual = <0>;
    num-cs = <1>;

    primary_flash: ps7-qspi@0 {
        #address-cells = <1>;
        #size-cells = <1>;
        compatible = "jedec,spi-nor";
        reg = <0x0>;
        spi-max-frequency = <104000000>;

        partition@qspi-fsbl-uboot {
            label = "qspi-fsbl-uboot";
            reg = <0x0 0x100000>;
        };
        partition@qspi-uboot-env {
            label = "qspi-uboot-env";
            reg = <0x100000 0x20000>;
        };
        partition@qspi-nvmfs {
            label = "qspi-nvmfs";
            reg = <0x120000 0xE0000>;
        };
        partition@qspi-linux {
            label = "qspi-linux";
            reg = <0x200000 0x1E00000>;
        };
    };
};
```

**Note**: Flash layout is similar to PlutoSDR but LibreSDR boots primarily from SD card.

---

## 4. FPGA AXI Bus Configuration

### 4.1 AXI Peripheral Addresses

```dts
fpga_axi: fpga-axi@0 {
    compatible = "simple-bus";
    #address-cells = <0x1>;
    #size-cells = <0x1>;
    ranges;

    axi_i2c0: i2c@41600000 {
        compatible = "xlnx,axi-iic-1.02.a";
        reg = <0x41600000 0x10000>;
        interrupt-parent = <&intc>;
        interrupts = <0 59 IRQ_TYPE_LEVEL_HIGH>;
    };

    rx_dma: dma-controller@7c400000 {
        compatible = "adi,axi-dmac-1.00.a";
        reg = <0x7c400000 0x1000>;
        #dma-cells = <1>;
        interrupts = <0 57 IRQ_TYPE_LEVEL_HIGH>;
    };

    tx_dma: dma-controller@7c420000 {
        compatible = "adi,axi-dmac-1.00.a";
        reg = <0x7c420000 0x1000>;
        #dma-cells = <1>;
        interrupts = <0 56 IRQ_TYPE_LEVEL_HIGH>;
    };

    cf_ad9364_adc_core_0: cf-ad9361-lpc@79020000 {
        compatible = "adi,axi-ad9361-6.00.a";
        reg = <0x79020000 0x6000>;
        dmas = <&rx_dma 0>;
        dma-names = "rx";
    };

    cf_ad9364_dac_core_0: cf-ad9361-dds-core-lpc@79024000 {
        compatible = "adi,axi-ad9364-dds-6.00.a";
        reg = <0x79024000 0x1000>;
        dmas = <&tx_dma 0>;
        dma-names = "tx";
    };
};
```

**Note**: Addresses are the same as PlutoSDR since they share the same FPGA design base.

---

## 5. AD9363 Configuration

### 5.1 SPI Device Node

```dts
&spi0 {
    status = "okay";

    adc0_ad9364: ad9361-phy@0 {
        #address-cells = <1>;
        #size-cells = <0>;
        #clock-cells = <1>;
        compatible = "adi,ad9363a";
        reg = <0>;
        spi-cpha;
        spi-max-frequency = <10000000>;

        clocks = <&ad9364_clkin 0>;
        clock-names = "ad9364_ext_refclk";
        clock-output-names = "rx_sampl_clk", "tx_sampl_clk";

        /* Digital Interface Configuration */
        adi,pp-tx-swap-enable;
        adi,pp-rx-swap-enable;
        adi,rx-frame-pulse-mode-enable;
        adi,xo-disable-use-ext-refclk-enable;
        adi,full-port-enable;
        adi,digital-interface-tune-fir-disable;
        adi,tx-fb-clock-delay = <0>;
        adi,tx-data-delay = <9>;

        /* RF Configuration */
        adi,rx-rf-port-input-select = <0>;
        adi,tx-rf-port-input-select = <0>;
        adi,tx-attenuation-mdB = <10000>;

        /* Bandwidth */
        adi,rf-rx-bandwidth-hz = <18000000>;
        adi,rf-tx-bandwidth-hz = <18000000>;

        /* Default Frequencies */
        adi,rx-synthesizer-frequency-hz = /bits/ 64 <2400000000>;
        adi,tx-synthesizer-frequency-hz = /bits/ 64 <2450000000>;

        /* AGC Configuration */
        adi,gc-rx1-mode = <2>;
        adi,gc-rx2-mode = <2>;

        /* GPIO Configuration */
        en_agc-gpios = <&gpio0 66 0>;
        reset-gpios = <&gpio0 67 0>;
    };
};
```

---

## 6. Network Configuration

### 6.1 Ethernet

LibreSDR uses Gigabit Ethernet (vs PlutoSDR's 100M):

```dts
&gem0 {
    status = "okay";
    phy-mode = "rgmii-id";
    phy-handle = <&ethernet_phy>;
};

&gem0 {
    local-mac-address = [00 0a 35 00 01 22];
};
```

### 6.2 Default IP Settings

| Parameter | LibreSDR | PlutoSDR |
|-----------|----------|----------|
| Device IP | 192.168.1.10 | 192.168.2.1 |
| Host IP | DHCP/Static | 192.168.2.10 |

---

## 7. SD Card Configuration

### 7.1 SD Card Boot Support

LibreSDR supports SD card as primary boot source:

```dts
&sdhci0 {
    status = "okay";
    clock-frequency = <50000000>;
};
```

---

## 8. Device Tree to Driver Mapping

### 8.1 I2C Bus

| Address | Device | Driver | Description |
|---------|--------|--------|-------------|
| 0x41600000 | axi_i2c0 | xilinx-axi-iic | AXI I2C master |

### 8.2 DMA Controllers

| Address | Node | Driver | IRQ | Purpose |
|---------|------|--------|-----|---------|
| 0x7C400000 | rx_dma | axi_dmac | 57 | RX sample DMA |
| 0x7C420000 | tx_dma | axi_dmac | 56 | TX sample DMA |

### 8.3 AD9363 Interface

| Address | Node | Driver | Description |
|---------|------|--------|-------------|
| 0x79020000 | cf_ad9364_adc_core_0 | axi_ad9361 | ADC core |
| 0x79024000 | cf_ad9364_dac_core_0 | axi_ad9361 | DAC/DDS core |

---

## 9. GPIO Assignments

### 9.1 LibreSDR GPIO Map

| GPIO | Function |
|------|----------|
| 14 | Button (if present) |
| 15 | LED (green) |
| 52 | USB PHY reset |
| 66 | EN_AGC |
| 67 | AD9363 reset |

**Note**: LibreSDR may vary by board revision from different manufacturers.

---

## 10. Interrupt Assignments

| IRQ | Source | Description |
|-----|--------|-------------|
| 56 | TX DMA | Transmit DMA |
| 57 | RX DMA | Receive DMA |
| 59 | I2C | AXI I2C controller |

---

## 11. Clock Configuration

### 11.1 LibreSDR Clock Tree

| Clock | Source | Default Frequency | Overclock Frequency |
|-------|--------|-------------------|---------------------|
| CPU | ARM PLL | 750 MHz | 1100 MHz |
| DDR | DDR PLL | 525 MHz | 750 MHz |
| Reference | TCXO | 40 MHz | 40 MHz |

### 11.2 Clock Assignments

| Clock | Frequency | Purpose |
|-------|-----------|---------|
| ad9364_ext_refclk | 40 MHz | AD9363 reference |
| arm_clk | 750 MHz | ARM Cortex-A9 |
| ddr_clk | 525 MHz | DDR controller |
| rx_sampl_clk | Variable | RX sample clock |
| tx_sampl_clk | Variable | TX sample clock |

---

## 12. Comparison with PlutoSDR Device Tree

| Parameter | LibreSDR | PlutoSDR |
|-----------|----------|----------|
| Memory | 1 GB | 512 MB |
| CPU Clock | 750 MHz | 666 MHz |
| DDR Clock | 525 MHz | 533 MHz |
| Ethernet | GbE (rgmii-id) | 100M |
| Boot Mode | SD + QSPI | QSPI only |

---

## 13. Build Commands

```bash
# Build DTB
make zynq-libre.dtb

# Decompile DTB to DTS
dtc -I dtb -O dts build/devicetree.dtb > build/devicetree.dts
```

---

## 14. Related Documents

- `LIBRESDR_SYSTEM_ARCHITECTURE.md` - System overview
- `LIBRESDR_FPGA_ARCHITECTURE.md` - FPGA HDL mapping
- `LIBRESDR_AD936X_CONFIG.md` - AD936x configuration
- `LIBRESDR_OVERCLOCKING.md` - Overclocking details
