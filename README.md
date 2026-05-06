# LARPII
Software Documentation for LARPII

This repository contains a complete ROS 2 autonomy pipeline for marker-based navigation, excavation, deposition, and return-home sequencing.

The system uses:
- Intel RealSense D435i camera for vision
- ArUco marker detection (OpenCV): https://chev.me/arucogen/ 4x4(50, 100, 250, 1000) was used
- RoboClaw motor controller for drive
- Basicmicro motor controller for excavation arm

---

## System Overview

The robot performs a fully autonomous cycle:

SEARCH → APPROACH → ALIGN → STOP → ARM ACTION → BACKUP → NEXT TARGET → REPEAT → HOME

### Marker IDs

| ID | Action |
|----|--------|
| 0  | Dig (Excavation) |
| 1  | Dump (Deposition) |
| 2  | Home (Stop mission) |

---

## How to Run Autonomous Cycles (Full Pipeline)

Open **4 terminals** and run the following:

---

## 1. Start RealSense Camera

```bash
ros2 launch realsense2_camera rs_launch.py \
  enable_color:=true \
  enable_depth:=true \
  enable_sync:=true \
  align_depth.enable:=true
```

---

## 2. Start ArUco Marker Detection

```bash
ros2 run aruco_opencv aruco_tracker_autostart --ros-args \
  -p cam_base_topic:=/camera/camera/color/image_raw \
  -p marker_dict:=4X4_50 \
  -p marker_size:=0.2 \
  -p image_is_rectified:=True \
  -p corner_refinement_method:=CORNER_REFINE_SUBPIX
```

---

## 3. Start the Excavator Node

```bash
python3 excavator_node.py
```

---

## 4. Start the Seeking Code

```bash
source ~/venvs/roboclaw/bin/activate
python3 spin_seek2.py
```

# LARP II Excavation Motor Control

This code provides a terminal-based motor control interface for the LARP II excavation assembly. It controls two Basicmicro-driven motors used for excavation and deposition.

---

## Motor Definitions

| Motor | Description |
|---|---|
| M1 | Shovel motor attached to the excavation bucket |
| M2 | Shoulder motor attached to the chassis/arm assembly |

---

## Purpose

This program allows the user to:

- Run a full excavation sequence
- Run a full deposition sequence
- Manually move the shovel motor
- Manually move the shoulder motor
- Move both motors together
- Reset encoder home positions
- Safely stop both motors before exiting

---

## Hardware Used

- Basicmicro motor controller
- Excavation shovel motor
- Shoulder/arm motor
- Encoder feedback for motor position control

---

## Before Running

Make sure the Basicmicro controller is connected to the computer.

Check the connected USB devices:

```bash
ls /dev/ttyACM*
```

The code is currently configured for:

```python
controller = Basicmicro("/dev/ttyACM0", 38400)
```

If the controller appears as a different device, update the port in the code.

Example:

```python
controller = Basicmicro("/dev/ttyACM1", 38400)
```

---

## Run the Program

From the folder containing the file, run:

```bash
python3 excavation_motor_control.py
```

Replace `excavation_motor_control.py` with the actual file name if different.

---

## Main Menu

After running the program, the following menu will appear:

```text
==============================================
LARP II EXCAVATION MOTOR CONTROL INTERFACE
==============================================

[MAIN MENU]
1. Full Excavation Sequence (excavate)
2. Full Deposition Sequence (deposit)
3. Manual Shoulder Move (M2 - Degrees)
4. Manual Shovel Move (M1 - Degrees)
5. Manual Dual Motor Move (M1 & M2 - Degrees)
6. Reset Encoder Home (Set 100k)
Q. Quit
```

---

## Option 1: Full Excavation Sequence

Select:

```text
1
```

This runs the automated excavation sequence using the `excavate()` function.

The shovel and shoulder motors move through a preset digging motion and then return to the home position.

---

## Option 2: Full Deposition Sequence

Select:

```text
2
```

This runs the automated deposition sequence using the `deposit()` function.

The shoulder motor lifts the arm, the shovel motor dumps material, and both motors return to the home position.

---

## Option 3: Manual Shoulder Move

Select:

```text
3
```

This controls Motor 2, the shoulder motor.

The program will ask for:

```text
Direction (CW / CCW):
Degrees of rotation:
Speed % (0-100):
```

Example input:

```text
Direction: cw
Degrees: 20
Speed: 10
```

This rotates the shoulder motor clockwise by 20 degrees at 10% speed.

---

## Option 4: Manual Shovel Move

Select:

```text
4
```

This controls Motor 1, the shovel motor.

The program will ask for:

```text
Direction (CW / CCW):
Degrees of rotation:
Speed % (0-100):
```

Example input:

```text
Direction: ccw
Degrees: 80
Speed: 10
```

This rotates the shovel motor counterclockwise by 80 degrees at 10% speed.

---

## Option 5: Manual Dual Motor Move

Select:

```text
5
```

This option moves both the shovel motor and shoulder motor.

The program will ask for:

```text
Direction of Shovel Motor (CW / CCW):
Degrees of rotation of Shovel Motor:
Speed of Shovel Motor % (0-100):
Direction of Arms Motor (CW / CCW):
Degrees of rotation of Arms Motor:
Speed of Arms Motor (0-100):
```

Use this option carefully because both motors may move at the same time.

---

## Option 6: Reset Encoder Home

Select:

```text
6
```

This resets both encoder positions to:

```text
100000
```

Use this when the excavation assembly is physically in its desired home/starting position.

---

## Quit the Program

Select:

```text
q
```

This stops both motors and closes the program.

---

## Encoder Logic

The code uses encoder counts to control motor position.

The home position is set to:

```text
100000 encoder counts
```

The code assumes:

```text
12500 encoder counts = 360 degrees
```

Target position is calculated using:

```text
target = current position ± (degrees / 360) × 12500
```

The motor runs until the encoder reaches the target position, then stops.

---

## Main Functions

| Function | Purpose |
|---|---|
| `excavate()` | Runs the main excavation sequence |
| `excavate2()` | Runs an alternate lab-box excavation sequence |
| `deposit()` | Runs the deposition/dumping sequence |
| `cw1()` | Rotates shovel motor clockwise |
| `ccw1()` | Rotates shovel motor counterclockwise |
| `cw2()` | Rotates shoulder motor clockwise |
| `ccw2()` | Rotates shoulder motor counterclockwise |
| `cw1return()` | Returns shovel motor to home |
| `ccw1return()` | Returns shovel motor to home |
| `cw2return()` | Returns shoulder motor to home |
| `ccw2return()` | Returns shoulder motor to home |
| `setreturn1()` | Sets Motor 1 encoder home |
| `setreturn2()` | Sets Motor 2 encoder home |
| `move_both()` | Moves both motors in one command |

---

## Recommended Startup Procedure

1. Place the excavation assembly in the home position.
2. Connect the Basicmicro motor controller.
3. Confirm the USB port using:

```bash
ls /dev/ttyACM*
```

4. Run the program:

```bash
python3 excavation_motor_control.py
```

5. Select option `6` to reset encoder home.
6. Test small manual movements using options `3` and `4`.
7. Run option `1` for excavation or option `2` for deposition.

---

## Safety Notes

- Keep hands and tools away from the excavation mechanism while powered.
- Start testing with low speeds between 5% and 15%.
- Always verify motor direction before running full sequences.
- Keep an emergency stop accessible.
- Press `Ctrl + C` if the program needs to be interrupted.

---
