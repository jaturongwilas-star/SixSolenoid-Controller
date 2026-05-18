[README.md](https://github.com/user-attachments/files/27940839/README.md)
# BYD EV · AC Valve Service Controller

> **ESP32-S3 · WiFi Direct · AC Valve System v2.0**  
> A browser-based service tool for controlling BYD EV HVAC 6-channel solenoid valves via an ESP32-S3 microcontroller over WiFi Direct.

---

## Overview

This single-file HTML web UI connects directly to an ESP32-S3 board (no internet or router required) and provides full control over 6 AC solenoid valves in a BYD EV cooling/HVAC system. It is designed for EV technicians and DIY service use cases such as coolant flushing, refrigerant line purging, oil recovery, and system priming.

The interface is entirely self-contained — no build tools, no dependencies to install. Open in any modern browser and connect.

---

## Features

### Three Control Modes

#### ⚙ Auto Sequence
Sequential, timed activation of all 6 valves one at a time. Configure a single ON duration and let the controller cycle through every valve automatically. Real-time valve status table and system event log included.

#### ◈ Manual Control
Individual per-valve configuration with enable/disable toggles, custom ON duration, label editing, and live current/voltage display. Supports:
- Activate selected valves simultaneously
- Run a configurable cycle sequence (repeats, interval, delay)
- Live metrics table for all 6 channels (voltage, current, power)

#### 〜 Smart Flow Sequence
Five specialized flushing/service modes for EV cooling system maintenance:

| Mode | Description |
|------|-------------|
| **Pulse Flush** | Open/close pressure shock cycles per valve. Configurable ON/OFF duration, pulse count, and sequence repeats. |
| **Cross Flow** | Open multiple valves simultaneously to create a real flow path through the coolant loop. |
| **Reverse Pulse** | Bi-directional flow — alternates Forward (Cabin side) and Reverse (Battery side) to dislodge oil and debris from microchannels. |
| **Vacuum Assist** | Zone-by-zone sequential opening (Cabin Loop → Battery Loop → Chiller) to prevent trapped gas during vacuum fill. |
| **Oil Recovery** | Pulse-based oil return on selectable paths (Cabin, Battery, Compressor, Condenser, etc.). |

### System Status & Safety
- Real-time header status pills: WiFi connection, CAN BUS, Relay, and supply voltage
- Live clock display
- Per-mode progress bars and runtime counters
- Timestamped event log in every mode
- **Emergency Stop** button accessible from all tabs — immediately halts all valves

---

## Hardware Requirements

| Component | Details |
|-----------|---------|
| MCU | ESP32-S3 |
| Connectivity | WiFi Direct (Access Point mode) |
| Valves | 6× solenoid valves (AC/HVAC type) |
| Power | 12V supply (nominal 12.8V) |
| Relay board | Compatible relay module (online by default) |

Default ESP32 IP: `192.168.4.1`

---

## Getting Started

### 1. Flash the ESP32-S3
Flash your ESP32-S3 with firmware that exposes an HTTP API at the endpoints below. The board should operate in **WiFi AP (Access Point) mode**.

### 2. Connect Your Device
Connect your laptop or phone to the ESP32's WiFi network (SSID configured in firmware).

### 3. Open the Web UI
Open `byd_ac_valve_controller_webui.html` in any modern browser — no server needed.

### 4. Connect to ESP32
Enter the ESP32 IP address (default `192.168.4.1`) in any tab and click **Connect to ESP32**.

### 5. Control Valves
Select your desired mode (Auto / Manual / Smart Flow) and start the sequence.

---

## ESP32 HTTP API Reference

The UI communicates via simple HTTP GET requests. Implement these endpoints in your ESP32 firmware:

```
GET /valve?id=<0-5>&state=<0|1>   — Open or close a single valve
GET /valves?mask=<bitmask>         — Open multiple valves by bitmask (6-bit, LSB = V1)
GET /start                         — Start auto sequence
GET /stop                          — Stop all valves / current sequence
GET /emergency                     — Emergency stop (immediate)
GET /status                        — (Optional) Return JSON system status
```

### Bitmask Example
To open V1, V3, V5 simultaneously:
```
V1=bit0, V2=bit1, V3=bit2, V4=bit3, V5=bit4, V6=bit5
mask = 0b010101 = 21
GET /valves?mask=21
```

---

## File Structure

```
byd_ac_valve_controller_webui.html   # Complete self-contained web application
README.md
```

Everything — HTML, CSS, and JavaScript — is bundled in a single file for maximum portability.

---

## Browser Compatibility

Any modern browser with `fetch()` API support:
- Chrome / Chromium 80+
- Firefox 75+
- Safari 13+
- Edge 80+

> **Note:** If your browser blocks HTTP requests from a local file (`file://`), serve the HTML via a simple local server:
> ```bash
> python3 -m http.server 8080
> ```

---

## Valve Channel Map

| Channel | Default Label | Typical Function |
|---------|--------------|-----------------|
| V1 | Valve 1 | Cabin Loop (supply) |
| V2 | Valve 2 | Cabin Loop (return) |
| V3 | Valve 3 | Battery Loop (supply) |
| V4 | Valve 4 | Battery Loop (return) |
| V5 | Valve 5 | Chiller / Compressor |
| V6 | Valve 6 | Condenser / Aux |

Valve labels are editable in Manual Control mode.

---

## Smart Flow Mode Details

### Pulse Flush Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| Open Duration | 2 s | Time valve stays open per pulse |
| Close Duration | 1 s | Pause between pulses |
| Pulse Count | 5 | Pulses per valve |
| Valve Interval | 1 s | Delay before moving to next valve |
| Sequence Repeats | 1 | Full-cycle repetitions |

### Vacuum Assist Zones

| Phase | Zone | Valves |
|-------|------|--------|
| Phase A | Cabin Loop | V1 + V2 |
| Phase B | Battery Loop | V3 + V4 |
| Phase C | Chiller | V5 + V6 |

### Oil Recovery Paths (selectable)
- Cabin Loop (V1 + V2)
- Battery Thermal (V3 + V4)
- Compressor Line (V1 + V5)
- Condenser Path (V2 + V6)
- Full System (V1 → V6)

---

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

---

## License

MIT License — free to use, modify, and distribute for personal and commercial purposes.

---

*Built for BYD EV service technicians. Tested on BYD Atto 3 / Seal / Dolphin HVAC systems.*
