# LIBRESDR ROOT FILESYSTEM ANALYSIS

## Engineering Reference - Root Filesystem Structure

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | ROOTFS_ANALYSIS |
| **Last Updated** | 2026-09-07 |
| **Source** | tezuka_fw buildroot, F5OEO/tezuka_fw |

---

## 1. Root Filesystem Overview

### 1.1 Build Information

LibreSDR uses the same root filesystem structure as PlutoSDR, built with Buildroot:

| Component | Source |
|-----------|--------|
| Buildroot | adi-2026.02-y |
| Linux Kernel | adi-xilinx-2026r1 |
| U-Boot | adi-xlnx-u-boot-2025.1-y |

### 1.2 RootFS Statistics

| Metric | LibreSDR | PlutoSDR |
|--------|----------|----------|
| Compressed size | ~15 MB | ~5-6 MB |
| Uncompressed | ~40-50 MB | ~40 MB |
| Total files | ~200-300 | ~233 |

---

## 2. Directory Structure

### 2.1 Top-level Directories

```
rootfs/
|-- bin/           (BusyBox utilities - symlinks)
|-- boot/         (Boot files - SD card)
|-- dev/           (Device nodes)
|-- etc/           (Configuration files)
|-- home/          (User home directories)
|-- lib/           (Shared libraries + firmware)
|-- media/         (Removable media)
|-- mnt/           (Mount points)
|   |-- jffs2/     (Persistent storage)
|   `-- msd/        (USB mass storage)
|-- opt/            (Optional software)
|-- proc/          (Virtual process filesystem)
|-- root/          (Root home directory)
|-- run/           (Runtime data)
|-- sbin/          (System binaries)
|-- sys/           (Sysfs)
|-- tmp/           (Temporary files)
|-- usr/           (User programs)
|   |-- bin/       (User binaries)
|   |-- lib/       (User libraries)
|   |-- sbin/      (User system binaries)
|   `-- share/     (Shared data)
|-- var/           (Variable data)
|   |-- cache/     (Application cache)
|   |-- lib/       (Persistent state)
|   |-- log/       (Log files)
|   `-- www/       (Web server root)
`-- www/           (Web interface - Maia-SDR Dashboard)
```

---

## 3. /etc Configuration

### 3.1 Init Scripts (tezuka_fw)

| Script | Purpose | Order |
|--------|---------|-------|
| S01syslogd | System logging | 1 |
| S02klogd | Kernel logging | 2 |
| S02sysctl | Sysctl settings | 2 |
| S05avahi-setup.sh | Avahi setup | 5 |
| S10mdev | Hotplug/mdev | 10 |
| S15watchdog | Watchdog | 15 |
| S20urandom | Random seed | 20 |
| S21misc | Miscellaneous | 21 |
| S23udc | USB gadget | 23 |
| S30dbus | D-Bus daemon | 30 |
| S40network | Network setup | 40 |
| S41network | Network (alt) | 41 |
| S45msd | Mass storage | 45 |
| S50avahi-daemon | Avahi daemon | 50 |
| S50dropbear | SSH server | 50 |
| S90maia | Maia-SDR | 90 |
| S95mqtt | MQTT broker | 95 |
| S96plutostream | DATV backend | 96 |
| S98autostart | User autorun | 98 |
| S99user | User scripts | 99 |

### 3.2 Key Configuration Files

| File | Description |
|------|-------------|
| `/etc/device_config` | Device configuration |
| `/etc/fstab` | Filesystem mount points |
| `/etc/fw_env.config` | U-Boot env partition config |
| `/etc/hostname` | Hostname (libre) |
| `/etc/hosts` | Host mappings |
| `/etc/inittab` | Init configuration |
| `/etc/passwd` | User accounts |
| `/etc/group` | User groups |

---

## 4. /opt Directory (tezuka_fw)

### 4.1 Contents

| File/Directory | Description |
|----------------|-------------|
| VERSIONS | Build version info |
| maia/ | Maia-SDR application |
| satdump/ | SatDump (optional) |

### 4.2 VERSIONS Content

```
device-fw tezuka
buildroot 2026.02
linux adi-xilinx-2026r1
u-boot adi-xlnx-2025.1-y
```

---

## 5. Network Configuration

### 5.1 Default Network Settings (LibreSDR)

```
Device IP:     192.168.1.10
Host IP:       DHCP or static
Netmask:       255.255.255.0
Gateway:       192.168.1.1
```

### 5.2 Files

- `/etc/network/interfaces` - Network interface config
- `/etc/udhcpd.conf` - DHCP server config
- `/etc/resolv.conf` - DNS servers

---

## 6. Boot Sequence (tezuka_fw)

### 6.1 Init Scripts Execution Order

```
1. /etc/init.d/rcS (runlevel S)
       |
       v
2. S01syslogd -> S02klogd -> S02sysctl
       |
       v
3. S10mdev (hotplug)
       |
       v
4. S15watchdog
       |
       v
5. S20urandom
       |
       v
6. S21misc
       |
       v
7. S23udc (USB gadget setup)
       |
       v
8. S30dbus
       |
       v
9. S40network -> S41network
       |
       v
10. S50avahi-daemon -> S50dropbear
       |
       v
11. S90maia (Maia-SDR)
       |
       v
12. S95mqtt (MQTT broker)
       |
       v
13. S96plutostream (DATV)
       |
       v
14. S98autostart
       |
       v
15. /etc/init.d/rcK (shutdown)
```

---

## 7. Key Services

### 7.1 iiod (IIO Daemon)

```bash
/usr/sbin/iiod -F

# Options:
# -F: Foreground mode
# -n <num>: Number of backend connections
```

### 7.2 dropbear (SSH Server)

```bash
/usr/sbin/dropbear -R

# SSH on port 22
# Root login enabled (password: analog)
```

### 7.3 avahi-daemon (mDNS)

```bash
/usr/sbin/avahi-daemon -D

# Enables libre.local mDNS name
```

### 7.4 MQTT Broker (tezuka)

```bash
# Mosquitto MQTT broker
mosquitto -c /etc/mosquitto.conf
```

### 7.5 Maia-SDR Dashboard

```bash
# Maia-HTTP web server
/usr/bin/maia-httpd -p 80 -h /opt/maia
```

---

## 8. Libraries

### 8.1 Key Libraries

| Library | Path | Description |
|---------|------|-------------|
| libiio | /usr/lib/libiio.so | IIO bindings |
| libad9361 | /usr/lib/libad9361.so | AD9361 support |
| libusb | /usr/lib/libusb-1.0.so | USB support |
| libxml2 | /usr/lib/libxml2.so | XML parsing |
| libz | /usr/lib/libz.so | Compression |
| libc | /lib/libuClibc.so | C library |

### 8.2 tezuka_fw Additional Libraries

| Library | Description |
|--------|-------------|
| libgse | DVB-S2 support |
| libNE10 | NEON optimized FFT |
| libmosquitto | MQTT client |
| libcurl | HTTP client |

---

## 9. tezuka_fw Specific Features

### 9.1 Maia-SDR Integration

```
/opt/maia/
|-- www/              # Web interface
|-- wasm/             # WebAssembly modules
|-- classifier/        # Signal classifier
`-- templates/        # Classification templates
```

### 9.2 MQTT Topics

| Topic | Description |
|-------|-------------|
| cmd/# | Command messages |
| state/# | Status messages |
| dt/<call>/# | DATV telemetry |

### 9.3 DATV Backend

```
/usr/bin/
|-- pluto_mqtt_ctrl    # MQTT control
|-- pluto_stream       # DVB-S2 streaming
`-- iio_ws_proxy      # WebSocket proxy
```

---

## 10. Persistent Storage

### 10.1 JFFS2 Filesystem

```
/mnt/jffs2/
|-- config.txt           # Runtime configuration
|-- bandplan.json        # RF band plan
`-- calibration/        # RF calibration data
```

### 10.2 U-Boot Environment

| Partition | Offset | Size |
|-----------|--------|------|
| qspi-uboot-env | 0x100000 | 128 KB |
| qspi-nvmfs | 0x120000 | 896 KB |

---

## 11. SD Card vs QSPI Boot

### 11.1 SD Card Boot

When booting from SD card:
- /boot is mounted from SD card partition 1
- Contains: BOOT.bin, image.ub, uEnv.txt

### 11.2 QSPI Boot

When booting from QSPI:
- Kernel and rootfs loaded from FIT image
- SD card can be used for data storage

---

## 12. Related Documents

- `LIBRESDR_BOOT_FLOW.md` - Boot sequence
- `LIBRESDR_IIO_ARCHITECTURE.md` - IIO stack
- `LIBRESDR_WEB_ARCHITECTURE.md` - Web interface
- `LIBRESDR_BUILD_SYSTEM.md` - Build system
