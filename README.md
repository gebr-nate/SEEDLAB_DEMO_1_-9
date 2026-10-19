# SEEDLAB_DEMO_1_-9: EENG350 SEED Lab Team Repository

**Course:** EENG350, Systems Exploration, Engineering, and Design Laboratory
**School:** Colorado School of Mines, Fall 2026
**Team:** [Team name/number]
**Members:** [Member 1], [Member 2], [Member 3], [Member 4]

## Project Overview

This repository holds all code, models, and documentation for our SEED Lab robot project. The robot is built around an Arduino (motor control and encoder feedback) and a Raspberry Pi with a camera (Aruco marker detection).

## Repository Structure

```
.
├── README.md
└── Demo 1/
    ├── Arduino/
    ├── Project_Board/
    ├── Python/
    └── Simulink/
```

### Demo 1

Demo 1 requires the robot to complete three critical tasks:

1. **Marker detection:** Detect an Aruco marker (marker 0, 6x6 dictionary, 5 cm x 5 cm, 1 to 5 feet away) and report the signed angle in degrees between the camera axis and the marker on the LCD. The angle is positive when the marker is to the left of the camera axis.
2. **Straight-line motion:** Drive forward in a straight line and stop after a specified distance (1 to 10 feet).
3. **Rotate, then drive:** Rotate by a specified angle, then drive forward a specified distance.

| Folder | Contents |
|--------|----------|
| `Demo 1/Arduino/` | Arduino sketches with detailed comments. Includes the motor control sketches for the straight-line move (task 2) and the rotate-then-drive move (task 3). Each uses a cascaded controller: an outer PI position loop and an inner P velocity loop. |
| `Demo 1/Project_Board/` | Record of the project board: which tasks were assigned and who was responsible for each. |
| `Demo 1/Python/` | Python code, with detailed comments, for Aruco marker detection and angle calculation, and for displaying the result on the LCD. |
| `Demo 1/Simulink/` | Simulink models used to model and simulate the system and to design and tune the controllers. |

## How to Run

### Arduino (tasks 2 and 3)
1. Open the sketch from `Demo 1/Arduino/` in the Arduino IDE.
2. Set the target values at the top of the sketch (`DESIRED_ANGLE_RAD` in radians and `DESIRED_DISTANCE_FT` in feet for task 3).
3. Upload to the Arduino. The robot starts on its own after a short delay.

### Python (task 1)
1. On the Raspberry Pi, install the dependencies (e.g. OpenCV with Aruco support).
2. Run the script from `Demo 1/Python/`, for example: `python3 <script_name>.py`
3. The detected marker angle is shown on the LCD.

### Simulink
Open the `.slx` models in `Demo 1/Simulink/` with MATLAB/Simulink.

## Hardware

- Arduino with motor driver shield
- Two DC drive motors with quadrature encoders (800 counts per revolution)
- Raspberry Pi with camera
- LCD display
- Robot geometry: wheel diameter 5.9 in; wheelbase [measure and fill in] in

## Project Management

Task assignments and responsibilities are tracked on the GitHub project board for this repository (see the **Projects** tab: [link to project board]). A record of the board is also kept in `Demo 1/Project_Board/`.

## Documentation

Reflection logs, weekly agendas and minutes, and the design presentation are submitted through Canvas.
