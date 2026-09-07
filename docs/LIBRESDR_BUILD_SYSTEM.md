# LIBRESDR BUILD SYSTEM

## Build System and Firmware Compilation

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | BUILD SYSTEM |
| **Last Updated** | 2026-09-07 |
| **Source** | F5OEO/tezuka_fw |

---

## 1. Build System Overview

LibreSDR firmware can be built using:

1. **tezuka_fw** (Recommended): Universal firmware builder
2. **hz12opensource/libresdr**: Original overclock-focused build
3. **Manual build**: From ADI plutosdr-fw with patches

---

## 2. tezuka_fw Build System

### 2.1 Prerequisites

```bash
# Install dependencies (Ubuntu/Debian)
sudo apt install pkg-config libssl-dev libclang-dev jq

# Additional for Vivado builds
sudo apt install libgmp-dev libmpc-dev git build-essential \
    fakeroot libncurses5-dev libssl-dev ccache dfu-util \
    u-boot-tools device-tree-compiler mtools bc python3 \
    cpio zip unzip rsync file wget flex bison
```

### 2.2 Setup

```bash
# Clone repository
git clone https://github.com/F5OEO/tezuka_fw.git
cd tezuka_fw

# Pull Buildroot
./getbuildroot.sh

# Source environment
source sourceme.first
```

### 2.3 Build Commands

```bash
# Build LibreSDR firmware
./build.sh libre

# Build with custom clock settings
OVERCLOCK_CPU_MULT=44 OVERCLOCK_DDR_MULT=30 ./build.sh libre

# Clean and rebuild
./build.sh -c libre

# Parallel build
./build.sh -j8 libre
```

### 2.4 Output

```
output/libre/images/
|-- BOOT.BIN
|-- image.ub
|-- pluto.frm
|-- boot.frm
|-- *.dfu
|-- sdimg/           # SD card contents
```

---

## 3. hz12opensource/libresdr Build

### 3.1 Prerequisites

```bash
# Install Vivado 2022.2
# Download from Xilinx website

# Additional packages
sudo apt install libgmp-dev libmpc-dev fakeroot \
    libncurses5-dev libssl-dev ccache dfu-util \
    u-boot-tools device-tree-compiler mtools bc \
    python3 cpio zip unzip rsync file wget \
    flex bison language-pack-en libtinfo5 \
    x11-utils xvfb dbus-x11
```

### 3.2 Build Process

```bash
# Clone original PlutoSDR firmware
git clone --branch v0.38 --recursive \
    https://github.com/analogdevicesinc/plutosdr-fw.git \
    plutosdr-fw_0.38_libre

cd plutosdr-fw_0.38_libre

# Apply LibreSDR patches
./apply.sh

# Set Vivado path
export VIVADO_SETTINGS=/opt/Xilinx/Vivado/2022.2/settings64.sh
export TARGET=libre

# Build firmware
make

# Build SD card image
make sdimg
```

### 3.3 Overclock Build

```bash
# Set overclock multipliers
export OVERCLOCK_CPU_MULT=44
export OVERCLOCK_DDR_MULT=30

# Build with overclock
make overclock
make sdimg
```

---

## 4. Build Configuration

### 4.1 Defconfig

LibreSDR uses: `libre_maiasdr_defconfig`

```bash
# Buildroot configuration
source sourceme.first
make -C buildroot O=output/libre libre_maiasdr_defconfig
make -C buildroot O=output/libre
```

### 4.2 Kernel Configuration

Kernel config is in: `board/tezuka/libre/kernel/zynq_pluto_linux_defconfig`

### 4.3 U-Boot Configuration

U-Boot config: `board/tezuka/libre/u-boot-config/zynq_pluto_defconfig`

---

## 5. Source Structure

### 5.1 LibreSDR Board Directory

```
board/tezuka/libre/
|-- bitstream/
|   |-- tezuka.xsa              # FPGA project export
|   |-- fsbl.elf                # FSBL
|   `-- overclock/              # Prebuilt FSBL variants
|-- dts/
|   |-- zynq-libre.dts         # Device tree source
|   `-- zynq-libre.dtsi        # Device tree includes
|-- kernel/
|   `-- zynq_pluto_linux_defconfig
|-- u-boot-config/
|   `-- zynq_pluto_defconfig
|-- u-boot-dts/
|   `-- zynq-pluto-sdr.dts     # U-Boot device tree
|-- uboot-env.txt               # U-Boot environment
`-- pluto.its                   # FIT image template
```

---

## 6. Build Artifacts

### 6.1 Generated Files

| File | Size | Description |
|------|------|-------------|
| BOOT.BIN | ~2-3 MB | Boot image |
| image.ub | ~15 MB | FIT image |
| pluto.frm | ~12 MB | Update package |
| boot.frm | ~600 KB | Boot update |

### 6.2 SD Card Contents

```
sdimg/
|-- BOOT.BIN
|-- image.ub
|-- uEnv.txt
|-- boot.scr
```

---

## 7. Troubleshooting

### 7.1 Common Issues

| Issue | Solution |
|-------|----------|
| CMake policy errors | Set `CMAKE_POLICY_VERSION_MINIMUM=3.5` |
| Missing dependencies | Install packages from prerequisites |
| Vivado license | Use Vivado WebPACK (free) |
| Memory errors | Increase RAM/disable parallel builds |

### 7.2 Build Logs

```bash
# Enable verbose build
./build.sh -j1 libre 2>&1 | tee build.log

# Check for errors
grep -i error build.log
```

---

## 8. Customization

### 8.1 Device Tree Modifications

Edit: `board/tezuka/libre/dts/zynq-libre.dts`

### 8.2 Kernel Config Changes

Edit: `board/tezuka/libre/kernel/zynq_pluto_linux_defconfig`

### 8.3 U-Boot Environment

Edit: `board/tezuka/libre/uboot-env.txt`

---

## 9. CI/CD

### 9.1 GitHub Actions

tezuka_fw uses GitHub Actions for CI:
- Matrix build for all boards
- Artifact generation
- Release publishing

### 9.2 Local CI

```bash
# Test build locally
./build.sh libre

# Verify output
ls -la output/libre/images/
```
