# DexVisio

**Computer Vision-Based Mimetic Arm**

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-blue.svg)](https://www.python.org/)
[![Arduino](https://img.shields.io/badge/Arduino-green.svg)](https://www.arduino.cc/)
[![OpenCV](https://img.shields.io/badge/OpenCV-red.svg)](https://opencv.org/)
[![MediaPipe](https://img.shields.io/badge/MediaPipe-purple.svg)](https://mediapipe.dev/)

DexVisio is an experimental computer-vision system that mimics human hand gestures using a five-finger robotic hand. It combines real-time hand landmark detection with Arduino-controlled servo motors, providing a compact platform for exploring vision-guided robotic hand control.

---

## Abstract

DexVisio captures hand movements using a webcam and MediaPipe hand landmark detection. Finger joint angles are estimated from the detected landmarks and transmitted to an Arduino over serial communication. The Arduino maps the received values to five servo motors controlling the thumb, index finger, middle finger, ring finger, and pinky.

The current implementation demonstrates direct camera-to-actuator control. It is an experimental prototype for exploring hand tracking, gesture-based control, and low-cost robotic manipulation.

## System Overview

The system operates as a real-time data pipeline:

```text
Webcam
   |
   v
OpenCV Frame Capture
   |
   v
MediaPipe Hand Landmark Detection
   |
   v
Finger Angle Estimation
   |
   v
Serial Communication
   |
   v
Arduino Servo Controller
   |
   v
Five-Finger Robotic Hand
```

The main processing stages are:

1. OpenCV captures frames from the computer's webcam.
2. MediaPipe identifies hand landmarks and handedness.
3. DexVisio estimates finger angles from the landmark positions.
4. The angle values are formatted as comma-separated integers.
5. An Arduino receives the values through serial communication.
6. Five servo motors move according to the transmitted values.

## Experimental Objectives

The project investigates:

- Real-time hand landmark tracking.
- Estimation of finger joint angles from two-dimensional imagery.
- Serial communication between a vision application and a microcontroller.
- Mapping vision-derived angles to servo actuator positions.
- Construction of a low-cost mimetic robotic hand prototype.

## Hardware Requirements

| Component | Function |
|---|---|
| Computer with an integrated or external webcam | Captures hand movement. |
| Arduino-compatible board | Receives angle data and controls the servo motors. |
| Five servo motors | Actuate the thumb and four fingers. |
| USB serial connection | Transfers data between the computer and Arduino. |
| Robotic hand structure | Provides mechanical mounting for the actuators. |

### Servo Connections

| Finger | Arduino pin |
|---|---:|
| Thumb | 9 |
| Index finger | 10 |
| Middle finger | 11 |
| Ring finger | 12 |
| Pinky | 13 |

> **Hardware note:** The exact mechanical assembly, servo mounting arrangement, and detailed build instructions are not included in the current repository documentation.

## Software Requirements

- Python 3.8 or newer
- OpenCV
- MediaPipe
- PySerial
- Arduino IDE
- Arduino `Servo.h` library

### Python Dependencies

Install the Python packages with:

```bash
pip install opencv-python mediapipe pyserial
```

## Installation and Setup

### 1. Clone the repository

```bash
git clone https://github.com/Subhrajay/DexVisio.git
cd DexVisio
```

### 2. Upload the Arduino sketch

Open `MimeticArmNEWV2.ino` in the Arduino IDE, select the appropriate board and port, and upload the sketch to the Arduino board.

### 3. Configure the serial port

Before running the Python application, update the serial device path in `MimeticArmNEWV2.py` to match the port used by your Arduino.

The existing configuration example is:

```python
arduino = serial.Serial('/dev/tty.usbmodem11101', 9600)
```

Common serial device paths include:

| Operating system | Example serial path |
|---|---|
| macOS | `/dev/tty.usbmodem*` |
| Linux | `/dev/ttyUSB0` |
| Windows | `COM3` |

The examples above are indicative. Use the actual device path assigned to your Arduino by your operating system.

## Running DexVisio

Start the tracking application from the repository directory:

```bash
python MimeticArmNEWV2.py
```

Position one hand within the webcam frame. The application is designed to:

- Detect the hand and its landmarks.
- Calculate five finger angles.
- Draw the detected landmarks and hand region.
- Send the angle values to the Arduino.
- Display the processed video feed.

Press **`q`** to terminate the application.

## Serial Data Format

The Python application sends five integer values separated by commas. The values represent the thumb, index finger, middle finger, ring finger, and pinky, in that order.

```text
thumb,index,middle,ring,pinky
```

Example:

```text
45,80,75,70,65
```

The Arduino reads the incoming data, separates the values at the commas, converts them to integers, and writes them to the corresponding servos.

## Repository Structure

| File | Description |
|---|---|
| `HandMovementModule.py` | MediaPipe-based hand detection, landmark extraction, finger-state detection, and distance calculation. |
| `MimeticArmNEWV2.py` | Webcam processing, angle calculation, and serial transmission. |
| `MimeticArmNEWV2.ino` | Arduino serial parsing and servo control. |
| `LICENSE` | GNU General Public License v3.0. |
| `README.md` | Project documentation. |

## Implementation Details

### Hand Landmark Detection

`HandMovementModule.py` uses MediaPipe Hands to:

- Detect up to two hands.
- Extract 21 landmarks per hand.
- Calculate the hand bounding box and center.
- Identify the left or right hand.
- Determine whether individual fingers are raised.
- Draw hand landmarks and connections.
- Calculate distances between selected landmarks.

The implementation can flip handedness classification to compensate for a mirrored webcam view.

Although the hand-detection module can detect up to two hands, the current application selects the first detected hand for control.

### Finger Angle Estimation

`MimeticArmNEWV2.py` estimates an angle using three landmark points associated with each finger. The calculation uses the difference between two `atan2` directions:

```python
angle = abs(
    atan2(y3 - y2, x3 - x2)
    - atan2(y1 - y2, x1 - x2)
)
```

The result is converted from radians to degrees and transmitted to the Arduino.

> **Implementation note:** This is the angle-calculation fragment documented for the project, not a complete standalone Python program. Landmark selection, imports, serial setup, frame processing, and the surrounding application logic remain in the project source files. The current implementation does not apply a complete calibration or constraint model before transmission.

### Servo Actuation

The Arduino sketch initializes five servo motors on pins 9 through 13. During operation, it:

1. Reads incoming serial data.
2. Separates the values at commas.
3. Converts the values to integers.
4. Writes each value to the corresponding servo.
5. Mirrors the angle for four finger servos.

The thumb is controlled directly. The index, middle, ring, and pinky servos receive the inverted value calculated as `180 - angle`.

## Calibration and Limitations

DexVisio is a functional prototype, not a fully calibrated robotic control system. The following limitations should be considered when adapting it to a particular robotic hand:

- Finger angles are estimated from a two-dimensional video projection.
- The mapping between detected landmarks and servo positions is not mechanically calibrated.
- Servo accuracy depends on actuator quality and mechanical assembly.
- Lighting, camera resolution, and camera viewpoint affect landmark detection.
- Finger occlusion may produce incorrect angle estimates.
- The current application selects the first detected hand.
- The serial device path is hard-coded for a specific environment.
- Servo values are not fully constrained at the application level.
- Hardware assembly details, build instructions, and a calibration procedure are not included in the repository.

Servo limits and angle mapping should be calibrated for the actual mechanical hand before regular operation.

## Experimental Validation

The repository does not include quantitative benchmarks, validation datasets, calibration procedures, or measured servo accuracy. Therefore, performance claims beyond the implemented hand tracking and actuator communication cannot be verified from the current project materials.

## Troubleshooting

### The serial port cannot be opened

- Confirm that the Arduino is connected to the computer.
- Select the correct serial device or COM port.
- Close other applications that may already be using the serial connection.
- Update the device path in `MimeticArmNEWV2.py`.

### Hand landmarks are not detected

- Improve the ambient lighting.
- Keep the hand inside the camera frame.
- Verify that the webcam is accessible to OpenCV.
- Move the hand closer to or farther from the camera.

### Servo movement is incorrect

- Verify the servo wiring and Arduino pin assignments.
- Supply sufficient power to the servos.
- Check for mechanical obstruction.
- Calibrate the servo limits for the particular robotic assembly.
- Review the angle inversion used in `MimeticArmNEWV2.ino`.

## Safety Considerations

Before operating a mechanical hand assembly:

- Secure the servos and structural components.
- Disconnect power before adjusting wiring or mechanical linkages.
- Keep the robotic fingers from contacting the user or the camera.
- Test servo movements at reduced speed and within limited angles.
- Use an external regulated servo power supply when required by the servos and assembly.

## Contributing

Contributions may address:

- Servo calibration.
- Improved angle mapping.
- Multi-hand control.
- Hardware assembly documentation.
- Camera and serial configuration.
- Runtime error handling.
- Quantitative system evaluation.
- Unit and integration testing.

Contributors should submit reproducible changes and clearly document hardware requirements or behavioral assumptions.

## License

DexVisio is distributed under the **GNU General Public License v3.0**. See the [`LICENSE`](LICENSE) file for the full license text.
