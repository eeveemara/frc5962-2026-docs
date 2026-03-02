# Elastic Dashboard Guide

Elastic is a real-time FRC dashboard that connects to your robot over NetworkTables. It shows live sensor data, boolean states, camera streams, and lets you adjust tunable values on the fly. Think of it as the cockpit instrument panel for the robot.

We have 4 Elastic layouts, each designed for a specific situation. You load them from `ref/dashboards/elastic/`.

## How to Load a Layout

1. Open Elastic Dashboard
2. Go to **File > Open Layout**
3. Navigate to `ref/dashboards/elastic/` and pick the JSON file you need
4. Done. The tabs and widgets will populate automatically.

To save changes: **File > Save Layout As** (saves as JSON).

---

## 1. Competition Layout (`rebuilt_driver_competition.json`)

**When to use:** During actual matches. This is what the drive team sees on the field.

### Match Tab
The primary driver view. Everything at a glance.

| Widget | What It Shows | What to Watch For |
|--------|--------------|-------------------|
| Match Time | Countdown timer | Goes red at 15s, yellow at 30s |
| READY TO SHOOT | Big green/red box | Green = all 6 scoring conditions met, safe to fire |
| Camera | Live camera feed | Should have a clear view of the field |
| Target Lock | Vision has a target | Orange = no lock, green = locked |
| Battery | Voltage bar (8-13V) | Should stay above 11V during a match |
| Shooter RPM | Flywheel speed (0-5700) | Should reach target before shooting |
| Hub Active | Current hub is scoring | Orange when hub is inactive (shifting) |
| Hub Shift# / Shift Timer | Which hub and time to next shift | Plan shots around shift timing |
| Auto Selector | Pick autonomous routine | Set this BEFORE the match starts |
| AMDA Mode / Confidence | Feedback system status | Shows current awareness mode |
| JAM! / Intake Jam | Jam detection for indexer and intake | Red = jam detected, auto-reverse should trigger |
| Agitator | Whether agitator is running | Green when active |
| Alerts | Active system alerts | Check for anything critical |

### Coach Tab
For the coach standing behind the drivers. More stats, less action.

Key additions over the Match tab: Shots Scored (big number), Fire Rate (shots/min), Hub Utilization, Active Hub Shots, Total Current draw, Brownout Risk, and the full alert bar. This helps the coach decide when to push harder or ease off.

---

## 2. Pit Diagnostic Layout (`rebuilt_pit_diagnostic.json`)

**When to use:** Between matches in the pit. Run this immediately after pulling the robot off the field.

### Quick Check Tab
A 30-second health scan. All green = good to go.

- **Top row:** Battery voltage, CAN utilization (should be under 70%), All CAN OK (green = every device found), Browned Out flag
- **Middle rows:** Motor temps for Shooter, Indexer, Intake, Agitator, Hanger (all should be under 60C). Gyro/Camera/Jam/Overcurrent status booleans.
- **Swerve modules:** All 8 connections (FL/FR/BL/BR drive + turn). Any red = loose CAN wire.
- **Bottom:** Disconnected device list, overcurrent channel, swerve drive temps, and alerts.

### Diagnostics Tab
Run structured diagnostic checks. Hit **Safe Check** (quick, no motion) or **Full Check** (spins motors). Watch the ALL PASSED indicator, plus Pass/Warn/Fail/Skip counts. Also shows gyro drift, vision dropout %, drive response time, and shooter spin-up time.

### System Detail Tab
Deep dive into system internals: battery, CAN util, loop time (should be under 20ms), bandwidth, CAN TX/RX errors, CAN bus off flag, loop overruns, CPU temp, RSL state, bandwidth warning, per-subsystem current draws, overcurrent details, ghost commands, brownout risk/voltage, rail voltages (3.3V/5V/6V), torque reversal alerts, and total CAN faults.

---

## 3. Tuning Session Layout (`rebuilt_tuning_session.json`)

**When to use:** During lab sessions when you are tuning PID, adjusting thresholds, or characterizing subsystems.

### Shooter Tab
The main PID tuning view. A live RPM graph on the left, with editable kP/kI/kD/FF fields (with submit buttons so changes push immediately). Set Target RPM, Tolerance, and Shot Drop threshold. Readbacks include At Speed %, spin-up time, velocity error, recovery time, shot detection, total shots, motor temp/current/output, and the shot predictor (distance, heading error, computed RPM).

### Intake Tab
Tune the intake pivot and roller. Live pivot position graph, editable PID for the pivot motor, deploy/retract position setpoints, roller speed. Status readbacks for pivot at-target, roller running, jam detection, stall state. Motor health for both pivot and roller (temp, current, output).

### Feeding Tab
Tune the indexer and agitator. Live indexer current graph (useful for seeing jam spikes). Editable PID for the indexer, target speed, jam current threshold, and jam duration threshold. Readbacks for jam state, stall duration, direction, RPM, output, jam count, jam frequency. Agitator section with similar readbacks. Torque reversal alert indicator.

### Hanger Tab
Simple view for climb tuning. Editable kP/kD, position/target readbacks, at-position and deployed/climbing booleans, climb progress bar, soft limit indicator, temp/current/output.

### Drive Input Tab
Tune driver feel. Sliders for exponential curve K (0=linear, 6+=aggressive), trigger slow range, slow-mode translation and rotation multipliers. Readbacks for slow mode active, trigger scale, effective scale, and current K value.

---

## 4. Feedback Test Layout (`rebuilt_feedback_test.json`)

**When to use:** When testing haptic feedback patterns and LED states. Have someone hold the controllers while you cycle through patterns.

### Feedback Test Tab
Split into three sections:

- **Haptic (left):** Pattern slider (0=off, 1-5 cycles through patterns), active pattern name, description, priority, left/right motor intensity bars, haptic scale slider (0-2x), pattern count, aim error, haptic target (DRIVER/COPILOT/BOTH).
- **LED (right):** State slider (0=off, 1-10 cycles through LED states), current state name, description, hardware connected indicator, brightness slider, state change count, brightness readback.
- **AMDA (far right):** Mode, confidence level, mode changed flag, vision locked, ready to shoot, hub active, target distance, vision confidence, consecutive frames.

### Hub Shift Practice Tab
Simulates hub shift timing so drivers can practice reacting. Shows shift label, simulated match time, countdown to next shift, hub active indicator, haptic pattern, motor intensity bars, shift number, LED state. Controls: Won Auto toggle, time scale slider (0.25x-5x speed), restart button, haptic scale. Also shows previous shift info and pattern count.

---

## Tips

**During a match:** Keep the Competition layout on the Match tab. The coach should have their own screen on the Coach tab. The only things worth glancing at mid-match are READY TO SHOOT and JAM.

**In the pit:** Run Quick Check first. If anything is red, go to System Detail for the specifics. Run Diagnostics Full Check if you have time between matches (takes about 15 seconds).

**During tuning:** Always watch the battery voltage bar at the bottom of each tuning tab. Low battery skews PID behavior. Change one parameter at a time and let the graph settle before judging the result.

**Creating/modifying layouts:** Drag widgets to reposition. Right-click a widget to edit its properties (topic, display range, colors). Drag edges to resize. Add new widgets from the widget palette. Save as JSON when you are happy with the arrangement.

---

**See also:** [AdvantageScope Guide](advantagescope-guide.md) | [Dashboard Quick Reference](quick-reference.md)
