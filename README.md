# ARM-6 — 6DOF Wi-Fi Controlled Robotic Arm

A fully remote-controlled, 6-degree-of-freedom robotic arm with live ESP32-CAM video feedback, controlled through a locally hosted Python/Flask web GUI over Wi-Fi.

Built by **Aidan Hernandez** — ENG GEN 101, Spring 2026.

---

## 📸 System Overview

```
Operator (Browser GUI)
        │
        ▼
Python Flask Server (Laptop)
   ├── Sends motion commands → ESP32 (ARM CONTROLLER)
   │                                   └── PCA9685 → 7 motors
   └── Streams video from  ← ESP32-CAM (CAMERA)
```

The system uses **two separate ESP32 boards**:
- `esp32_arm_controller` — receives JSON commands over HTTP, drives all 7 motors via a PCA9685 servo driver
- `esp32_cam_streamer` — streams MJPEG video from the OV2640 camera sensor over HTTP

---

## 🤖 Hardware

| Component | Qty | Notes |
|---|---|---|
| ESP32 Dev Board | 1 | Arm controller |
| ESP32-CAM (AI Thinker) | 1 | Video stream |
| PCA9685 16-Channel Servo Driver | 1 | I2C, drives all motors |
| REV Robotics HD Hex Motor (Continuous) | 3 | Base (×1), Shoulder (×2, mirrored) |
| MG996R Servo | 2 | Elbow, Wrist U/D |
| Micro Servo | 2 | Wrist Rotation, Claw |
| 5V 20A Power Supply | 1 | Powers motors + electronics |
| 3D Printed Arm Structure | — | See `/cad/stl/` |

### Motor Channel Map (PCA9685)

| Channel | Joint | Type |
|---|---|---|
| 0 | Base | Continuous motor |
| 1 | Left Shoulder | Continuous motor |
| 2 | Right Shoulder | Continuous motor (mirrored) |
| 3 | Elbow | Positional servo |
| 4 | Wrist Up/Down | Positional servo |
| 5 | Wrist Rotation | Positional servo |
| 6 | Claw | Positional servo |

### PWM Reference (50 Hz)

| Value | Meaning |
|---|---|
| `102` | Full reverse (continuous motors) |
| `307` | Stop / center (CONT_STOP) |
| `512` | Full forward (continuous motors) |
| `102–512` | Mapped from 0°–270° (positional servos) |

---

## 🗂 Repository Structure

```
arm6-robot/
├── firmware/
│   ├── esp32_arm_controller/
│   │   └── esp32_arm_controller.ino   # ESP32 arm command receiver
│   └── esp32_cam_streamer/
│       └── esp32_cam_streamer.ino     # ESP32-CAM MJPEG stream server
│
├── server/
│   ├── app.py                         # Flask routes
│   ├── arm_controller.py              # RobotArm class — state + HTTP send
│   ├── arm_sequence.py                # Scripted sequences (wave, scan, pickup…)
│   ├── config.py                      # IP addresses
│   ├── templates/
│   │   └── index.html                 # Jinja2 dashboard template
│   └── static/
│       ├── script.js                  # Hold-to-move button logic, status polling
│       └── style.css                  # Dark dashboard theme
│
├── cad/
│   └── stl/                           # 3D-printable arm parts (upload .stl files here)
│       ├── Arm_Parts.stl
│       ├── Base_Parts.stl
│       ├── Claw_Parts.stl
│
├── docs/
│   ├── wiring_diagram.md
│   ├── setup_guide.md
│   └── images/                        # Photos / diagrams (upload here)
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## ⚡ Quick Start

### 1. Configure IPs

Edit `server/config.py`:
```python
ARM_IP    = "192.168.x.xxx"   # IP of the ESP32 arm controller
CAMERA_IP = "192.168.x.xxx"   # IP of the ESP32-CAM
```

### 2. Flash the ESP32 boards

- Open `firmware/esp32_arm_controller/esp32_arm_controller.ino` in Arduino IDE
- Open `firmware/esp32_cam_streamer/esp32_cam_streamer.ino` in Arduino IDE
- Update `ssid` and `password` in **both** sketches
- Flash each to the correct board (see [Setup Guide](docs/setup_guide.md))

### 3. Install Python dependencies

```bash
pip install -r requirements.txt
```

### 4. Start the Flask server

```bash
cd server
python app.py
```

### 5. Open the GUI

Navigate to `http://localhost:5000` in any browser on the same network.

---

## 🎮 Controls

### Continuous Motors (Base, Shoulder)
Hold a direction button — each tick nudges the PWM pulse by ±8 counts. Releasing the button sends a stop command that resets the pulse to `307` (CONT_STOP).

### Positional Servos (Elbow, Wrist U/D, Wrist Rotation, Claw)
Hold a direction button — each tick moves the servo ±15°. Range is clamped to 0°–270°. A CENTER button snaps the joint to 135°.

### Preset Sequences (CLI — `arm_sequence.py`)
Run standalone from the command line, bypasses Flask and sends directly to the ESP32:

```bash
cd server
python arm_sequence.py
```

| Key | Sequence |
|---|---|
| `1` | Wave |
| `2` | Scan (base sweep) |
| `3` | Pickup simulation |
| `4` | Wrist display |
| `5` | Full showcase |
| `6` | Idle loop |
| `0` | Home all joints |

---

## 📡 API Reference

All endpoints are POST (except `/status`), expect and return JSON.

| Route | Method | Body | Description |
|---|---|---|---|
| `/servo` | POST | `{name, delta}` | Relative servo move |
| `/servo_set` | POST | `{name, angle}` | Absolute servo position |
| `/motor_nudge` | POST | `{name, delta}` | Nudge continuous motor pulse |
| `/motor_stop` | POST | `{name}` | Stop a motor (reset to 307) |
| `/home` | POST | — | Center all joints |
| `/stop` | POST | — | Emergency stop all motors |
| `/status` | GET | — | Returns current servo angles + motor pulses |

Valid `name` values: `base`, `shoulder`, `elbow`, `wrist_ud`, `wrist_rotate`, `claw`

---

## 🖨 CAD / 3D Printing

The arm was designed in **Onshape**. All printable parts are in `cad/stl/`. Recommended print settings:
- Material: PLA or PETG
- Layer height: 0.2 mm
- Infill: 30–40%
- Supports: Required for shoulder and elbow joints

> **Note:** STL files must be exported from Onshape and placed in `cad/stl/` manually — they are not tracked by Git due to file size. Placeholder files are included to document the expected parts.

---

## ⚠️ Known Limitations

- Requires a stable 2.4 GHz Wi-Fi network (ESP32 does not support 5 GHz)
- Continuous motors use open-loop control — no positional feedback
- Power supply must be external (5V, ≥10A recommended for full motion)
- Camera stream is low resolution (160×120) to reduce latency

---

## 📄 License

MIT License — free to use, modify, and build upon.
