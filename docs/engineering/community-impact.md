# Community Impact & Open Source

We open-sourced our fire control system in March 2026: three Java files, MIT licensed, drop-in for any FRC team using WPILib. We didn't expect what happened next.

## What We Released

The fire control package has three core files that handle the full pipeline from "where should I aim" to "what does the ball do after I shoot":

| File | Lines | What It Does |
|------|-------|-------------|
| `ShotCalculator` | 623 | Newton-method shoot-on-the-move solver. Warm start from previous cycle means it converges in 1-2 iterations instead of 5-10. Compensates for drag on the ball, robot velocity, and mechanism latency. Outputs RPM, aim angle, time of flight, and a confidence score. |
| `ProjectileSimulator` | 375 | RK4 projectile physics with aerodynamic drag (Cd=0.47, smooth sphere) and Magnus lift from ball spin. Generates a 91-point shooter lookup table from your CAD measurements in about 200ms. |
| `FuelPhysicsSim` | 2,167 | Full-field ball physics for the 2026 REBUILT field. 43 collision elements (bumps, trenches, towers, hubs, nets, walls). Symplectic Euler integration, sequential impulse constraint solver with warm starting, spatial hashing for ball-ball collisions, continuous collision detection for fast projectiles. MIT-licensed so any team can use it. |

We also released `ShotParameters` and `ShotLUT` in v1.1.0 for teams with adjustable hoods.

## Who Adopted It

Within two weeks, we verified 5 teams running our code in competition with our MIT license header in their repositories:

| Team | What They're Known For | What They Adopted | Result |
|------|----------------------|-------------------|--------|
| **2609 BeaverworX** | 2023 World Champion (Einstein), 4x Innovation in Control | Full pipeline (all 3 files) | **Won Innovation in Control** at North Bay, Rank 4, alliance captain |
| **5427 Steel Talons** | 4x FIRST Impact Award, back-to-back Innovation in Control (2024, 2025) | Full pipeline | Won Creativity at McAllen. Competing April 2 with our code. |
| **7461 Sushi Squad** | Innovation in Control (2023), Excellence in Engineering | ShotCalculator + ProjectileSimulator + ShotLUT (custom rewrite) | 8-6-0, deep integration |
| **5561 Raider Robotics** | Innovation in Control (2023), 2x Autonomous Award, District Champion | ShotCalculator | 11-5-0, alliance captain |
| **10584 Ridge Robotics** | Rookie All-Star (2025) | Full pipeline (in lib/frcfirecontrol/) | **Rising All-Star**, Rank 6, alliance captain |

The teams running our code have won Innovation in Control **8 times** in their careers. That includes a World Champion and the back-to-back defending Innovation in Control champions.

## The 2609 Story

This is the one that still gets us. Team 2609 is a 2023 World Champion. Before our code, their shooting system was about 15 lines: calculate the angle to the hub, turn the turret, spin the flywheel at a fixed RPM. No velocity compensation, no time-of-flight calculation, no distance-based RPM.

On March 14, they deleted that code and replaced it with our Newton solver, our RK4 simulator, and our physics engine. They adapted it for their turret robot (converting our chassis-aim output to turret-relative angles), plugged in their CAD measurements (71-degree launch angle, 3-inch wheels, 0.9 slip factor), added zone-based passing targets, and tuned for 5 days.

On March 20, they competed at North Bay. Rank 4. Alliance captain. Won Innovation in Control.

The judges said their robot had "robust simulation modeling, intelligent spatial navigation, and comprehensive data logging." Two of those three things are our code.

## Expert Endorsements

Within 27 minutes of our Chief Delphi post, we got a response from the developer who wrote the WPILib command-based framework, SysId, and SlewRateLimiter. Almost every FRC team uses at least one of those tools. He said:

> "Very cool; one of the more comprehensive implementations I've seen this year. I love your warm start logic and shot quality advisory."

He also gave us practical calibration advice: find a simulator parameter set that matches empirical data for your specific shooter, then when the mechanism changes, update the physical parameters instead of remeasuring the whole table. That's standard practice in industry for complex multifactor problems.

The creator of YAGSL, the swerve drive library that hundreds of FRC teams depend on, wants to port our fire control code into his simulation and shot calculation framework. If that happens, our solver ships to every team using his library.

## Peer Review

A community member from Team 2702 studied our Newton solver math closely enough to find a real bug in the derivative computation. The drag compensation factor wasn't being applied to the TOF derivative in the Newton iteration, which meant the convergence was slightly off when the robot was moving fast. We fixed it the same day and released v1.0.1.

That's the kind of peer review most FRC teams never get. Someone read our math, found a genuine error, and helped us fix it. The code got better because we shared it.

## The Release Cycle

```
Mar 11  v1.0.0 released on Chief Delphi
Mar 11  Expert endorsement within 27 minutes
Mar 12  Peer reviewer finds Newton derivative bug, fixed same day (v1.0.1)
Mar 14  Team 2609 adopts all 3 files
Mar 15  Teams 5427, 7461 adopt
Mar 14-19  Community requests: adjustable hoods, rear-facing shooters,
           backspin support, ShotLUT record, variable-angle LUT,
           unit conversion helpers, custom distance ranges
Mar 20-22  Teams compete with our code, 2609 wins Innovation in Control
Mar 23  v1.1.0 ships all 7 community-requested features
```

Twelve days from release to a World Champion winning an award with our code, with a bug fix and a feature release driven entirely by community feedback in between.

## What We Learned From Sharing

Sharing the code forced us to think about it differently. When the code was just for our robot, we could get away with constants that only made sense in our context. When other teams needed to plug in their own measurements, we had to make the API clean, the configuration obvious, and the physics assumptions explicit.

The community also validated our approach in ways we couldn't do alone. Five different teams, with five different robot architectures, five different shooter geometries, and five different competition environments, all ran our solver and it worked. That's a broader validation than any amount of our own testing could provide.

## By the Numbers

| Metric | Value |
|--------|-------|
| Chief Delphi views | 3,068 in 12 days |
| Likes | 81 (38 on original post) |
| GitHub link clicks | 430 |
| Verified teams with MIT license in code | 5 |
| Awards won by teams using our code (2026) | 3 |
| Innovation in Control career wins among adopters | 8 |
| Community features shipped in v1.1.0 | 7 |
| Time from bug report to fix | Same day |

---

**Related:** [Fire Control Pipeline](../architecture/fire-control-pipeline.md) | [Ball Physics Simulation](fuel-simulation.md) | [Testing & Quality](testing-and-quality.md)
