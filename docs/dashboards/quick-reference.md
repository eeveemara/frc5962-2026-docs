# Dashboard Quick Reference

Cheat sheet for finding the right dashboard view for what you need to check.

## "I Want to Check X" Lookup Table

| I want to check... | Elastic Layout | Tab | AdvantageScope Layout |
|---------------------|---------------|-----|----------------------|
| Is the robot ready to shoot? | Competition | Match | match_review (Scoring Timeline) |
| Why a shot missed | - | - | shooter_tuning (Shot Prediction, 3D Trajectory) |
| Shooter flywheel speed | Competition | Match (Shooter RPM bar) | shooter_tuning (Velocity vs Target) |
| Shooter PID tuning | Tuning Session | Shooter | shooter_tuning (Velocity vs Target) |
| Intake jam / stall | Competition | Match (Intake Jam box) | pit_triage (Subsystem Faults) |
| Indexer jam status | Competition | Match (JAM! box) | pit_triage (Subsystem Faults) |
| Battery voltage | Competition | Match or Coach | system_overview (Battery & Power) |
| Battery trending over match | - | - | power_and_health (Battery & Power) |
| CAN bus health | Pit Diagnostic | Quick Check | system_overview (CAN Bus Health) |
| Motor temperatures | Pit Diagnostic | Quick Check | system_overview (Motor Temperatures) |
| Swerve module connections | Pit Diagnostic | Quick Check (FL/FR/BL/BR) | pit_triage (Swerve Health) |
| Vision target lock | Competition | Match (Target Lock box) | vision_debug |
| Vision confidence | Feedback Test | Feedback Test (Vision Conf) | vision_debug |
| Hub shift timing | Competition | Match (Shift Timer) | match_review (Scoring Timeline) |
| Current alliance role | - | - | cycle_and_strategy |
| Haptic feedback patterns | Feedback Test | Feedback Test | - |
| LED states | Feedback Test | Feedback Test | - |
| Drive input feel | Tuning Session | Drive Input | drive_and_auto |
| Robot position on field | - | - | match_review (Field Position / 3D Field) |
| Shot trajectory | - | - | shooter_tuning (3D Trajectory) |
| Ghost / orphaned commands | Pit Diagnostic | System Detail | pit_triage (Ghost Commands) |
| Loop timing / overruns | Pit Diagnostic | System Detail | system_overview (Loop Timing) |
| Brownout risk | Competition | Coach (Brownout Risk) | system_overview (Battery & Power) |
| Post-match health summary | - | - | system_overview (Post-Match Summary) |
| Climb progress | Tuning Session | Hanger | mechanism_debug |
| Run diagnostics | Pit Diagnostic | Diagnostics | - |

## Key Signals Everyone Should Know

| Signal Path | Type | What It Means |
|-------------|------|---------------|
| `Scoring/ReadyToShoot` | boolean | True = all 6 conditions met (shooter ready, indexer clear, vision locked, has ball, fire authorized, shot confident). Safe to fire. |
| `Scoring/ShotConfidence` | 0-100% | How confident we are the shot will land. Geometric mean of 5 components. |
| `Shooter/VelocityRPM` | number | Current flywheel speed in RPM. Target is distance-dependent. |
| `Shooter/AtSpeed` | boolean | True = flywheel is within tolerance of target RPM. |
| `Intake/Stalled` | boolean | True = intake motor stall detected. Possible jam. |
| `Indexer/JamDetected` | boolean | True = indexer jam detected. Copilot feels an L-R-L buzz. |
| `Vision/LockedOnTarget` | boolean | True = camera has a valid target lock (enough consecutive frames). |
| `Match/TimeRemaining` | seconds | Seconds left in the current match period. |
| `AMDA/Confidence` | LOW/HIGH | Vision confidence level. Hysteresis: drops to LOW below 40%, recovers to HIGH at 55%+. Affects feedback intensity. |
| `Strategy/CurrentRole` | SHOOTER/FEEDER | Current alliance role. Set by copilot at match start. |
| `SystemHealth/BatteryVoltage` | volts | Main battery voltage. Below 11V is concerning. Below 9V risks brownout. |
| `Scoring/Conditions/HubActive` | boolean | True = current hub window is active for scoring. False during shifts. |

## TunableNumber Reference

These values can be adjusted live through SmartDashboard during testing. They are locked when FMS is connected (competition mode).

| Category | Tunable | Default Location | What It Controls |
|----------|---------|-----------------|-----------------|
| Shooter | kP, kI, kD, FF | `SmartDashboard/Shooter/` | Flywheel PID gains |
| Shooter | TargetRPM, ToleranceRPM | `SmartDashboard/Shooter/` | Speed setpoint and at-speed window |
| Shooter | ShotDropRPM | `SmartDashboard/Shooter/` | Velocity drop threshold for shot detection |
| Indexer | kP, kI, kD, FF | `SmartDashboard/Indexer/` | Indexer PID gains |
| Indexer | JamAmps, JamSeconds | `SmartDashboard/Indexer/` | Jam detection thresholds |
| Intake Pivot | kP, kI, kD, FF | `SmartDashboard/IntakeActuator/` | Pivot arm PID gains |
| Intake Pivot | DeployPos, RetractPos | `SmartDashboard/IntakeActuator/` | Pivot setpoints (rotations) |
| Intake | Speed | `SmartDashboard/Intake/` | Roller speed |
| Hanger | kP, kD | `SmartDashboard/Hanger/` | Climb PID gains |
| Driver | ExpCurveK | `SmartDashboard/Driver/` | Stick exponential curve (0=linear, 3=moderate) |
| Driver | TriggerSlowRange | `SmartDashboard/Driver/` | How much the trigger slows you (0.7 = 30% speed at full squeeze) |
| Driver | SlowModeTranslation | `SmartDashboard/Driver/` | Slow-mode translation multiplier |
| Driver | SlowModeRotation | `SmartDashboard/Driver/` | Slow-mode rotation multiplier |
| Haptic | hapticScale | `SmartDashboard/DriverFeedback/` | Global haptic intensity multiplier (0-2x) |
| LED | brightness | `SmartDashboard/LED/` | LED strip brightness (0-1) |
| LED | TestState | `SmartDashboard/LED/` | Force a specific LED state for testing (0-9) |
| Haptic | TestPattern | `SmartDashboard/DriverFeedback/` | Force a specific haptic pattern (0-9) |

## Layout File Locations

All layouts live in `dashboards/`:

```
dashboards/elastic/
    rebuilt_driver_competition.json
    rebuilt_pit_diagnostic.json
    rebuilt_tuning_session.json
    rebuilt_feedback_test.json

dashboards/advantagescope/
    match_review.json
    cycle_and_strategy.json
    mechanism_debug.json
    vision_debug.json
    shooter_tuning.json
    power_and_health.json
    system_overview.json
    pit_triage.json
    drive_and_auto.json
    video_showcase.json
    full_video_showcase.json
```

---

**Full guides:** [Elastic Guide](elastic-guide.md) | [AdvantageScope Guide](advantagescope-guide.md)
