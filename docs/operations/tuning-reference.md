# Tuning & Adjustment Reference

TunableNumber is our system for live-adjusting values without redeploying code. When `Constants.TUNING_MODE = true`, each TunableNumber publishes to SmartDashboard as a NetworkTables entry. You can change it from the Elastic dashboard and the robot picks up the new value immediately.

Safety lock: when FMS is attached (at a real competition match), TunableNumber always returns the compile-time default from Constants.java regardless of TUNING_MODE. You can't accidentally change PID gains mid-match.

## All Tunable Parameters

### Shooter

| Key | Default Source | What It Does |
|-----|---------------|-------------|
| `Shooter/kP` | ShooterConstants.P | Proportional gain for flywheel speed control |
| `Shooter/kI` | ShooterConstants.I | Integral gain (usually 0 or very small) |
| `Shooter/kD` | ShooterConstants.D | Derivative gain for dampening overshoot |
| `Shooter/FF` | ShooterConstants.FF | Feedforward, the primary term for velocity control |
| `Shooter/TargetRPM` | ShooterConstants.TARGET_RPM | Default flywheel target speed |
| `Shooter/ToleranceRPM` | ShooterConstants.SPEED_TOLERANCE_RPM | How close to target counts as "at speed" |
| `Shooter/ShotDropRPM` | ShooterConstants.SHOT_DETECTION_DROP_RPM | RPM dip that indicates a ball was fired |

### Indexer

| Key | Default Source | What It Does |
|-----|---------------|-------------|
| `Indexer/kP` | IndexerConstants.P | Proportional gain |
| `Indexer/kI` | IndexerConstants.I | Integral gain |
| `Indexer/kD` | IndexerConstants.D | Derivative gain |
| `Indexer/FF` | IndexerConstants.FF | Feedforward |
| `Indexer/TargetSpeed` | IndexerConstants.TARGET_SPEED | Feed wheel speed |
| `Indexer/JamAmps` | IndexerConstants.JAM_CURRENT_THRESHOLD_AMPS | Current above this triggers jam detection |
| `Indexer/JamSeconds` | IndexerConstants.JAM_TIME_THRESHOLD_SECONDS | How long high current must persist before jam |

### Agitator

| Key | Default Source | What It Does |
|-----|---------------|-------------|
| `Agitator/kP` | AgitatorConstants.P | Proportional gain |
| `Agitator/kI` | AgitatorConstants.I | Integral gain |
| `Agitator/kD` | AgitatorConstants.D | Derivative gain |
| `Agitator/FF` | AgitatorConstants.FF | Feedforward |
| `Agitator/TargetSpeed` | AgitatorConstants.TARGET_RPM | Agitator target speed |
| `Agitator/JamAmps` | AgitatorConstants.JAM_CURRENT_THRESHOLD_AMPS | Jam current threshold |
| `Agitator/JamSeconds` | AgitatorConstants.JAM_TIME_THRESHOLD_SECONDS | Jam time threshold |

### Intake

| Key | Default Source | What It Does |
|-----|---------------|-------------|
| `Intake/Speed` | MotorConstants.DESIRED_INTAKE_SPEED | Intake roller speed |

### Hanger

| Key | Default Source | What It Does |
|-----|---------------|-------------|
| `Hanger/kP` | HangerConstants.P | Proportional gain |
| `Hanger/kD` | HangerConstants.D | Derivative gain |

### Driver Controls

| Key | Default Source | What It Does |
|-----|---------------|-------------|
| `Driver/Deadband` | OperatorConstants.DEADBAND | Joystick deadband (ignore small inputs) |
| `Driver/TurnConstant` | OperatorConstants.TURN_CONSTANT | Turning sensitivity multiplier |

### Feedback Testing

| Key | Default | What It Does |
|-----|---------|-------------|
| `DriverFeedback/TestPattern` | 0 | Select a haptic pattern to test (1-10) |
| `DriverFeedback/hapticScale` | 1.0 | Overall haptic intensity multiplier |
| `LED/brightness` | 0.8 | LED strip brightness (0.0 to 1.0) |
| `LED/TestState` | 0 | Force a specific LED state for testing |

## How to Tune PID in the Lab

1. Set `Constants.TUNING_MODE = true` and deploy
2. Open Elastic, load `rebuilt_tuning_session.json`
3. Go to the subsystem tab you're tuning (e.g., Shooter)
4. Start with FF only (set kP/kI/kD to 0). Increase FF until the motor reaches ~80-90% of target speed
5. Add kP gradually until it reaches target. If it oscillates, you added too much
6. Add kD if there's overshoot. Small amounts go a long way
7. Add kI only if there's a persistent steady-state error. Keep it very small
8. Watch `Shooter/AtSpeedPercent` on the graph. You want it steady at 98-102%
9. Once tuned, copy the values back into Constants.java
10. Set `TUNING_MODE = false` before competition

## RPM Offset at Competition

The copilot's bumper buttons adjust the flywheel RPM in real time:
- Each press adds or subtracts 25 RPM
- Clamped to +/- 200 RPM from the calculated target
- Resets to zero when the robot is disabled

Use this if shots are consistently landing high or low. If you're adding more than +/- 100 RPM, something else is probably wrong (check flywheel condition, ball wear, battery voltage).

## Important Notes

- TunableNumbers only publish when `TUNING_MODE = true`. In competition mode, they don't clutter NetworkTables bandwidth.
- The `ifChanged()` helper method only runs the PID update callback when a value actually changes, so there's zero overhead during matches.
- Never tune at competition with `TUNING_MODE = true` and FMS disconnected. The FMS lock only works when FMS is attached. If you're in practice mode without FMS, someone could accidentally drag a slider.

---

[Back to Documentation Home](../README.md)
