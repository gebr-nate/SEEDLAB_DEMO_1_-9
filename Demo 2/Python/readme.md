
# Python: Beacon Detection and Arrow Direction

Raspberry Pi code for Demo 2: find the Aruco beacon, work out how far to turn and how far away it is, read the arrow direction, and command the Arduino.

## Files

| File | What it does |
|------|--------------|
| `[beacon_detect].py` | Reads the camera, detects marker 0 (6x6 dictionary), computes angle and distance, determines the arrow direction, and sends commands to the Arduino. |

> Replace the file name(s) with your actual script(s).

## Beacon

- Aruco marker ID 0, 6x6 dictionary (`DICT_6X6_50`)
- 2 inches (5.08 cm) wide and long when printed. This differs from the 5 cm marker in Demo 1, so update the marker size used for distance.
- Placed 4 to 6 feet from the robot, with a **green arrow on the left** or a **red arrow on the right**.

## How it works

1. Grabs frames from the camera and converts them to grayscale.
2. Detects markers with OpenCV's Aruco module and looks for marker ID 0.
3. Computes the horizontal angle to the marker center:
   `angle = atan((center_x_of_image - marker_center_x) / fx)`. Positive means the marker is left of the camera axis. `fx` comes from camera calibration.
4. Estimates distance from the marker's apparent size (marker size, `fx`, and pixel width) or from pose estimation.
5. Determines the arrow direction: [describe your method, e.g. check for green or red pixels to the left or right of the marker].
6. Sends the Arduino the mode (search, approach, turn), the angle, the distance, and the arrow direction over [I2C / serial].
7. Shows status on the LCD. LCD updates run in a separate thread so they do not slow the camera loop.

## Behavior by phase

| Phase | Pi behavior |
|-------|-------------|
| Search | No marker seen: tell the Arduino to rotate. |
| Approach | Marker seen: send angle and distance so the robot steers toward it and stops at 1.5 ft. |
| Turn (type 2) | Within 1.5 ft: send the 90 degree turn, left for green or right for red. |

## Hardware

- Raspberry Pi with a camera
- Adafruit 16x2 RGB character LCD (I2C)
- Connection to the Arduino: [fill in]

## Dependencies

- `opencv-contrib-python` (Aruco support)
- `adafruit-circuitpython-charlcd`
- `board` / `busio` (Adafruit Blinka)
- [Other libraries, e.g. `numpy`, `smbus2` or `pyserial`]

## Run

    python3 [beacon_detect].py

The robot must be untethered during trials, so configure the script to launch automatically at boot (e.g. systemd service or `crontab @reboot`) and do not rely on a keyboard or monitor.

## Known issues / TODO

- Arrow color detection can be sensitive to room lighting. Test at the demo location if possible.
- Marker size changed from 5 cm (Demo 1) to 2 in (Demo 2). Check the distance calculation.
- Angle and distance accuracy depend on the calibrated `fx` value.
- [Add anything else you find while testing.]
