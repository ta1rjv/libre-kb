# LIBRESDR WEB ARCHITECTURE

## Engineering Reference - Web Interface Architecture

---

## Document Information

| Field | Value |
|-------|-------|
| **Status** | ENGINEERING REFERENCE |
| **Type** | WEB INTERFACE |
| **Last Updated** | 2026-09-07 |
| **Source** | tezuka_fw Dashboard, F5OEO/tezuka_fw |

---

## 1. Overview

LibreSDR uses the tezuka firmware which includes the Maia-SDR Dashboard, a modern web-based interface for controlling the SDR.

### 1.1 Web Stack Components

| Component | Technology |
|-----------|------------|
| HTTP Server | Custom maia-httpd (Rust) |
| Web Interface | React/JavaScript |
| MQTT Broker | Mosquitto |
| WebSocket | MQTT over WebSocket |
| API | REST over MQTT |

### 1.2 Web Root

```
/opt/maia/
|-- www/                # Web interface
|-- wasm/              # WebAssembly modules
|-- classifier/         # Signal classifier
`-- templates/         # Classification templates
```

---

## 2. File Structure

### 2.1 Build-Time Generation

Source files are located in:
```
tezuka_fw/
|-- Dashboard/
|   |-- Tezuka Dashboard.html
|   |-- Tezuka Spectrum.html
|   |-- app.jsx
|   |-- pages*.jsx
|   |-- charts.jsx
|   |-- styles.css
|   `-- data.jsx
```

### 2.2 Runtime Location

```
/opt/maia/
|-- index.html              # Dashboard main page
|-- spectrum.html            # Spectrum analyzer
|-- app.js                  # Compiled JS bundle
|-- wasm/                   # WebAssembly modules
`-- assets/                 # Static assets
```

---

## 3. HTTP Server Configuration

### 3.1 Server Startup

```bash
# From tezuka init script
/usr/bin/maia-httpd -p 80 -h /opt/maia
```

### 3.2 Features

- WebSocket support for MQTT
- HTTPS (optional)
- Static file serving
- CORS headers
- SPA (Single Page Application) support

---

## 4. Dashboard Pages

### 4.1 Available Pages

| URL | Content |
|-----|---------|
| `/` | Main dashboard |
| `/spectrum` | Spectrum analyzer |
| `#datv` | DATV Controller |
| `#network` | Network settings |
| `#rf` | RF settings |
| `#signal-classifier` | Signal classifier |

### 4.2 Dashboard Sections

```
Main Dashboard
|-- Network Status
|-- RF Controls
|   |-- RX/TX Frequency
|   |-- Gain Settings
|   |-- Sample Rate
|-- Spectrum View
|-- DATV Controller
|-- Signal Classifier
|-- Band Plan
|-- VPN Configuration
```

---

## 5. MQTT Architecture

### 5.1 MQTT Topics

| Topic | Direction | Description |
|-------|-----------|-------------|
| cmd/# | Host -> Device | Commands |
| state/# | Device -> Host | Status |
| waterfall/# | Device -> Host | FFT data |
| classifier/# | Device -> Host | Signal classification |

### 5.2 MQTT Commands

```json
// Set RX frequency
cmd/rx/frequency: 2400000000

// Set gain
cmd/rx/gain: 50

// Set TX frequency
cmd/tx/frequency: 2450000000
```

### 5.3 MQTT States

```json
// Current status
state/rx/frequency: 2400000000
state/rx/gain: 50
state/temperature: 45.2
```

---

## 6. WebSocket Interface

### 6.1 Connection

```javascript
// Connect to MQTT over WebSocket
const ws = new WebSocket('ws://192.168.1.10:9001/mqtt');
```

### 6.2 MQTT.js Configuration

```javascript
const client = mqtt.connect('ws://192.168.1.10:9001/mqtt', {
    clientId: 'dashboard-' + Math.random().toString(16).substr(2, 8)
});
```

---

## 7. Signal Classifier

### 7.1 PSD-Based Classification

The signal classifier analyzes spectrum shape:
- OFDM detection
- FSK detection
- Chirp detection
- CW/beacon identification

### 7.2 Classification Output

```json
classifier/result: {
    "type": "OFDM",
    "confidence": 0.92,
    "center_freq": 2450000000,
    "bandwidth": 20000000
}
```

---

## 8. Band Plan

### 8.1 Band Plan Data

```json
// /mnt/jffs2/bandplan.json
[
    {"start": 88000000, "end": 108000000, "name": "FM Broadcast", "color": "#FF0000"},
    {"start": 108000000, "end": 137000000, "name": "Airband", "color": "#00FF00"},
    {"start": 2400000000, "end": 2500000000, "name": "ISM Band", "color": "#0000FF"}
]
```

### 8.2 Display

The band plan is shown as a color-coded strip under the spectrum, marking known allocations.

---

## 9. VPN Configuration (WireGuard)

### 9.1 Configuration

```javascript
// Submit WireGuard config
const wgConfig = `[Interface]
PrivateKey = <key>
Address = 10.0.0.2/24
[Peer]
PublicKey = <key>
Endpoint = <host:port>
AllowedIPs = 0.0.0.0/0`;

// Send via MQTT
client.publish('cmd/vpn/config', wgConfig);
```

### 9.2 Status

```javascript
// Enable/disable VPN
client.publish('cmd/vpn/enable', 'true');
```

---

## 10. Network Configuration

### 10.1 Settings

| Setting | Default | Description |
|---------|---------|-------------|
| IP Address | 192.168.1.10 | Device IP |
| Netmask | 255.255.255.0 | Network mask |
| Gateway | 192.168.1.1 | Default gateway |
| DHCP | Disabled | Enable DHCP client |

### 10.2 MQTT Configuration

```json
cmd/network/config: {
    "ipaddr": "192.168.1.10",
    "netmask": "255.255.255.0",
    "gateway": "192.168.1.1",
    "dhcp": false
}
```

---

## 11. DATV Controller

### 11.1 Features

- DVB-S2 TX/RX
- Symbol rate selection
- MODCOD selection
- PTT control
- Stream monitoring

### 11.2 Topics

| Topic | Description |
|-------|-------------|
| cmd/datv/ptt | PTT on/off |
| cmd/datv/symbol_rate | Symbol rate |
| cmd/datv/modcod | MODCOD |
| dt/datv/status | Stream status |

---

## 12. Security

### 12.1 Authentication

The Dashboard has NO built-in authentication:
- Designed for local network use
- MQTT channel (port 9001) is unauthenticated

### 12.2 Recommendations

- Use VPN (WireGuard) for remote access
- Limit network exposure
- Change default passwords

### 12.3 SSH Access

- SSH on port 22
- Default: root/analog
- Can be changed via config.txt

---

## 13. Build Process

### 13.1 Dashboard Build

```bash
cd Dashboard
npm install
npm run build
```

### 13.2 Installation

The compiled Dashboard is installed to `/opt/maia` during Buildroot build.

---

## 14. Runtime Behavior

### 14.1 Startup

```bash
# Init script starts services
S90maia     # Start maia-httpd
S95mqtt     # Start mosquitto
S96plutostream  # Start DATV backend
```

### 14.2 Services

```
maia-httpd  # Web server on port 80
mosquitto   # MQTT broker on port 1883/9001
pluto_stream # DATV streaming
maia-wasm   # WebAssembly signal processor
```

---

## 15. Comparison with PlutoSDR Web

| Feature | LibreSDR (tezuka) | PlutoSDR |
|---------|-------------------|----------|
| Interface | React SPA | Static HTML |
| Backend | Rust HTTPd | BusyBox httpd |
| API | MQTT | CGI/forms |
| Signal Classifier | Yes | No |
| Band Plan | Yes | No |
| VPN Config | Yes | No |
| DATV Controller | Yes | No |

---

## 16. Related Documents

- `LIBRESDR_IIO_ARCHITECTURE.md` - IIO stack
- `LIBRESDR_SYSTEM_ARCHITECTURE.md` - System overview
- `LIBRESDR_BUILD_SYSTEM.md` - Build system
