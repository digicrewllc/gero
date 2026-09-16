# gero

**Gero** is the edge-device Node.js agent for the [Geroix](https://iot.geroix.com) IoT platform. It runs on embedded Linux boards (Raspberry Pi, Arduino Edison, Fedora-based SBCs) and connects them to the Geroix cloud over a persistent WebSocket, allowing remote monitoring and control of GPIO pins and application peripherals.

---

## Features

- **WebSocket communication** — maintains a reconnecting WebSocket connection to `wss://iot.geroix.com/gero.ashx` and handles keep-alive, configuration requests, and command dispatch.
- **GPIO management** — supports LEDs, buttons, PWM outputs, digital/analog inputs, and on/off switches across Raspberry Pi (raspi-io), Arduino Edison, and generic Linux targets.
- **Built-in applications** — ready-to-use peripherals including:
  - RGB colour picker (via BlueGiga BLE)
  - Thermometer (Dallas 1-Wire DS18B20)
  - Light meter
  - Shift register (74HC595) driver
  - Morse-code output on any LED/GPIO
  - EEG detector (local HTTP server)
  - System command execution
- **Bluetooth / BLE** — optional Bleno-based BLE service that advertises the device to the Geroix mobile app.
- **Auto-start** — ships with a systemd unit file (`gero.service`) and a SysV init script (`gero-init-pi`) for boot-time launch.

---

## Requirements

- Node.js (v6 or later recommended)
- Linux-based OS (Raspberry Pi OS, Raspbian, Fedora, or Intel Edison)
- npm packages: `websocket`, `sleep`, and optionally `bleno` (BLE) and the appropriate GPIO library for your platform

---

## Installation

```bash
# Clone the repository to the recommended path
git clone https://github.com/digicrewllc/gero.git /opt/geroix/gero
cd /opt/geroix/gero
npm install
```

### Configure

Edit `config.json` with your device settings:

```json
{
  "mainServerUrl": "wss://iot.geroix.com/gero.ashx",
  "gerokey": "<your-gero-key>",
  "localWebServerListeningPort": 8000,
  "configRequestRetryInMillis": 2000
}
```

| Key | Description |
|---|---|
| `mainServerUrl` | WebSocket URL of the Geroix cloud server |
| `gerokey` | Device API key from the Geroix portal |
| `localWebServerListeningPort` | Port for the local HTTP server (EEG detector, etc.) |
| `configRequestRetryInMillis` | Retry interval (ms) for configuration requests |

---

## Running

### Manual

```bash
node index.js
```

### Auto-start with systemd (Raspberry Pi / Fedora)

```bash
sudo cp gero.service /etc/systemd/system/
sudo systemctl enable gero
sudo systemctl start gero
```

### Auto-start with SysV init (older Raspbian)

```bash
sudo cp gero-init-pi /etc/init.d/gero
sudo chmod +x /etc/init.d/gero
sudo update-rc.d gero defaults
sudo /etc/init.d/gero start
```

---

## Project Structure

| File | Purpose |
|---|---|
| `index.js` | Entry point — loads config, detects OS/hardware, initialises WebSocket and BLE |
| `gero.js` | Core device object — GPIO registry, app runner, state reporting |
| `gpio.js` | GPIO abstraction — wraps vendor-specific drivers |
| `gpio-raspbian.js` | Raspberry Pi GPIO driver |
| `gpio-arduino.js` | Arduino Edison GPIO driver |
| `gpio-linux.js` | Generic Linux GPIO driver |
| `wscomm.js` | WebSocket connection manager (reconnect, keep-alive, command routing) |
| `httpcomm.js` | Local HTTP server for sensor data ingestion |
| `bleno-gero.js` | BLE advertisement / GATT service via Bleno |
| `bluegiga.js` | BlueGiga BLE serial driver (RGB light control) |
| `iot.js` | IoT message envelope helpers |
| `constants.js` | Command codes and application IDs |
| `skky.js` | General-purpose utilities |
| `config.json` | Safe-mode configuration (read on startup) |
| `config.all.json` | Full runtime configuration (written by the agent) |
| `gero.service` | systemd unit file |
| `gero-init-pi` | SysV init script |
| `start.sh` | Startup helper (sets `NODE_PATH`, delays 30 s, runs `node index.js`) |

---

## License

See [LICENSE](LICENSE).
