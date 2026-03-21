# Vision System & Filtering

## What Vision Does

The robot uses AprilTag-based localization to figure out where it is on the field. Cameras see AprilTags (the black-and-white square markers mounted around the field), and PhotonVision on the coprocessor solves for the robot's 3D position relative to those tags. This pose estimate gets fused with wheel odometry and the gyro through a Kalman filter, giving us a field-relative position that's way more accurate than odometry alone.

Without vision, the robot drifts. Wheel slip, carpet variation, and small encoder errors add up over a match. Vision pulls that estimate back toward reality.

The [fire control pipeline](fire-control-pipeline.md) depends on this. ShotCalculator needs to know the robot's position to compute distance to the hub, which determines flywheel RPM and heading. Bad vision data means bad shots.

## Camera Setup

We run 4 cameras configured in `Cameras.java`, each as a PhotonVision camera with its own pose estimator:

| Camera | Name | Position | Orientation | Purpose |
|--------|------|----------|-------------|---------|
| LEFT_CAM | `back-left` | (-0.293, 0.293, 0.229)m | 15 deg down, 135 deg yaw | Rear-left coverage |
| RIGHT_CAM | `back-right` | (-0.293, -0.293, 0.229)m | 15 deg down, -135 deg yaw | Rear-right coverage |
| FRONT_LEFT_CAM | `front-left` | (-0.113, 0.145, 0.483)m | 0 pitch, 35 deg yaw | Front-left coverage |
| FRONT_RIGHT_CAM | `front-right` | (-0.113, -0.145, 0.483)m | 0 pitch, -35 deg yaw | Front-right coverage |

*Front camera positions are from inches (-4.44, +/-5.7, 19) converted to meters. Rounded here for readability.*

Each camera has independent standard deviation baselines: single-tag (0.3, 0.3, 0.6) and multi-tag (0.1, 0.1, 0.2). Multi-tag gets tighter std devs because seeing multiple tags lets PhotonVision triangulate much more precisely. Each camera also has a per-camera std dev factor so you can tune individual cameras up or down if one consistently performs better or worse.

The pose strategy is `MULTI_TAG_PNP_ON_COPROCESSOR` with `LOWEST_AMBIGUITY` as the single-tag fallback.

## VisionFilter: 10-Gate Rejection Pipeline

Not every pose estimate is usable. Reflections, partial tag occlusion, motion blur, and bad PnP solves can all create garbage poses. VisionFilter is a pure-math rejection pipeline that throws those out before they can corrupt the Kalman filter.

Every pose runs through these checks in order. The first failure rejects the pose:

```mermaid
flowchart TB
    RAW[Raw Camera Pose]

    subgraph GATES ["10-Gate Rejection Pipeline"]
        direction TB
        G1[Gate 1: Gyro Rate<br/>Reject if robot spinning too fast]
        G2[Gate 2: Ambiguity<br/>Reject if PnP solver confidence too low]
        G3[Gate 3: Z-Height<br/>Reject if robot appears airborne or underground]
        G4[Gate 4: Roll/Pitch<br/>Reject if tilt exceeds 12 degrees]
        G5[Gate 5: Field Bounds<br/>Reject if pose is outside field walls]
        G6[Gate 6: Heading Divergence<br/>Reject if heading disagrees with gyro]
        G7[Gate 7: Pose Jump<br/>Reject if pose teleports more than 2m]
        G8[Gate 8: Opposing Alliance<br/>Reject single-tag from wrong alliance]
        G9[Gate 9: Staleness<br/>Reject if pose data is too old]
        G10[Gate 10: Distance<br/>Reject single-tag too far away]
        G1 --> G2 --> G3 --> G4 --> G5 --> G6 --> G7 --> G8 --> G9 --> G10
    end

    ACC[Accepted Pose<br/>std devs scaled by distance, speed, tag count]
    KF[Kalman Filter<br/>fuses vision with wheel odometry + gyro]
    SC[ShotCalculator<br/>uses fused position for RPM and heading]

    RAW --> G1
    G10 --> ACC --> KF --> SC

    style RAW fill:#7c3aed,stroke:#5b21b6,color:#fff
    style G1 fill:#dc2626,stroke:#b91c1c,color:#fff
    style G2 fill:#d97706,stroke:#b45309,color:#fff
    style G3 fill:#fbbf24,stroke:#f59e0b,color:#000
    style G4 fill:#059669,stroke:#047857,color:#fff
    style G5 fill:#0891b2,stroke:#0e7490,color:#fff
    style G6 fill:#2563eb,stroke:#1d4ed8,color:#fff
    style G7 fill:#7c3aed,stroke:#5b21b6,color:#fff
    style G8 fill:#f472b6,stroke:#ec4899,color:#fff
    style G9 fill:#9ca3af,stroke:#6b7280,color:#fff
    style G10 fill:#db2777,stroke:#be185d,color:#fff
    style ACC fill:#34d399,stroke:#10b981,color:#000
    style KF fill:#db2777,stroke:#be185d,color:#fff
    style SC fill:#f87171,stroke:#ef4444,color:#fff
```

| Gate | Threshold | What it catches |
|------|-----------|----------------|
| **Gyro Rate** | > 90 deg/s | Robot is spinning too fast for any camera to get a clean frame. Cheapest check, so it runs first. |
| **Ambiguity** | > 0.25 (single-tag only) | PnP solver couldn't confidently distinguish between two possible poses. Multi-tag doesn't have this problem. |
| **Z-Height** | abs(z) > 0.5m | Pose says the robot is flying or underground. Clearly wrong. |
| **Roll/Pitch** | abs(roll or pitch) > 12 deg | Pose says the robot is tilted far beyond what's physically possible on flat carpet. |
| **Field Bounds** | Outside field + 0.5m margin | Pose puts the robot outside the field walls. The margin accounts for bumper overhang at the perimeter. |
| **Heading Divergence** | > 30 deg from gyro (single-tag only) | Single-tag heading is unreliable, so if it disagrees with the gyro by more than 30 degrees, reject it. Skipped while disabled since the gyro may be stale. |
| **Pose Jump** | > 2.0m from current pose | Pose teleports the robot. Skipped during the first 2 seconds of auto (grace period for initial localization). |
| **Opposing Alliance** | Single-tag from wrong alliance | If we're red and we only see 1 tag from the blue side (IDs 1-16), the heading estimate is too unreliable to use. Multi-tag from opposing alliance is fine since triangulation compensates. |
| **Staleness** | Threshold scales with speed | Stale threshold goes from 1.25s when stationary down to 0.3s when driving fast. At high speed, even slightly old data places the robot in the wrong spot. |
| **Distance** | Single-tag > 5.0m away (tunable) | A single tag from far away gives a very noisy pose. Beyond 5 meters we just skip it. |

The gates are ordered cheapest-first so we bail early on obvious junk without running expensive checks.

## Standard Deviation Scaling

Accepted poses don't all get equal trust. The Kalman filter uses standard deviations to weight how much to trust each measurement, and VisionFilter scales these dynamically:

- **Distance scaling**: Std devs grow with the square of the distance to the tag (`1 + dist^2 / 30`). This matches photogrammetry principles: pixel error grows quadratically with range.
- **Velocity scaling**: Std devs grow with the square of robot speed (`1 + speed^2 * 0.3`, capped at 5x). Motion blur and latency displacement degrade accuracy at high speed.
- **Tag count (quadratic)**: More tags = tighter std devs. The scaling is quadratic with tag count (n*n), so going from 1 tag to 3 tags makes a big difference. Multi-tag uses the tighter base std devs (0.1, 0.1, 0.2), single-tag uses the looser ones (0.3, 0.3, 0.6).
- **Per-camera factor**: Each camera can have its own multiplier for std devs, so a camera that consistently gives better results can be trusted more.

So when we are close to a tag, moving slowly, and seeing multiple tags, the filter trusts vision a lot. When we are far away, moving fast, and only see one tag, it leans more on odometry.

## Single-Tag Pose Blending

Single-tag poses are tricky. The translation is usable, but the heading estimate is poor (one tag doesn't give enough geometric constraint). VisionFilter handles this with blending:

- **Heading**: Always uses the fused (gyro-backed) heading. Single-tag heading is set to infinite std dev, so the Kalman filter ignores it.
- **Translation blending**: `computeBlendWeight()` returns a weight based on distance. Close tags (under 3.0m) get blended in using `1 / (1 + dist^2)`. Beyond 3.0m, the weight drops to zero (single-tag translation is too noisy to help).
- **Blended pose**: Interpolates between the current fused position and the single-tag position, then keeps the fused heading.

Multi-tag poses don't need blending. They're used directly with their full std devs.

## Vision Trust State Machine

On top of per-frame rejection, `Vision.java` tracks a higher-level trust state that detects sustained problems like the robot crossing field bumps (which causes rapid roll/pitch rejections):

| State | What's happening |
|-------|-----------------|
| `NORMAL` | Everything's fine, accepting poses normally |
| `TILT_DETECTED` | Multiple roll/pitch rejections in a short window (0.5s). Robot is probably crossing a bump. Vision updates paused until the tilt stops. |
| `RECOVERING` | Tilt stopped, but we wait an extra 0.3s for the robot to settle before trusting vision again. |

This feeds into `ChannelCoordinator` (the AMDA system), which drops to LOW confidence mode and changes the LED strip and haptic intensity to match.

## Alliance Tag Partition

The 2026 field has 32 AprilTags. We partition them by alliance:
- **IDs 1-16**: Blue alliance tags
- **IDs 17-32**: Red alliance tags

When we only see a single tag from the opposing alliance, we reject it entirely (gate 9). The reason is simple: one tag from across the field can give a heading estimate that points the wrong way, and we cannot fix that confidently without more geometry. Multi-tag observations from the opposing side are fine because the solve is much better constrained.

## Diagnosing Vision Issues

When shots are missing or confidence is low, check these signals:

| Signal | What to look for |
|--------|-----------------|
| `Vision/Camera/{name}/Connected` | Camera dropout. Check USB cables, coprocessor health. |
| `Vision/RejectionReason` | Which gate is rejecting most often. If AMBIGUITY is frequent, tags might be partially occluded. If GYRO_RATE, the robot is spinning too much while trying to localize. |
| `Vision/TagCount` | 0 = no tags visible. Check camera aim, field layout loaded correctly. |
| `Vision/AvgDistanceM` | High distance means loose std devs. Get closer to tags before shooting. |
| `Vision/StdDevX`, `Vision/StdDevY` | If these are huge, the filter doesn't trust vision. Check distance and speed. |
| `Vision/PoseJumpM` | Spikes mean intermittent bad solves getting through. Might need to tighten thresholds. |

Common patterns:
- **Lots of AMBIGUITY rejections**: Camera is seeing tags at steep angles. Reposition or tilt camera.
- **HEADING_DIVERGENCE spikes**: Gyro might be drifting, or single-tag heading is fighting a correct gyro. This is normal and the filter is correctly protecting you.
- **POSE_JUMP at match start**: Expected. The 2-second auto grace period exists for this reason.
- **OPPOSING_ALLIANCE rejections**: You're only seeing tags from the other side of the field. Drive toward your own alliance's tags.
- **STALE rejections during fast driving**: Working as intended. The threshold tightens at speed because old data is misleading when the robot has moved significantly since the frame was captured.
- **High std devs during fast driving**: Working as intended. The robot relies more on odometry when moving fast.
