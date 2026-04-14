# Community Impact & Open Source

We posted three Java files on Chief Delphi. MIT licensed, drop-in, just needs WPILib.

Five weeks later, twenty-three teams were running our code, four of them had won Innovation in Control with our solver in production, and two of the most respected developers in FRC had endorsed it.

> "Very cool; one of the more comprehensive implementations I've seen this year. I love your warm start logic and shot quality advisory."
>
> *Eli Barnett (Oblarg), developer of the command-based framework, SysId, SlewRateLimiter, and other core WPILib tools. Responded 27 minutes after our post.*

> "It's impressive!! I love how you combined everything. It is intuitive."
>
> *nstrike, creator of YAGSL, the swerve drive library used by hundreds of FRC teams. Asked us to build a fire control example for YAMS after competition.*

## The Numbers

| | |
|---|---|
| **23** | verified adopter teams across 11 US states and 3 countries |
| **3,700** | Chief Delphi views (87 likes) |
| **1,300+** | GitHub views, 60+ cloners |

## Who Is Using It

All 23 adopters. 

| Team | Name | Region | Awards (since Mar 6) |
|------|------|--------|----------------------|
| 571 | Paragon Robotics | Connecticut | District Event Winner |
| 4322 | Clockwork | California | Innovation in Control + Event Finalist |
| 4512 | Otter Chaos | Washington | Leadership Semi-Finalist |
| 8048 | ChurroBots | California | Event Finalist + Team Spirit |
| 5010 | Tiger Dynasty | Indiana | Event Finalist + Engineering Inspiration |
| 5572 | Rosbots | Texas | Creativity + Quality + Event Finalist |
| 8032 | SAASquatch | Washington | none yet |
| 2609 | Beaverworx | Ontario | Innovation in Control x2 (2023 World Champion) |
| 6619 | GravitechX | California | none yet |
| 5427 | Steel Talons | Texas | FIRST Impact + Creativity |
| 7160 | O-Bots | Michigan | District Event Winner + Judges' Award x2 |
| 5561 | Raider Robotics | Michigan | Sustainability |
| 7461 | Sushi Squad | Washington | Leadership Semi-Finalist |
| 3566 | Gone Fishin' | Massachusetts | NE DCMP qualifier |
| 3211 | The Y Team | Israel | none yet |
| 10584 | Ridge Robotics | Pennsylvania | Rising All-Star x2 |
| 5113 | Combustible Lemons | New Jersey | Sustainability + Team Spirit |
| 852 | ARC Robotics | California | Team Spirit + Imagery |
| 5655 | KelRot | Turkey | none yet |
| 2022 | Titan Robotics | Illinois | Leadership Finalist + Sustainability |
| 2903 | Neobots | Washington | none yet |
| 6004 | f(x) Robotics | North Carolina | Excellence in Engineering x2 + FIRST Impact + Leadership x2 + DCMP Finalist |
| 1 | Juggernauts | Michigan | Creativity |



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

Twenty-three teams, twenty-three different robots, twenty-three different shooter setups, multiple competitions. The solver worked on all of them. We couldn't have tested that range on our own.

---

**Related:** [Fire Control Pipeline](../architecture/fire-control-pipeline.md) | [Ball Physics Simulation](fuel-simulation.md) | [Testing & Quality](testing-and-quality.md)
