# Arduino: Demo 2 Motor Control

Motor control for the two-wheel robot (rear ball caster). The Arduino searches by turning in place in short steps. When the Raspberry Pi sends a command, it turns to face the beacon, drives toward it, stops short of it, and (trial type 2) turns 90 degrees in the arrow direction. The robot runs untethered, so no serial input from a computer is needed during a trial.

## Files

| File | Demo 2 role | What it does |
|------|-------------|--------------|
| `demo2_main.ino` | Trial types 1 and 2 | Step-and-pause search rotation, serial command parsing, align turn, drive, and the 90 degree arrow turn, all driven by commands from the Raspberry Pi. |

## Trial type selection

`ENABLE_ARROW_TURN` is a constant near the top of the sketch:

| Value | Trial type | After driving |
|-------|------------|---------------|
| `false` | Type 1 | Stops |
| `true` | Type 2 | Turns 90 degrees left or right, then stops |

The same flag exists in the Pi script and **the two must match**. With `true`, the Arduino requires the arrow field, so a message without it gets `ERROR: ARROW FORMAT`. With `false`, it accepts either format but ignores the arrow, so the robot never turns. Re-upload the sketch after changing it.

## Behavior

1. **Start:** There is no go signal. On power-up or reset the Arduino prints `ARDUINO READY` and immediately begins its first search turn, so the robot is moving well inside the **30 second** limit.
2. **Search:** Turns in place by 30 degrees (`SEARCH_STEP_DEG`), pauses 3.5 seconds (`SEARCH_PAUSE_MS`) so the Pi can look for the beacon, and repeats. Search turns in the same direction as a left turn. A command is accepted during either the turn or the pause.
3. **Align:** When a valid command arrives, turns in place by the commanded angle. A positive angle turns left and a negative angle turns right.
4. **Approach:** Drives straight for the commanded distance minus `STOP_DISTANCE_IN` (8 in), never less than zero. The requirement is that the center of the robot ends within **1.5 ft** of the beacon without touching it. The 8 in margin is measured from the camera's distance reading, so it depends on where the camera sits relative to the midpoint between the wheels.
5. **Turn (type 2 only):** Turns **90 degrees** in the direction of the arrow letter in the command: `L` (green, left, counterclockwise) or `R` (red, right, clockwise). A wrong direction is a failure.
6. **Hold:** Enters `DONE` and commands zero wheel speed. Further commands get `ERROR: NOT SEARCHING`. Reset the Arduino to run again.

Mode sequence: `SEARCH_TURN`, `SEARCH_WAIT`, `ALIGN_TURN`, `DRIVE`, `ARROW_TURN` (type 2 only), `DONE`.

## Interface with the Raspberry Pi

- Link: USB serial, 115200 baud (the Pi sees it as `/dev/ttyACM0`).
- The robot is untethered during trials, so all commands come from the Pi, not a laptop.
- Message format, one line ending in a newline. Distance is in inches and angle is in degrees:

  | `ENABLE_ARROW_TURN` | Message | Example |
  |---------------------|---------|---------|
  | `false` | `distance,angle` (an arrow field is accepted and ignored) | `60.00,-12.50` |
  | `true` | `distance,angle,arrow` where arrow is `L` or `R` | `60.00,-12.50,L` |

- Accepted ranges: angle from -25 to 25 degrees, distance above 0 and up to 200 in. Commands are only accepted while searching (`SEARCH_TURN` or `SEARCH_WAIT`).
- Replies sent back to the Pi:

  | Reply | Meaning |
  |-------|---------|
  | `ARDUINO READY` | Booted or reset, search is starting |
  | `RECEIVED: <message>` | A full line arrived |
  | `COMMAND ACCEPTED` | Valid command, alignment starts |
  | `ERROR: DISTANCE FORMAT` or `ERROR: ANGLE FORMAT` | The numbers could not be parsed |
  | `ERROR: ARROW FORMAT` | The arrow is missing while `ENABLE_ARROW_TURN` is `true`, or the text after the angle is not `,L` or `,R` |
  | `ERROR: ANGLE RANGE` or `ERROR: DISTANCE RANGE` | Value outside the accepted range |
  | `ERROR: NOT SEARCHING` | A command arrived after the search ended (aligning, driving, turning, or done) |

## How it works

- 100 Hz control loop (10 ms sample time), carried over from Demo 1.
- Cascaded control per wheel: an outer PI position loop (`Kp_pos` = 5.0, `Ki_pos` = 0.5) produces a speed command capped at `MAX_SPEED` (3.0 radians per second), and an inner P velocity loop (`Kp` = 2.5) produces motor voltage. PWM is scaled against the 7.6 V battery voltage. The integrator resets inside the deadband and backs off when the speed command saturates.
- A move is finished when both wheels stay within 0.05 rad of their targets for 10 consecutive samples (100 ms), or after a 24 second timeout.
- Degrees are converted to wheel radians using the track width $T$ and the wheel radius $r$:

  $$
  \phi = \frac{T\,\theta}{2\,r}
  $$

  where $\theta$ is the body turn in radians and $\phi$ is the rotation of each wheel in radians. With $T = 15.0$ in and $r = 2.95$ in, a 90 degree turn is about 3.99 rad per wheel (an arc of 11.78 in at each wheel), and a 30 degree search step is about 1.33 rad.
- Forward travel is the drive distance divided by the wheel radius, applied to both wheels.

## Robot parameters

- Wheel diameter (`WHEEL_DIAMETER_IN`): 5.9 in
- Track width (`TRACK_WIDTH_IN`): 15.0 in
- Encoder: 800 counts per wheel revolution
- Battery voltage used for PWM scaling: 7.6 V
- Pins: enable 4, direction 7/8, PWM 9/10, encoder A 2/3, encoder B 5/6
- Motor 2 is physically reversed, handled in the direction logic and its encoder ISR.

## Tuning notes

- If distance is off by a constant percentage, adjust `WHEEL_DIAMETER_IN`. If the Pi's measured distance is what is off, recalibrate `distance_constant` in the Python script instead.
- If turns are off by a constant percentage, adjust `TRACK_WIDTH_IN`.
- If a turn goes the wrong way, flip `TURN_SIGN`.
- Gains: `Kp` (velocity), `Kp_pos` and `Ki_pos` (position), `MAX_SPEED`.
- Since trials are scored on time, raise `MAX_SPEED` only as far as the robot can still stop inside 1.5 ft without overshooting.
- `STOP_DISTANCE_IN` sets how far short of the beacon the robot stops. Increase it if the robot gets too close, decrease it if it stops outside 1.5 ft.
- Search: keep `SEARCH_STEP_DEG` smaller than the camera's horizontal field of view (about 52 degrees with the current calibration) so the marker cannot be skipped between pauses. `SEARCH_PAUSE_MS` must leave the Pi enough time to collect 5 stable frames.
- `MAX_ANGLE_DEG` is 25 degrees, close to the edge of the camera's view (about 26 degrees), so a marker at the very edge of the image is rejected with `ERROR: ANGLE RANGE`.

## Known issues / TODO

- The Arduino accepts one command per run. After `COMMAND ACCEPTED` it will not take another until it is reset, and the Pi does not resend after an `ERROR:` reply.
- The approach is open loop. The camera is not used to correct the heading or distance during the drive.
- Stopping position depends on the camera measurement, `WHEEL_DIAMETER_IN`, and `TRACK_WIDTH_IN` all being accurate. Retest after any change to the robot.
