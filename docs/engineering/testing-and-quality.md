# Testing & Quality Assurance

Here's how we test our robot code and why we care about it. If mutation testing is new to you, that is fine. We'll walk through it.

## What is JUnit Testing?

In simple terms: we write small programs that check if our robot code does what we expect. Each test sets up a scenario, runs the code, and checks the result. If anything is wrong, the test fails and tells us exactly what broke.

For example, say we have code that detects whether a motor is stalled. A test for that would:

1. Simulate a motor spinning at 0 RPM with high current draw (that's what a stall looks like)
2. Run the detection logic
3. Check that the code correctly reports "stalled = true"
4. Then simulate the motor spinning normally again
5. Check that "stalled" clears to false

If someone accidentally breaks the stall detection logic later, this test catches it immediately instead of letting the bug ride all the way to competition.

We have hundreds of these tests across 74 test files, and they all run in about 10 seconds.

## Why We Care More About Test Quality Than Test Count

The number of tests is not the interesting part. What matters is whether those tests would catch a real bug. We found that out the hard way when the copilot controller kept buzzing after aiming stopped. We had 274 passing tests and not one of them checked whether the vibration actually turned off.

That experience changed how we think about testing. Now every test is a bug that can't come back. When we change something, the tests tell us immediately if we broke something. Not at 2am at competition, not when the shooter starts clicking during a qualification match. Right now, in the lab, with time to think.

It also means multiple people can work on the same codebase without guessing whether they broke someone else's logic. If we change a stall threshold, the tests tell us whether jam protection still behaves. If we refactor fire control, the tests tell us whether ReadyToShoot still makes sense.

## What We Test (and What We Don't)

We test the **detection logic** (telemetry), not the motor control (subsystems). Here's why:

- **Subsystems are simple.** They set a motor to a speed. Not much to go wrong there, and the interesting behavior happens in the motor controller firmware, not our code.
- **Telemetry is where the interesting logic lives.** Is the motor stalled? Is there a jam? Is the flywheel at speed? Is the shot ready? Is vision confidence high enough? These decisions have conditions, thresholds, state machines, and edge cases. That's where bugs hide.

This is a deliberate architectural choice. By separating motor control from detection logic, we can test the detection logic in simulation without needing a physical robot.

## Test Base Classes

We have two base classes that handle all the simulation boilerplate so each test file can focus on actual test logic:

**SparkSimTestBase** is for motor-backed subsystems like Shooter, Intake, and Indexer:
- Sets up SparkSim backends so motor encoder values are controllable
- Provides `setMotorVelocity(sim, rpm)` which directly sets the encoder reading. This is deterministic: you set 3000 RPM, you get 3000 RPM. No physics delay, no randomness.
- Never use the `iterate()` method in unit tests. It simulates motor physics with ~112ms of lag and is non-deterministic, which makes tests flaky.

**TelemetryTestBase** is for non-motor telemetry like match state, network health, and driver input:
- Sets up DriverStationSim for simulating robot mode, alliance color, and FMS connection
- Lighter weight since no motor simulation is needed

## Example: Testing Stall Detection

Here's how a stall detection test works conceptually:

1. **Setup**: Create the telemetry object. Set the simulated motor to 0 RPM, 0 current. Run an update cycle. Verify that `stalled` is false (the motor is just sitting there, not stalled).

2. **Trigger the stall**: Set motor velocity to 0 RPM but current to something high (say 30 amps). That's the signature of a stall: the motor is drawing lots of power but not spinning. Wait past the startup ignore window (0.5 seconds). Run an update cycle.

3. **Verify detection**: Check that the telemetry class now reports `stalled = true`.

4. **Clear the stall**: Set motor velocity back to normal (say 3000 RPM). Run an update cycle.

5. **Verify clearing**: Check that `stalled` goes back to false.

This pattern repeats across all our tests. Set up a known state, run the logic, check the result. The key part is we can simulate any motor condition we want without a physical robot.

## Code Coverage (Jacoco)

Before mutation testing, we ran Jacoco to see how much of our code is actually exercised by tests. The overall numbers are 49% instruction coverage and 34% branch coverage, but those are misleading because a lot of our code is hardware wiring (motor configuration, CAN setup) that can't run in simulation.

The number that matters: **76% coverage on core logic classes**. That includes the fire control pipeline, scoring readiness, jam detection, driver feedback routing, and hub shift timing. These are the classes where bugs actually hide, and they're well covered.

We use Jacoco as a "did we forget to test something?" tool, not a target to chase. 100% coverage is meaningless if the tests don't check the right things. That's where mutation testing comes in.

## What is Mutation Testing (PITest)?

Regular tests answer "does this code work in the cases we wrote down?" Mutation testing asks a harder question: **"would our tests notice if the code were wrong?"**

Here's how it works:

1. PITest takes our source code and makes tiny changes called **mutations**. For example:
   - Flip a `>` to `<` (so "is velocity greater than threshold" becomes "is velocity less than threshold")
   - Change a `+` to a `-`
   - Replace `true` with `false`
   - Remove a method call entirely
   - Change a constant from 0.5 to 0.0

2. For each mutation, PITest runs our full test suite.

3. If a test **fails**, the mutation is **"killed"**. That means our tests caught the artificial bug. Good.

4. If **no test fails**, the mutation **"survived"**. That means we have a gap in our testing. Something in our code could be wrong and we wouldn't know. Bad.

5. The **kill rate** is the percentage of mutations that were caught. Higher is better.

The easiest way to think about it is that mutation testing stress-tests the tests, not the robot.

## Our PITest Results

We run mutation testing on 10 target classes. Overall: **53% kill rate, 75% test strength**.

| Class | Kill Rate | Test Strength | What It Tests |
|-------|-----------|---------------|---------------|
| HubShiftEngine | 94% | 97% | Hub timing windows during the match |
| HubScoringUtil | 90% | 90% | Score detection from sensor readings |
| StrategySelector | 85% | 88% | Switching between SHOOTER and FEEDER roles |
| FireAuthorization | 81% | 81% | Whether the robot is allowed to shoot right now |
| DriverFeedback | 64% | 90% | Haptic patterns routed to the right controller |
| JamProtection | 51% | 52% | Jam detection and alert state machine |
| AlertManager | 35% | 45% | Alert raising for subsystem health issues |
| LEDStatusDisplay | 25% | 84% | Choosing the right LED color for robot state |

The classes with higher kill rates (HubShiftEngine, HubScoringUtil) have the most thorough tests. Classes with lower rates (AlertManager, LEDStatusDisplay) are areas where we could still add more targeted tests.

**Kill rate vs. test strength**: Kill rate is the percentage of all mutations killed. Test strength is the percentage killed out of the ones that were actually *reachable* by our tests (some mutations are in code paths that tests can't trigger due to hardware dependencies). Test strength is usually higher because it filters out the unreachable ones.

## How Mutation Testing Improved Our Code

**DriverFeedback** is the clearest success story. It started at a 40% kill rate, meaning our tests only caught 40% of injected bugs. After analyzing which mutations survived, we wrote targeted tests and pushed it to 64%. That's 24% more bugs our test suite would catch.

Here's a concrete example of the kind of thing mutation testing reveals:

PITest mutated a condition in the endgame warning from `matchTime <= 30.0` to `matchTime < 30.0`. None of our tests failed. Why? Because all our tests used match times that were clearly inside or clearly outside the threshold (like 20s and 45s). None tested the boundary at exactly 30.0 seconds. If the real code had this off-by-one error, a 30.0-second endgame warning would never fire, and we'd never know from our tests alone.

The fix: add a boundary test that sets match time to exactly 30.0 and verifies the endgame warning fires. That single test killed the mutation and would catch any future boundary regression.

That is why mutation testing matters to us. It finds blind spots in places where the code already looks reasonable and the normal tests are all green.

## Running Tests

All commands run from the `robotcode2026-lab/` directory:

```bash
./gradlew test                                          # All tests (~10 seconds)
./gradlew pitest                                        # Mutation testing (~5 minutes, build first)
./gradlew simulateJava                                  # Basic robot simulation
./gradlew simulateJava -DsimScenario=SubsystemSuite     # Full showcase scenario
```

JUnit results land in `build/reports/tests/test/index.html`. PITest results in `build/reports/pitest/index.html`.

## Simulation Scenarios

We have 19 simulation scenarios that test different match situations. Here are the most commonly used ones:

| Scenario | CLI Flag | What It Tests |
|----------|----------|---------------|
| SubsystemSuite | `-DsimScenario=SubsystemSuite` | Full field navigation, intake, shoot cycle. Best overall demo (100s, 14 phases). |
| CompetitionMatch | `-DsimScenario=CompetitionMatch` | Full 169s match with hub shifts, scoring during active windows, hanger climb. |
| RapidFire | `-DsimScenario=RapidFire` | Fast 1.5s shoot cycles to stress shooter and indexer signals (13s). |
| Brownout | `-DsimScenario=Brownout` | Two-stage voltage decline to trigger brownout detection and battery prediction (25s). |
| FaultInjection | `-DsimScenario=FaultInjection` | Walks voltage through all 4 brownout risk levels under motor load, then recovers (60s). |
| RapidModeTransition | `-DsimScenario=RapidModeTransition` | Rapid enable/disable cycling to stress mode transitions and command scheduling (30s). |
| FullVideoShowcase | `-DsimScenario=FullVideoShowcase` | Full match with HubArcDrive orbits during active shifts. For video recording (169s). |
| AMDAShowcase | `-DsimScenario=AMDAShowcase` | All 4 feedback channels: haptic, LED, HUD, dashboard through 13 phases (45s). |
| SignalCoverage | `-DsimScenario=SignalCoverage` | Gap-filler covering signals other scenarios miss: voltage sweep, mode transitions, stall cycles (30s). |
| HubShiftPractice | `-DsimScenario=HubShiftPractice` | Full 140s teleop with real-time hub shift haptics for driver training. |
| FuelPhysicsShowcase | `-DsimScenario=FuelPhysicsShowcase` | Ball physics demo with field collisions, scoring, and intake pickup. |
| BallCycleShowcase | `-DsimScenario=BallCycleShowcase` | Full intake-to-score ball cycling through the robot. |
| SOTMShowcase | `-DsimScenario=SOTMShowcase` | Shoot-on-the-move demo with velocity compensation and polar speed limiting. |
| FireControlTest | `-DsimScenario=FireControlTest` | Fire control pipeline exerciser: ShotCalculator, confidence, authorization. |
| ShooterTest | `-DsimScenario=ShooterTest` | Shooter subsystem isolation test: spin-up, at-speed latch, RPM recovery. |
| DefensiveFeeder | `-DsimScenario=DefensiveFeeder` | Feeder role demo with eject-to-feed-spot and role switching. |
| DriverPractice | `-DsimScenario=DriverPractice` | Free-drive practice with haptic and LED feedback active. |
| JudgeDemo | `-DsimScenario=JudgeDemo` | Interactive demo for judge pit visits. Judge drives robot with Xbox controller, shoots balls, sees full physics in AdvantageScope Field3d. ShotCalculator outputs displayed in real time. No time limit. |
| OffensiveBlitz | `-DsimScenario=OffensiveBlitz` | Aggressive multi-cycle scoring run to stress the full pipeline. |

**What you can verify in sim**: telemetry signals updating correctly, haptic feedback timing, state machine transitions, fire control computations, ReadyToShoot composite signal logic.

**What you can't verify in sim**: motor temperatures (SparkSim has no setTemperature), real CAN bus behavior, physical ball handling.

## Common Test Gotchas

**SparkSim: setVelocity vs iterate** \
`setMotorVelocity(sim, rpm)` sets the encoder value directly. Deterministic, instant, use this for tests. `iterate(vel, vbus, dt)` simulates motor physics with ~112ms lag and is non-deterministic. Never mix them on the same SparkSim instance.

**DriverStationSim.notifyNewData()** \
After changing any sim state (velocity, alliance, mode), you must call `DriverStationSim.notifyNewData()` or the robot code won't see the change until the next natural cycle.

**forkEvery = 1** \
Each test class runs in its own JVM process. This isolates CAN IDs so singleton subsystems from one test class don't collide with another. It's slower but prevents flaky failures that are incredibly hard to debug.

**TunableNumber in tests** \
The TunableNumber constructor writes the default value to SmartDashboard. If you're setting a custom value in your test, set it AFTER constructing the object that owns the TunableNumber, or your value gets overwritten by the default.

**Exit 134 crashes** \
These are a known WPILib HAL double-free on Linux. They look scary but they're not a test failure. Check the actual test results, not the process exit code.

## Validation Pipeline

Before any code goes to the robot, it passes through these gates:

```mermaid
flowchart LR
    B[Build] --> T[Tests]
    T --> P[PITest]
    P --> S[Simulation]
    S --> R[Robot Deploy]

    style B fill:#2563eb,stroke:#1d4ed8,color:#fff
    style T fill:#059669,stroke:#047857,color:#fff
    style P fill:#d97706,stroke:#b45309,color:#fff
    style S fill:#db2777,stroke:#be185d,color:#fff
    style R fill:#dc2626,stroke:#b91c1c,color:#fff
```

| Gate | Command | What It Checks |
|------|---------|----------------|
| Build | `./gradlew build` | Compiles without errors |
| Tests | `./gradlew test` | All tests pass, no regressions |
| PITest | `./gradlew pitest` | Kill rate is stable (not regressing) |
| Simulation | `./gradlew simulateJava -DsimScenario=SignalCoverage` | All signals publish correctly |
| Robot Deploy | Deploy + physical safety check | Hardware responds correctly |

Each gate catches a different kind of problem. Build catches syntax issues. Tests catch logic bugs. PITest catches holes in the tests. Simulation catches integration problems. The real robot catches the last hardware-specific surprises.
