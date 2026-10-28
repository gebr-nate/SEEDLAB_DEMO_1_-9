# SEEDLAB_DEMO_2_-9: EENG350 SEED Lab Team Repository

**Course:** EENG350, Systems Exploration, Engineering, and Design Laboratory
**School:** Colorado School of Mines, Fall 2026
**Team:** [Team 9]
**Members:** [Megan Brady], [Nathan Gebre-Hiwot], [Jake Toschik], [Bryson Lapham]

## Project Overview

This repository holds all code, models, and documentation for our SEED Lab robot project. The robot is built around an Arduino (motor control and encoder feedback) and a Raspberry Pi with a camera (Aruco marker detection). The Pi finds the beacon and sends commands to the Arduino, which drives the motors.

## Repository Structure

```
.
├── README.md
├── Demo 1/
│   ├── Arduino/
│   ├── Project_Board/
│   ├── Python/
│   └── Simulink/
└── Demo 2/
    ├── Arduino/
    ├── Project_Board/
    ├── Python/
    └── Simulink/
```

### Demo 1

Marker detection with angle reporting, straight-line motion, and rotate-then-drive. See the READMEs in `Demo 1/`.

### Demo 2

Demo 2 is due **November 13th** (100% of performance score). Completing on November 16th earns 90%, November 18th earns 70%, and after that the performance score is zero.

The robot must find a single beacon and respond to the arrow next to it. The beacon is an Aruco marker (marker ID 0, 6x6 dictionary, 2 inches wide and long when printed). A **green arrow on the left** means turn left, and a **red arrow on the right** means turn right. The beacon is placed about 5 feet from the robot (between 4 and 6 feet), and the robot starts with a random initial rotation.

There are two trial types:

1. **Find and approach:** Rotate to locate the beacon, drive to within 1.5 feet of it, and stop without hitting it. Robot position is the center of rotation (midpoint between the wheels). Judged on time and number of failures.
2. **Find, approach, and turn:** Drive to within 1.5 feet of the beacon, then turn 90 degrees in the direction the arrow points. Judged on time and number of failures.

Each team gets 4 trials (2 of each type, one with each beacon for type 2) and needs 2 successful runs of each type.

**A trial counts as a failure if:**
- The robot does not start moving within 30 seconds of the go signal.
- The robot moves but does not stop within 1.5 feet (type 1).
- The robot hits the beacon, does not come within 1.5 feet before turning, or turns the wrong direction (type 2).

The robot must be untethered during the test, with no communication of any kind (no keyboard, USB cable, etc.).

| Folder | Contents |
|--------|----------|
| `Demo 2/Arduino/` | Arduino sketch(es), with detailed comments, for motor control: rotate-to-search, drive toward the beacon, stop, and 90 degree turn. |
| `Demo 2/Project_Board/` | Record of the project board: which tasks were assigned and who was responsible for each. |
| `Demo 2/Python/` | Python code, with detailed comments, for finding the beacon, estimating its angle and distance, identifying the arrow direction, and commanding the Arduino. |
| `Demo 2/Simulink/` | Simulink models used to model the system and tune the controllers, including the 90 degree turn case. |

## Performance Scoring

Scores are relative to a target metric B (the third-best result across all sections). Each category is worth up to 50 points, for a maximum of 150.

- **Time:** (B / S) x 50
- **Failures:** 50 - (S - B) x 10

The score is set on the first day a team successfully completes the demo, then multiplied by the percentage for the day it was completed.

## How to Run

### Arduino
1. Open the sketch from `Demo 2/Arduino/` in the Arduino IDE.
2. Upload to the Arduino.
3. Power the robot from the battery. After the start delay, unplug all cables. The robot must run untethered.

### Python (Raspberry Pi)
1. On the Raspberry Pi, install the dependencies (OpenCV with Aruco support, Adafruit LCD libraries).
2. Run the script from `Demo 2/Python/` (set it to start automatically on boot so no keyboard is needed).
3. The Pi detects the beacon and tells the Arduino how to move.

### Simulink
Open the `.slx` models in `Demo 2/Simulink/` with MATLAB/Simulink.

## Hardware

- Arduino with motor driver shield
- Two DC drive motors with quadrature encoders (800 counts per revolution)
- Raspberry Pi with camera
- LCD display
- Robot geometry: wheel diameter 5.9 in, wheelbase 12.5 in

## Project Management

Task assignments and responsibilities are tracked on the GitHub project board for this repository (see `Demo 2/Project_Board/`).

## Documentation

- Reflection logs are uploaded to Canvas individually.
- Weekly agendas and minutes are uploaded to the Canvas "Demo 2 Team Documentation" assignment after the demo is completed. Keep recording minutes weekly.
- Code, models, and the project board are all kept in this repository.
