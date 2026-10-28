
# Arduino: Demo 2 Motor Control

Motor control for the two-wheel robot (rear ball caster). The Arduino executes motion commands: rotate in place to search, drive toward the beacon, stop within 1.5 ft, and (trial type 2) turn 90 degrees. The robot must run untethered, so no serial input from a computer is needed during a trial.

## Files

| File | Demo 2 role | What it does |
|------|-------------|--------------|
| `[demo2_main].ino` | Trial types 1 and 2 | [Search rotation, approach, stop, and 90 degree turn driven by commands from the Raspberry Pi.] |

> Replace the file names and descriptions with your actual sketches.

## Behavior

1. **Start:** Robot waits for the go signal (power-up delay or Pi command). It must begin moving within **30 seconds** or the trial is a failure.
2. **Search:** Rotates slowly in place until the Pi reports the beacon.
3. **Approach:** Drives toward the beacon using the angle and distance reported by the Pi. Stops when the center of the robot (midpoint between the wheels) is within **1.5 ft** of the beacon. Must never hit the beacon.
4. **Turn (type 2 only):** Turns **90 degrees** in the direction the arrow points (green/left = counterclockwise, red/right = clockwise). A wrong direction is a failure.
5. **Hold:** Stops and holds position.

## Interface with the Raspberry Pi

- Link: [I2C / serial: fill in]
- Message format: [fill in, e.g. mode, angle in degrees, distance in feet, arrow direction]
- The robot is untethered during trials, so all commands must come from the Pi, not a laptop.

## How it works

- 100 Hz control loop (10 ms sample time), carried over from Demo 1.
- Cascaded control per wheel: outer PI position loop produces a speed command (capped at `MAX_SPEED`), inner P velocity loop produces motor voltage.
- Feet and degrees are converted to wheel radians using the wheel radius and wheelbase.
- A 90 degree turn is about 3.33 rad of wheel rotation per wheel (arc of 9.82 in at the wheels).

## Robot parameters

- Wheel diameter: 5.9 in
- Wheelbase (center to center of wheel contact): 12.5 in
- Encoder: 800 counts per wheel revolution
- Pins: enable 4, direction 7/8, PWM 9/10, encoder A 2/3, encoder B 5/6
- Motor 2 is physically reversed, handled in the direction logic and its encoder ISR.

## Tuning notes

- If distance is off by a constant percentage, adjust `WHEEL_DIAMETER_IN`.
- If turns are off by a constant percentage, adjust `WHEELBASE_IN`.
- If a turn goes the wrong way, flip `ROTATE_SIGN`.
- Gains: `Kp` (velocity), `Kp_pos` and `Ki_pos` (position), `MAX_SPEED`.
- Since trials are scored on time, raise `MAX_SPEED` only as far as the robot can still stop inside 1.5 ft without overshooting.
- Search rotation should be slow enough that the camera does not blur or skip past the marker.
