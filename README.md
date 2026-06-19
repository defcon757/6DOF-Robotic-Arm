# 6DOF Wi-Fi Controlled Robotic Arm with Camera Feedback
Designed and built a 6DOF Wi-Fi controlled robotic arm with live camera feedback using ESP32, ESP32-CAM, Python, Flask, and a PCA9685 driver. Features real-time remote control, video streaming, 3D-printed components, and a custom GUI for robotic manipulation and inspection.

## Overview

This project is a **6DOF Wi-Fi controlled robotic arm** operated through a web-based interface built with **Python Flask**, an **ESP32** controller, and an **ESP32-CAM** for live video feedback. The system allows users to remotely control the arm over a local Wi-Fi network while viewing a live camera stream from the robot's perspective.

## Features

* Web-based control interface
* Live camera feedback using ESP32-CAM
* Wireless communication over Wi-Fi
* 6 degrees of freedom (6DOF) robotic arm
* Dual synchronized shoulder motors
* PCA9685 PWM servo driver support
* Home position reset
* Emergency stop function
* Smooth servo motion control

## Hardware

* ESP32 Development Board
* ESP32-CAM with antenna
* PCA9685 16-Channel Servo Driver
* 2 × REV Continuous Rotation Motors (Shoulder)
* 1 × REV Continuous Rotation Motor (Base)
* 2 × MG996R Servos (Elbow, Wrist Up/Down)
* 2 × Micro Servos (Wrist Rotation, Claw)
* 5V 20A Power Supply
* 3D Printed 6DOF Robotic Arm

## Software

* Python 3
* Flask
* HTML/CSS/JavaScript
* Arduino IDE
* ESP32 Board Package
* ArduinoJson
* Adafruit PWM Servo Driver Library

## System Architecture

```text
Web GUI
   ↓
JavaScript
   ↓
Flask Server
   ↓
JSON Commands
   ↓
ESP32
   ↓
PCA9685
   ↓
Servos & Motors
   ↓
Robotic Arm Motion
```

Camera Feedback:

```text
ESP32-CAM
   ↓
Wi-Fi Stream
   ↓
Web Interface
   ↓
User
```

## Setup

### Python Server

1. Clone the repository.
2. Install dependencies:

```bash
pip install flask requests
```

3. Set the ESP32 IP address in `config.py`.

4. Run:

```bash
python app.py
```

5. Open:

```text
http://localhost:5000
```

### ESP32

1. Install required Arduino libraries:

   * ArduinoJson
   * Adafruit PWM Servo Driver

2. Configure Wi-Fi credentials.

3. Upload the firmware using Arduino IDE.

## Controls

| Joint          | Control Type           |
| -------------- | ---------------------- |
| Base Rotation  | Continuous Motor       |
| Shoulder       | Dual Continuous Motors |
| Elbow          | Servo                  |
| Wrist Up/Down  | Servo                  |
| Wrist Rotation | Servo                  |
| Claw           | Servo                  |

Additional functions:

* 🏠 Home Position
* 🛑 Emergency Stop

## Future Improvements

* Joystick control
* Object tracking using computer vision
* Autonomous pick-and-place operations
* Motion recording and playback
* Mobile application support

## Author

**Aidan Hernandez**

Developed as part of an engineering design project demonstrating the integration of mechanical design, embedded systems, networking, and software engineering into a remotely operated robotic platform.
