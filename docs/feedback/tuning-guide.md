# Driver Feedback Tuning Guide

Test and tune all LED and haptic signals with the drivers before competition.

## How to Test

### Haptics
1. Connect both controllers (driver port 0, copilot port 1)
2. Open Elastic dashboard, load `ref/dashboards/elastic/rebuilt_feedback_test.json`
3. Set `DriverFeedback/TestPattern` slider to 1-9 to play each pattern
4. Set to 0 to stop

### LEDs
1. Set `LED/TestState` slider to 1-10 to force each LED state
2. Set to 0 for normal operation
3. `LED/brightness` slider (0.0-1.0) controls overall brightness

Both are FMS-locked (can't accidentally trigger during a match).

---

## Haptic Patterns (Controller Vibration)

| # | Pattern | Priority | Who Feels It | What It Means | What It Feels Like |
|---|---------|----------|-------------|---------------|-------------------|
| 1 | **AUTO_WON** | CRITICAL | BOTH | We scored more fuel in auto, our hub goes first | Strong buzz, pause, two right taps |
| 2 | **AUTO_LOST** | CRITICAL | BOTH | We lost auto, opponent hub goes first | Strong buzz, pause, one left thump |
| 3 | **ENDGAME_WARNING** | CRITICAL | BOTH | 30 seconds left in match | Two quick double-pulses |
| 4 | **READY_TO_SHOOT** | HIGH | COPILOT | All conditions met, pull the trigger | Gentle right tap |
| 5 | **HUB_ACTIVATED** | HIGH | COPILOT | Hub just became active, start shooting | Two right pings |
| 6 | **HUB_DEACTIVATED** | HIGH | COPILOT | Hub just went inactive, stop shooting | One left thump |
| 7 | **HUB_COUNTDOWN_5** | MEDIUM | COPILOT | 5 seconds until hub shift | Light pulse |
| 7 | **HUB_COUNTDOWN_4** | MEDIUM | COPILOT | 4 seconds until hub shift | Slightly stronger pulse |
| 7 | **HUB_COUNTDOWN_3** | MEDIUM | COPILOT | 3 seconds until hub shift | Medium pulse |
| 7 | **HUB_COUNTDOWN_2** | MEDIUM | COPILOT | 2 seconds until hub shift | Strong pulse |
| 7 | **HUB_COUNTDOWN_1** | MEDIUM | COPILOT | 1 second until hub shift | Strongest pulse |
| 8 | **JAM_DETECTED** | HIGH | COPILOT | Ball jam detected in intake/indexer/agitator | Left-right-left rocking (feels "stuck") |
| 9 | **GAME_DATA_MISSING** | HIGH | BOTH | FMS connected but no game data received | Three strong symmetric pulses |

### Progressive Aim (continuous, not a discrete pattern)
When AimAndShootCommand is active, copilot feels heading error converging:
- **>10 degrees off**: no rumble (too far, don't shoot yet)
- **10 to 0 degrees**: right side rumble intensity increases as aim gets closer
- **On target**: max right rumble (fire now)
- **Auto-clears after 250ms** if AimAndShoot stops updating (safety timeout)

### Priority Rules
Higher priority patterns interrupt lower ones. CRITICAL > HIGH > MEDIUM > LOW.
Progressive aim only runs when no discrete pattern is playing.

### Who Feels What
| Target | Who | Why |
|--------|-----|-----|
| DRIVER | Driver only | Awareness signals (spin-up rumble if added) |
| COPILOT | Copilot only | Scoring signals (aim, ready, hub state, jams) |
| BOTH | Both controllers | Match events (teleop start, endgame, game data) |

If copilot controller is not connected, COPILOT signals fall back to DRIVER.

---

## LED States (AddressableLED strip)

| # | State | Color/Pattern | What It Means | When It Shows |
|---|-------|--------------|---------------|---------------|
| 1 | **DISABLED** | Dim green | Robot disabled, ready | Robot off or disabled |
| 2 | **IDLE** | Solid green | Enabled, nothing happening | Teleop, no subsystems active |
| 3 | **MATCH_OVER** | Green breathing | Match ended | After disable following an enabled period |
| 4 | **VISION_LOCKED** | Blue breathing | Camera sees AprilTags, pose is good | Vision tracking active |
| 5 | **AUTO_RUNNING** | Rainbow scroll | Autonomous mode running | During auto period |
| 6 | **WARNING** | Orange chase | Something needs attention | Low battery, vision degraded, etc. |
| 7 | **SHOOTER_SPINUP** | Blue progress bar | Shooter spinning up to target RPM | Shooter PID active, not at speed yet |
| 8 | **AIM_PROGRESS** | Blue pulse (speed varies) | Heading converging on hub | AimAndShoot active, pulse speeds up as aim improves |
| 9 | **READY_TO_SHOOT** | Solid blue | All conditions met, fire when ready | Shooter at speed + aimed + valid shot |
| 10 | **CRITICAL_ALERT** | Red/orange chase | Something is wrong | Battery critical, CAN fault, etc. |

### LED State Priority (highest to lowest)
1. CRITICAL_ALERT (red/orange)
2. WARNING (orange)
3. READY_TO_SHOOT (solid blue)
4. AIM_PROGRESS (blue pulse)
5. SHOOTER_SPINUP (blue progress)
6. AUTO_RUNNING (rainbow)
7. VISION_LOCKED (blue breathe)
8. MATCH_OVER (green breathe)
9. IDLE (solid green)
10. DISABLED (dim green)

### Safety Rules (R203-M compliant)
- No flashing above 1.5 Hz (seizure prevention, rule requires <5 Hz)
- Minimum 15% brightness even in "off" segments (never goes full black)
- 0.5s minimum hold time prevents rapid state flickering

---

## What Drivers Should Know

### Copilot
- **Right side buzz = good news** (ready to shoot, hub active)
- **Left side buzz = bad news** (hub deactivated, lost auto)
- **Rocking buzz = jam**, wait for auto-reverse
- **Progressive aim on right side while holding RT**, fire when it's strongest
- **Countdown pulses get stronger** as hub shift approaches, finish shooting before it peaks

### Driver
- **Two quick pulses = endgame** (30s left)
- **Strong buzz at teleop start = auto result** (right taps = won, left thump = lost)
- **Three pulses = game data missing**, tell drive coach

### LEDs (both drivers watch)
- **Blue = shooting mode** (progress bar → pulse → solid = increasing readiness)
- **Green = idle/ready** (nothing happening)
- **Orange = warning** (check dashboard)
- **Red = critical** (stop, something is wrong)
- **Rainbow = auto running** (don't touch anything)

---

## Tuning Checklist

Before competition, test each signal with the actual drivers:

- [ ] Haptic 1-2: Can drivers tell apart AUTO_WON vs AUTO_LOST?
- [ ] Haptic 3: Does ENDGAME_WARNING feel urgent enough?
- [ ] Haptic 4: Can copilot feel READY_TO_SHOOT while concentrating?
- [ ] Haptic 5-6: Can copilot distinguish HUB_ACTIVATED vs HUB_DEACTIVATED?
- [ ] Haptic 7: Are countdown pulses noticeable but not distracting?
- [ ] Haptic 8: Does JAM_DETECTED feel different from other patterns?
- [ ] Progressive aim: Does copilot instinctively know when to fire?
- [ ] LED brightness: Visible on the field under arena lights?
- [ ] LED states: Can pit crew read robot state from 20 feet away?

If any signal is confusing or too subtle, adjust the Step values (left/right intensity 0.0-1.0, duration in seconds) in `DriverFeedback.java`.
