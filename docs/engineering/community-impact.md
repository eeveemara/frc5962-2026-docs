# Community Impact & Open Source

We posted three Java files on Chief Delphi. MIT licensed, drop-in, just needs WPILib.

Fifteen days later, sixteen teams were running our code, a World Champion had won Innovation in Control with our solver, and two of the most respected developers in FRC had endorsed it.

> "Very cool; one of the more comprehensive implementations I've seen this year. I love your warm start logic and shot quality advisory."
>
> *Eli Barnett (Oblarg), developer of the command-based framework, SysId, SlewRateLimiter, and other core WPILib tools. Responded 27 minutes after our post.*

> "It's impressive!! I love how you combined everything. It is intuitive."
>
> *nstrike, creator of YAGSL, the swerve drive library used by hundreds of FRC teams. Asked us to build a fire control example for YAMS after competition.*

## The Numbers

| | |
|---|---|
| **16** | teams running our code (9 verified with MIT license in their repos, 7 reported on Chief Delphi) |
| **5** | award-winning teams are running our code (across 4 different award categories) |
| **8** | combined Innovation in Control wins across the verified adopters' careers |
| **3,300** | Chief Delphi views (87 likes) |
| **1,300+** | GitHub views, 60+ cloners |
| **5 days** | from a World Champion integrating our code to winning Innovation in Control |

## Who Is Using It

### 9 Verified Adopters (MIT License in Code)

| Team | What They Adopted | 2026 Result |
|------|-------------------|-------------|
| **2609 BeaverworX** | All 3 files | **Won Innovation in Control**, 2023 World Champion, 4x Innovation winner |
| **5427 Steel Talons** | All 3 files | **Won Creativity**, back-to-back Innovation in Control (2024-25) |
| **10584 Pennridge Robotics** | All 3 files (lib/) | **Won Rising All-Star**, Rank 6, alliance captain |
| **5010 Tiger Dynasty** | Full clone + variable-angle extension | **Event Finalist** |
| **7461 Sushi Squad** | ShotCalc + ProjectileSim + ShotLUT | Deep customization |
| **5561 Raider Robotics** | ShotCalculator | Alliance captain |
| **2903 Neobots** | All 3 files (frc.firecontrol package) | |
| **7160 O-Bots** | ShotCalc (adapted for turret) | |
| **Juggernauts** | ShotCalc + ProjectileSim | Founding-era team, replaced their own working system |

### 7 More Reported on Chief Delphi

852 ARC Robotics (**Team Spirit Award**), 5113 Combustible Lemons (**Sustainability + Team Spirit**), 4322 Clockwork, 3566 Gone Fishin', 3211 The Y Team, 6619 GravitechX, 2022 Titan Robotics.

### Award-Winning Teams Running Our Code

| Award | Team |
|-------|------|
| **Innovation in Control** | 2609 BeaverworX |
| **Creativity** | 5427 Steel Talons |
| **Rising All-Star** | 10584 Pennridge Robotics |
| **Sustainability** | 5113 Combustible Lemons |
| **Team Spirit** | 852 ARC Robotics, 5113 Combustible Lemons |

## The Stories

### 2609 BeaverworX: World Champion Wins Innovation with Our Code

Team 2609 is a 2023 World Champion. They had a working turret shooter. On March 14, they integrated our fire control pipeline to add velocity compensation, drag-corrected time-of-flight, and distance-based RPM from the Newton solver.

They converted our chassis-aim output to turret-relative angles for their independent turret, plugged in their CAD measurements (71-degree launch angle, 3-inch wheels, 0.9 slip factor), and added zone-based passing targets.

On March 20, they competed at North Bay. Rank 4. Alliance captain. Won Innovation in Control.

The judges said:

> "Their robot stood out for its robust simulation modeling, intelligent spatial navigation, and comprehensive data logging."

The simulation modeling is our `ProjectileSimulator` and `FuelPhysicsSim`. The spatial navigation is our `ShotCalculator`. Two out of three of those are us.

### The Juggernauts: 30-Year Team Replaced Their Own System

A founding-era FRC team with about 30 years of history had their own shot calculator and velocity compensator. They found our code, adopted the Newton solver and projectile simulator, renamed the files to fit their naming convention, and kept their old system alongside for reference. Their commit history shows the before and after. Proper MIT attribution throughout.

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

---

**Related:** [Fire Control Pipeline](../architecture/fire-control-pipeline.md) | [Ball Physics Simulation](fuel-simulation.md) | [Testing & Quality](testing-and-quality.md)
