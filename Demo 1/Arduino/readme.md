# Arduino: Demo 1 Motor Control

Open-loop-start, closed-loop-position drive code for the two-wheel robot (rear ball caster). Each sketch runs automatically after power-up and needs no serial input.

## Files

| File | Task | What it does |
|------|------|--------------|
| `move_straight.ino` | Demo 1, Task 2 | Drives both wheels forward a set distance in feet, then holds position. |
| `rotate_then_drive.ino` | Demo 1, Task 3 | Rotates in place by a set angle in degrees, then drives forward a set distance in feet. |

## How to use

1. Edit the constants at the top of the sketch:
   - `move_straight.ino`: `DISTANCE_FT`
   - `rotate_then_drive.ino`: `DESIRED_ANGLE_DEG` (+ = counterclockwise/left) and `DESIRED_DISTANCE_FT`
2. Upload to the Arduino.
3. The motors stay off for 3 seconds (`START_DELAY_MS`) so you can unplug the cable and set the robot down, then the move starts.
4. Press reset or cycle power to run again.

## How it works

- 100 Hz control loop (10 ms sample time).
- Cascaded control per wheel: outer PI position loop produces a speed command (capped at `MAX_SPEED`), inner P velocity loop produces motor voltage.
- Targets are converted from feet or degrees to wheel radians using the wheel radius (and wheelbase for turns).
- `rotate_then_drive.ino` runs in phases: rotate, wait until both wheels settle, reset the encoders, then drive straight.
- `move_straight.ino` includes a position deadband that cuts the motors once the wheels are close to the target, to stop end-of-move twitching.

## Robot parameters

- Wheel diameter: 5.9 in (two matching drive wheels)
- Wheelbase (center to center of wheel contact): 12.5 in, used by `rotate_then_drive.ino`
- Encoder: 800 counts per wheel revolution
- Pins: enable 4, direction 7/8, PWM 9/10, encoder A 2/3, encoder B 5/6
- Motor 2 is physically reversed, which is handled in the direction logic and its encoder ISR.

## Tuning notes

- If distance is off by a constant percentage, adjust `WHEEL_DIAMETER_IN`.
- If turns are off by a constant percentage, adjust `WHEELBASE_IN`.
- If a positive angle turns the wrong way, flip `ROTATE_SIGN`.
- Gains: `Kp` (velocity), `Kp_pos` and `Ki_pos` (position), `MAX_SPEED`.
- Serial output at 115200 baud shows heading or distance traveled while the cable is plugged in.
