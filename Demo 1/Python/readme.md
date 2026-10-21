
# Python: Aruco Marker Detection

Raspberry Pi code for Demo 1, Task 1: detect an Aruco marker and report its angle from the camera axis.

## File

| File | What it does |
|------|--------------|
| `aruco_detect.py` | Reads the camera, detects marker 0 (6x6 dictionary), computes the horizontal angle to it, and shows the result on the LCD. |

## How it works

1. Grabs 320x240 frames from the camera and converts them to grayscale.
2. Detects markers with OpenCV's Aruco module (`DICT_6X6_50`) and looks for marker ID 0.
3. Finds the marker's center pixel and computes the angle:
   `angle = atan((160 - center_x) / fx)`, with `fx = 326.25` from camera calibration.
4. Positive angle means the marker is to the left of the camera axis.
5. Prints the angle and displays "Marker Detected / X Angle: ..." on the 16x2 LCD. LCD updates run in a separate thread so they don't slow the camera loop.
6. A preview window shows the detected marker and quadrant lines. Press `q` to quit.

## Hardware

- Raspberry Pi with a USB camera
- Adafruit 16x2 RGB character LCD (I2C)

## Dependencies

- `opencv-contrib-python` (Aruco support)
- `adafruit-circuitpython-charlcd`
- `board` / `busio` (Adafruit Blinka)

## Run

    python3 aruco_detect.py

## Known issues / TODO

- Leftover Arduino serial code (`arduino.write`, `arduino.close()`) refers to variables that are never defined. Remove it or finish the serial setup, or the script will error when you quit.
- If a marker other than ID 0 is the only one visible, `angle_deg` is undefined and the LCD message line will fail. Move the LCD message inside the `if 0 in ids:` block.
- Angle accuracy depends on the calibrated `fx` value.
