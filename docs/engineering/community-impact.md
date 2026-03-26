# Community Impact & Open Source

We posted three Java files on Chief Delphi. MIT licensed, drop-in, just needs WPILib.

Twelve days later, a World Champion had integrated our solver, won Innovation in Control, and judges were citing our physics engine by name. We genuinely did not expect that.

> "Very cool; one of the more comprehensive implementations I've seen this year. I love your warm start logic and shot quality advisory."
>
> *The developer behind the command-based framework, SysId, SlewRateLimiter, and other core WPILib tools, responding 27 minutes after our post.*

## The Numbers

| | |
|---|---|
| **9** | verified teams running our code with our MIT license in their repos (16 total adopters) |
| **8** | combined Innovation in Control wins across those teams' careers |
| **5** | awards won in 2026 by teams using our code, across 4 categories |
| **3,300** | Chief Delphi views |
| **6 days** | from a World Champion integrating our code to winning Innovation in Control |

## Who Is Using It

| Team | Career Highlights | What They Adopted | 2026 With Our Code |
|------|-------------------|-------------------|--------------------|
| **2609 BeaverworX** | 2023 World Champion (Einstein), 4x Innovation in Control | All 3 files | **Won Innovation in Control**, Rank 4, alliance captain |
| **5427 Steel Talons** | 4x FIRST Impact, back-to-back Innovation in Control (2024, 2025) | All 3 files | Won Creativity. Competing April 2 with our code. |
| **7461 Sushi Squad** | Innovation in Control (2023), Excellence in Engineering | ShotCalc + ProjectileSim + ShotLUT | 8-6-0, deep customization |
| **5561 Raider Robotics** | Innovation in Control (2023), 2x Autonomous, District Champion | ShotCalculator | 11-5-0, alliance captain |
| **10584 Ridge Robotics** | Rookie All-Star (2025) | All 3 files | **Rising All-Star**, Rank 6, alliance captain |

The back-to-back defending Innovation in Control champions (2024 and 2025) adopted our pipeline for their next event. We're still processing that one.

## The 2609 Story

Team 2609 is a 2023 World Champion. They had a working turret shooter. On March 14, they integrated our fire control pipeline to add velocity compensation, drag-corrected time-of-flight, and distance-based RPM from the Newton solver.

They converted our chassis-aim output to turret-relative angles for their independent turret, plugged in their CAD measurements (71-degree launch angle, 3-inch wheels, 0.9 slip factor), added zone-based passing targets, and tuned for 6 days.

On March 20, they competed at North Bay. Rank 4. Alliance captain. Won Innovation in Control.

The judges said:

> "Their robot stood out for its robust simulation modeling, intelligent spatial navigation, and comprehensive data logging."

The simulation modeling is our `ProjectileSimulator` and `FuelPhysicsSim`. The spatial navigation is our `ShotCalculator`. Two out of three of those are us.

## Peer Review

Someone from Team 2702 went through our Newton solver math and found a real bug. The drag compensation factor wasn't being applied to the TOF derivative, so convergence was slightly off when the robot was moving fast. We fixed it the same day.

That kind of feedback is why we shared the code. One person reading our math carefully caught something our 800+ tests missed.

## What We Released

Three files, MIT licensed, only needs wpimath and ntcore (already in GradleRIO):

| File | Lines | What It Does |
|------|-------|-------------|
| `ShotCalculator` | 623 | Newton-method shoot-on-the-move solver. Warm start convergence in 1-2 iterations. Drag compensation, second-order pose prediction, 5-component confidence scoring. |
| `ProjectileSimulator` | 375 | RK4 projectile physics with drag (Cd=0.47) and Magnus lift. Generates 91-point shooter LUTs from CAD measurements in ~200ms. |
| `FuelPhysicsSim` | 2,285 | Full-field ball physics: 43 collision elements, spatial hashing, ball sleeping, CCD for fast projectiles, symplectic Euler with sequential impulse solver. |

v1.1.0 added `ShotParameters` and `ShotLUT` for teams with adjustable hoods, plus rear-facing shooter support, backspin, variable-angle LUT generation, unit conversion helpers, and custom distance ranges. All seven came from community requests.

## What We Learned

When the code was just for our robot, we could get away with constants that only made sense in our context. Once other teams needed to plug in their own measurements, we had to actually think about the API and document our physics assumptions properly. That made the code better even for us.

Sixteen teams, sixteen different robots, sixteen different shooter setups, multiple competitions. The solver worked on all of them. We couldn't have tested that range on our own.

Team 5427 competes with our code on April 2. We'll be watching.

---

**Related:** [Fire Control Pipeline](../architecture/fire-control-pipeline.md) | [Ball Physics Simulation](fuel-simulation.md) | [Testing & Quality](testing-and-quality.md)
