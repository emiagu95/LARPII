# LARPII
Software Documentation for LARPII

This repository contains a complete ROS 2 autonomy pipeline for marker-based navigation, excavation, deposition, and return-home sequencing.

The system uses:
- Intel RealSense D435i camera for vision
- ArUco marker detection (OpenCV)
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

## How to Run (Full Pipeline)

Open **4 terminals** and run the following:

---

## 1. Start RealSense Camera

ros2 launch realsense2_camera rs_launch.py \
  enable_color:=true \
  enable_depth:=true \
  enable_sync:=true \
  align_depth.enable:=true

## 2. Start RealSense Camera

ros2 run aruco_opencv aruco_tracker_autostart --ros-args \
  -p cam_base_topic:=/camera/camera/color/image_raw \
  -p marker_dict:=4X4_50 \
  -p marker_size:=0.2 \
  -p image_is_rectified:=True \
  -p corner_refinement_method:=CORNER_REFINE_SUBPIX

## 3. Start the excavator node
python3 excavator_node.py

## 4. Start the seeking code

source ~/venvs/roboclaw/bin/activate
python3 spin_seek2.py
