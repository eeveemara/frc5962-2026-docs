# What We Learned & Design Decisions

This doc explains why we built our system the way we did. Every major design choice came from a real problem we hit, and most of them came from the question "what happens when this breaks during a match?" It's useful for judges who want to understand our thinking, and for future team members who'll inherit this codebase.

## Key Design Decisions

### 1. Why Separate Telemetry from Subsystems

Originally we had logging and detection logic mixed into the subsystem classes. Then one bad sensor read threw an exception, and the entire subsystem crashed. The motor stopped, the logging stopped, everything died because it was all in the same code path.

Now subsystems only do motor control. Each telemetry class wraps every hardware read in a try-catch, so if a sensor returns garbage, the telemetry class catches it, logs safe defaults, and tries again next cycle. The motor keeps running because it's in a completely separate class. One broken sensor doesn't cascade.

### 2. Why SafeLog

Even within a single telemetry class, we wrap each individual signal publication in its own crash isolation. If one signal computation produces a NaN or throws, the other 499 signals still publish normally. We learned this the hard way when a division-by-zero in one velocity calculation took down an entire telemetry class worth of logging. SafeLog makes every signal independent.

### 3. Why Two Controllers

In a real aircraft, the pilot flies and the weapons officer targets. We applied the same idea: the driver handles movement (swerve drive, positioning, defense avoidance) and the copilot handles scoring (watching progressive aim feedback, pulling the trigger at the right moment, adjusting RPM offsets, managing role switches).

Without this split, one person has to simultaneously drive into position AND watch for the ReadyToShoot signal AND pull the trigger at the right moment. That's three cognitive tasks competing for attention during a loud, chaotic match. With two controllers, each person can focus on what they're responsible for, and the haptic feedback goes to the right person automatically.

### 4. Why Confidence Scoring

A naive approach is "aim at the target and fire." But that misses too often because it doesn't account for distance, velocity, flywheel stability, or vision quality.

Our ShotConfidence system computes a physics-based score from 0-100% using five weighted components. The copilot feels this as progressive haptic feedback. At 30% confidence, there's a light rumble. At 80%, it's strong. At the threshold, the ReadyToShoot pulse fires. Over time, the copilot learns to feel when a shot is good without ever looking at a screen. The copilot stops thinking about numbers and just feels when the shot is ready.

### 5. Why Four Feedback Channels (AMDA)

We call it Adaptive Multi-Modal Driver Awareness because different situations need different senses:

- **Haptic** goes straight to your hands. Works even when you're staring at the field and can't read a screen. Works in a loud arena where audio cues are useless.
- **LEDs** are visible to the drive coach and pit crew from across the field. They show robot state at a glance without requiring a dashboard connection.
- **Dashboard** has full detail for between-match analysis and pit diagnostics. Too much information for mid-match use, perfect for debugging.
- **Camera HUD** (in progress) overlays targeting info directly in the camera feed, so the copilot sees confidence and aim state right in the video stream.

The ChannelCoordinator ties them together. When vision confidence drops, it adjusts all four channels simultaneously. This isn't just redundancy. Each channel serves a different person and situation. For more on the feedback system, see [Driver Feedback](../feedback/driver-feedback.md).

### 6. Why Hysteresis on Vision Confidence

Without hysteresis, when vision confidence sits near a threshold, it rapidly flickers between HIGH and LOW every cycle. This makes the LED flash erratically and the haptic intensity jump around. Annoying and useless.

With hysteresis (drops to LOW at <40%, recovers to HIGH at >=55%), there's a 15-point dead zone. The system settles into one state and stays there. The operators see stable, trustworthy feedback instead of noise.

### 7. Why Mutation Testing Matters More Than Test Count

Finding a bug in the lab takes 5 minutes to fix. Finding the same bug at competition, with 3 minutes between matches and pressure from the drive team, takes forever and the fix might introduce something worse. So we write a lot of tests. But the real question isn't "how many tests do we have?" It's "would our tests actually catch a real bug?"

PITest mutation testing answers that question. It changes the code (flips a > to >=, removes a line, changes a constant) and checks if any test fails. If no test catches the change, we have a blind spot. Our kill rate (~53%) tells us what percentage of real mutations we'd catch. That's a more honest metric than code coverage, which only measures "did the line execute" not "did we verify it's correct." For the full deep dive, see [Testing & Quality](testing-and-quality.md).

## What We'd Do Differently Next Season

- **Start two-controller routing earlier.** We added it later in the build season and it works great, but the driver and copilot would've benefited from more practice time with the split controls.
- **Lock down the signal contract sooner.** Adding signals late in the season means dashboard layouts need updating, verification tools need re-running, and there's always a chance of naming conflicts.
- **Build the log analytics platform earlier.** We designed it thoroughly but only got ~40% implemented. Starting earlier would've given us automated post-match analysis at competition instead of manual AdvantageScope review.
- **Get mechanical measurements on day one.** Three placeholder constants (exit height, launch angle, wheel diameter) blocked our neural network training pipeline for 2 weeks. A 10-minute measurement session with the mechanical team would've unblocked it immediately.

## What We're Proudest Of

The moment it all clicked was during our test when the copilot said "I can feel when the shot is ready." That's exactly the point. The robot assesses conditions using physics, the copilot receives that assessment through progressive haptic feedback, and the driver flies the robot into position. Nobody has to stare at a screen during a match. The information just flows to the right person through the right channel at the right time.

The crash isolation architecture also proved itself: during testing, we deliberately injected faults into individual telemetry classes, and every time, the rest of the system kept running. That's the kind of reliability that matters when you're on the field and something unexpected happens.

---

[Back to Documentation Home](../README.md)
