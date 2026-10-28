
# Simulink: Motor Position Control Model

Simulation of the wheel position controller used on the robot, compared against data logged from the Arduino. The Arduino sketches implement the same cascaded control (outer PI position loop, inner P velocity loop).

## Files

| File | What it is |
|------|------------|
| `PI_control.slx` | Simulink model of the closed-loop position controller and motor. |
| `PI_control.mlx` | MATLAB live script that sets the parameters, runs the model, and plots simulation vs. experimental data. |

## Model structure

**Top level**
- A step input called "Desired Position". Final value is **3.328 rad**, which is one wheel's rotation for a **90 degree in-place turn** (wheelbase 12.5 in, wheel diameter 5.9 in). Change it to model a drive: distance in inches / 2.95 in = radians.
- A PID Controller block (parallel form, continuous time) acts on the position error and outputs a desired velocity.
- A velocity subsystem turns the desired velocity into motor voltage and wheel velocity.
- An integrator (`1/s`) turns wheel velocity into wheel position, which feeds back to the sum block.
- Band-limited white noise (covariance 0.001, **0.01 s sample time, matching the 100 Hz Arduino loop**) is injected to mimic measurement noise.
- To Workspace blocks log `DesiredPosition`, `DesiredVelocity`, `Voltage`, and `Position`.

**Velocity subsystem**
- Proportional gain `Kp_vel` on the velocity error.
- Voltage saturation of +/-7.5 V (battery limit).
- Motor model from system ID: `K*sigma / (s + sigma)`.
- Noise added on the measured velocity.

## Parameters (set in `PI_control.mlx`)

| Parameter | Value | Meaning | Matches Arduino |
|-----------|-------|---------|-----------------|
| `K` | 0.364 rad/(V*s) | Motor DC gain (system ID) | n/a |
| `sigma` | 18 1/s | Motor time constant reciprocal (system ID) | n/a |
| `Kp_vel` | 11 | Inner velocity loop gain | `Kp` |
| `Kp` | 5 | Outer position loop gain | `Kp_pos` |
| `Ki` | 1 | Outer position loop integral gain | `Ki_pos` |
| `Ts` | 0.01 s | Sample time | 100 Hz loop |
| `wheel_radius` | 2.95 in | Half of the 5.9 in diameter | `WHEEL_DIAMETER_IN` |
| `wheelbase` | 12.5 in | Used for turn conversion | `WHEELBASE_IN` |
| `MAX_SPEED` | [fill in] rad/s | Speed command limit | `MAX_SPEED` |

> **Check before the demo:** the gains above are carried over from Demo 1. Confirm `Kp_vel`, `Kp`, `Ki`, and `MAX_SPEED` against the constants in your Demo 2 sketch and update both if they differ. Add a rate limit or saturation block of `MAX_SPEED` on the desired velocity in the model.

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
- Use the model to check that a 90 degree turn and a roughly 3 to 5 ft approach settle without overshooting the 1.5 ft stop line.
