# Simulink: Motor Position Control Model

Simulation of the wheel position controller used on the robot, compared against data logged from the Arduino. These are the models from earlier in the project; the Arduino sketches implement the same cascaded control.

## Files

| File | What it is |
|------|------------|
| `PI_control.slx` | Simulink model of the closed-loop position controller and motor. |
| `PI_control.mlx` | MATLAB live script that sets the parameters, runs the model, and plots simulation vs. experimental data. |

## Model structure

**Top level**
- A step input called "Desired Position" (final value 3.14159 rad, about half a wheel turn).
- A PID Controller block (parallel form, continuous time) acts on the position error and outputs a desired velocity.
- A velocity subsystem turns the desired velocity into motor voltage and wheel velocity.
- An integrator (`1/s`) turns wheel velocity into wheel position, which feeds back to the sum block.
- Band-limited white noise (covariance 0.001, 0.01 s sample time) is injected to mimic measurement noise.
- To Workspace blocks log `DesiredPosition`, `DesiredVelocity`, `Voltage`, and `Position`.

**Velocity subsystem**
- Proportional gain `Kp_vel` on the velocity error.
- Voltage saturation of +/-7.5 V (battery limit).
- Motor model from system ID: `K*sigma / (s + sigma)`.
- Noise added on the measured velocity.

This matches the Arduino controller: an outer PI position loop produces a speed command and an inner P velocity loop produces the motor voltage.

## Parameters (set in `PI_control.mlx`)

| Parameter | Value | Meaning |
|-----------|-------|---------|
| `K` | 0.364 rad/(V*s) | Motor DC gain (system ID) |
| `sigma` | 18 1/s | Motor time constant reciprocal (system ID) |
| `Kp_vel` | 11 | Inner velocity loop gain |
| `Kp` | 5 | Outer position loop gain |
| `Ki` | 1 | Outer position loop integral gain |

Simulation stop time is 3 s.

## How to run

1. Put `PI_control.slx`, `PI_control.mlx`, and `rawdata.txt` in the same folder.
2. Open `PI_control.mlx` in MATLAB and run it.
3. The script runs the model, loads `rawdata.txt`, and plots two figures:
   - Desired position: simulated vs. experimental
   - Closed-loop position: simulated vs. experimental (wheel 0)

`rawdata.txt` is the Arduino serial log. Column 1 is time, column 2 is desired position, and column 3 is actual position of wheel 0.

## Notes

- The script defines `s = tf('s')` because the transfer function block was not working directly in the model.
- The simulation is in radians of wheel rotation. The Arduino code converts feet and degrees to radians before using the same loops.
