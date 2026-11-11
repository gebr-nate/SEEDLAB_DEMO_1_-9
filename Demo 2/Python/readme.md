# Python: Beacon Detection and Arrow Direction

Raspberry Pi code for Demo 2: find the Aruco beacon, work out how far to turn and how far away it is, read the arrow color, and send one command to the Arduino. The Arduino handles the search rotation on its own, so the Pi only sends a command once it has a stable detection.

## Files

| File | What it does |
|------|--------------|
| `beacon_detect.py` | Reads the camera, detects marker 0 (6x6 dictionary), computes the angle and distance, detects the arrow color, and sends one command to the Arduino over USB serial. Also drives the LCD status display. |

## Beacon

- Aruco marker ID 0, 6x6 dictionary (`DICT_6X6_50`)
- 2 inches (5.08 cm) wide and long when printed. This differs from the 5 cm marker in Demo 1. The 2 in size is built into the calibrated `distance_constant`, so recalibrate it if the marker size or the camera changes.
- Placed 4 to 6 feet from the robot, with a **green left arrow** or a **red right arrow**. The arrow is recognized by color only, not by shape.
- The Pi accepts a marker anywhere from 30 to 100 inches away (2.5 to about 8.3 ft), which covers the 4 to 6 ft trial range with margin.

## How it works

1. Grabs 640x480 frames from the camera (buffer size 1 so frames stay current) and converts them to grayscale.
2. Detects markers with OpenCV's Aruco module (`minMarkerPerimeterRate` is lowered to 0.01 so small, distant markers are still found) and looks for marker ID 0.
3. Computes the horizontal angle to the marker center:

   $$
   \theta = \arctan\left(\frac{320 - x_c}{f_x}\right)
   $$

   where $x_c$ is the marker center in pixels, 320 is the image center, and $f_x = 652.5$ px comes from camera calibration. Positive means the marker is left of the camera axis, so the robot should turn left.
4. Estimates distance from the marker's apparent size. No pose estimation is used:

   $$
   d = \frac{1251.5}{w}
   $$

   where $d$ is the distance in inches and $w$ is the length in pixels of the marker's top edge (corner 0 to corner 1). The constant 1251.5 has units of inch-pixels. For reference, 30 in appears about 42 px wide, 60 in about 21 px, and 100 in about 12.5 px.
5. Determines the arrow direction by color (Test 2 only):
   - Converts the frame to HSV and builds a green mask and a red mask.
   - Looks at a window around the marker: 2 marker widths to the left and right, and half a marker height above and below.
   - Counts mask pixels in that window. A color wins if it has at least `max(20, 3% of the marker's bounding box area)` pixels **and** more than twice as many pixels as the other color. Otherwise no arrow is reported and no command is sent.

   | Arrow | HSV range | Sent to Arduino |
   |-------|-----------|-----------------|
   | Green | (35, 80, 60) to (85, 255, 255) | `L` (turn left) |
   | Red | (0, 100, 70) to (10, 255, 255) and (170, 100, 70) to (179, 255, 255) | `R` (turn right) |

6. Waits for a stable reading before sending. A command goes out only after **5 frames in a row** where the distance is between 30 and 100 in, each frame is within 4 in and 4 degrees of the previous one, and (Test 2) an arrow color was found. Any frame that fails resets the count.
7. Sends **one** newline-terminated command to the Arduino and then stops sending (see the serial protocol below).
8. Shows status on the LCD: `Marker Detected` with `X Angle: <degrees>`, or `Searching...` when marker 0 is not visible. LCD updates run in a separate thread fed by a one-item queue, so they do not slow the camera loop.
9. Opens a preview window (`aruco detection`) showing the detected markers, a green line at the image center, and a red dot on the marker center. Press `q` in that window to quit.

## Serial protocol

- USB serial to the Arduino: `/dev/ttyACM0`, 115200 baud. Opening the port normally resets the Arduino, so the script waits 2 seconds after opening it.
- Command format (distance in inches, angle in degrees, positive angle means turn left):

  | Trial type | `ENABLE_ARROW_TURN` | Message | Example |
  |------------|---------------------|---------|---------|
  | Type 1 (approach and stop) | `False` | `distance,angle` | `60.00,-12.50` |
  | Type 2 (approach and turn) | `True` | `distance,angle,arrow` | `60.00,-12.50,L` |

- `ENABLE_ARROW_TURN` is set at the top of this script **and** in the Arduino sketch. The two must match. If the Arduino has it `true` and the Pi has it `false`, the Arduino replies `ERROR: ARROW FORMAT`. In the other mismatch the Arduino ignores the arrow and the robot never turns.
- The Pi prints every line the Arduino sends back, prefixed with `ARDUINO:`.
- After a command is sent the Pi does not send again. It re-arms only when the Arduino prints `ARDUINO READY`, meaning the Arduino restarted. It then clears its sent flag and stability counters.

## Behavior by phase

| Phase | Pi behavior | Arduino behavior |
|-------|-------------|------------------|
| Search | Marker 0 not seen or not yet stable: LCD shows `Searching...` and nothing is sent. | Turns 30 degrees, pauses 3.5 s, and repeats, all on its own. |
| Approach | Stable detection (5 frames, 30 to 100 in, and an arrow color for type 2): sends one command with distance, angle, and arrow. | Turns by the angle to face the marker, then drives straight for the measured distance minus 8 in. |
| Turn (type 2) | Nothing more to send. The arrow direction was part of the command. | After driving, turns 90 degrees: left for `L` (green), right for `R` (red). Then stops. |

Once the command is sent the Arduino runs the whole sequence without further input from the Pi.

## Hardware

- Raspberry Pi with a camera (opened as camera index 0, 640x480)
- Adafruit 16x2 RGB character LCD (I2C)
- USB cable to the Arduino (appears as `/dev/ttyACM0`)

## Dependencies

- `opencv-contrib-python` (Aruco support)
- `pyserial`
- `adafruit-circuitpython-charlcd`
- `adafruit-blinka` (provides `board` and `busio`)
- Python standard library: `math`, `time`, `threading`, `queue`

## Run

    python3 beacon_detect.py

The robot must be untethered during trials, so configure the script to launch automatically at boot (for example a desktop autostart entry, a systemd service, or `crontab @reboot`) and do not rely on a keyboard or monitor. The script opens a preview window, so it needs a display session when it starts. For a fully headless launch, remove the `cv2.imshow` and `cv2.waitKey` calls.

## Known issues / TODO

- Arrow color detection can be sensitive to room lighting because the HSV ranges are fixed. Test at the demo location if possible.
- Angle and distance accuracy depend on the calibrated `fx` and `distance_constant`. Both are tied to the 640x480 resolution and the 2 in marker, so recalibrate if either changes.
- The Pi only prints the Arduino's replies. If the Arduino answers with an `ERROR:` (for example an angle just beyond its 25 degree limit, which is near the edge of the camera's view), the Pi does not resend and the robot keeps searching until the Arduino is reset.
- Distance is measured from the camera, not from the center of the robot. The Arduino's 8 in stop margin is relative to that camera measurement.
- Color detection counts pixels in the whole window around the marker, so other red or green objects near the beacon can be mistaken for the arrow.
