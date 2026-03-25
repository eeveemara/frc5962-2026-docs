# Projectile Physics & Hardware Tools

The fire control pipeline needs to know what RPM and time-of-flight to use for any given distance. We could have just measured a few distances on the real robot and interpolated between them. Instead, we wrote a physics simulator that generates the whole table from CAD measurements, so we had usable shot parameters before the robot ever fired a ball.

## ProjectileSimulator

`ProjectileSimulator` traces a ball's trajectory from the moment it leaves the shooter to the moment it reaches hub height. The integration uses 4th-order Runge-Kutta (RK4), which is more accurate than simple Euler stepping because it samples the derivative at four points per timestep instead of one.

The physics model:

- **Gravity** (obviously)
- **Aerodynamic drag**: Cd = 0.47 for a smooth sphere, air density 1.225 kg/m^3 at sea level. Drag force scales with the square of velocity, so fast shots slow down a lot more than slow ones.
- **Magnus lift**: Spinning balls curve. Topspin gives upward lift and extends range. Backspin pushes the ball down. We use Cm = 0.2, which is a conservative estimate. Magnus is about 8x less sensitive than slip factor, so even if we're off by 2x on the coefficient, it only shifts RPM by 50-100.

For each distance from 0.50m to 5.00m (at 0.05m steps), the simulator binary-searches for the RPM that lands the ball at hub height. 25 iterations of binary search gives ~0.004 RPM precision, way more than we need. The result is ~90 entries, each with the exact RPM and time of flight for that distance. The whole table generates in about 200ms at startup.

You don't need a real robot to get started. Plug in your exit height, launch angle, wheel diameter, and a rough slip factor. The simulator gives you physics-based shot parameters on day one. Then you tune from real data.

## How Slip Factor Works

The ball doesn't leave the wheel at wheel speed. It slips. The exit velocity is:

```
v_exit = slipFactor * RPM * pi * wheelDiameter / 60
```

Slip factor (typically 0.5 to 0.85) depends on the wheel material, ball compression, and how long the ball stays in contact with the wheel. A urethane wheel gripping a foam ball might get 0.75. A smooth PLA hood might only get 0.55.

This matters more than almost any other variable. Being off on slip factor by 0.1 shifts the required RPM by 400+. Being off on Magnus by 2x only shifts it by 50-100. That's why field calibration of the slip factor matters more than getting the aerodynamics model perfect.

## Hardware Analysis Tools

Before we wrote robot code, we wanted to answer hardware questions with math instead of guessing. We built three tools.

### FlywheelMassSweep

Given motor specs (NEO stall torque, KV), a gear ratio, and a wheel diameter, this sweeps flywheel inertia to find the minimum mass that meets performance targets. It models:

- **RPM drop on impact**: Brettle impulse-momentum for how much the flywheel slows down when a ball hits it
- **Battery voltage sag**: Open circuit voltage minus internal resistance times current draw during recovery
- **Motor thermal rise**: Copper winding resistance goes up ~0.39% per degree, which shifts the torque curve
- **Current limiting**: SparkMax 60A limit caps recovery torque
- **Rapid-fire cadence**: Multiple shots in a row to check if the motor can recover fast enough between shots

We used this to tell our mechanical team: "for a 4-inch wheel at 60 degrees with a 1:1 ratio, the flywheel needs at least this much moment of inertia to recover within 0.8 seconds per shot." That let them design the flywheel before the shooter was built.

### AngleSweep

Compares launch angles (45-70 degrees) across different shooter configurations. Shows RPM and time-of-flight at 10 key distances for each angle.

We ran this when the team was deciding between the front flywheel (68 degrees) and the rear drum shooter (60 degrees). It showed the mechanical team exactly how RPM requirements change with angle, so they could make the decision with data instead of intuition.

### SlipFactorSweep

Sweeps across slip factor values (0.55 pessimistic to 0.85 best-case) and shows the RPM range you need at each distance. It answers the question: "if our slip factor is somewhere between 0.6 and 0.8, how much does that change the RPM we need?"

The answer is a lot. That uncertainty is why we built the 3-layer calibration workflow: sim baseline, per-distance corrections from practice, and live copilot trim during matches. Each layer narrows the gap left by the previous one.

## 3-Layer Calibration Workflow

1. **Sim baseline**: ProjectileSimulator generates the starting LUT from CAD measurements and estimated slip factor. Good enough for day-one testing.
2. **Per-distance corrections**: After real-robot testing, we add RPM offsets at specific distances where the sim was wrong. `ShotCalculator.addRpmCorrection(distance, deltaRpm)`.
3. **Live copilot trim**: During matches, the copilot's D-pad adjusts a flat RPM offset across all distances. If shots are consistently falling short, bump it up 50 RPM. `ShotCalculator.adjustOffset(deltaRpm)`.

The sim gives you a foundation. Real data refines it. The copilot adapts in real time.

All of these tools are in our [open-source fire control repo](https://github.com/eeveemara/frc-fire-control) alongside the three core files.

---

**Related:** [Fire Control Pipeline](../architecture/fire-control-pipeline.md) | [Ball Physics Simulation](fuel-simulation.md) | [Neural Network Coprocessor](nn-coprocessor.md) | [Community Impact](community-impact.md)
