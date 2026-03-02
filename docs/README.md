<p align="center">
  <img src="team_logo.png" alt="Team 5962 perSEVERE" width="150">
</p>

# Team 5962 perSEVERE | Control System Documentation

> *The robot assesses. The copilot fires. The driver flies.*

Welcome to our control system docs for the 2026 FRC season (game: REBUILT). This covers everything from our telemetry and fire control pipeline to the feedback systems that keep our drivers informed without ever needing to glance at a screen.

## Site Map

```mermaid
graph TD
    HOME[Documentation Home] --> ARCH[Architecture]
    HOME --> FB[Driver Feedback]
    HOME --> DASH[Dashboards]
    HOME --> OPS[Operations]
    HOME --> ENG[Engineering]
    ARCH --> A1[System Overview]
    ARCH --> A2[Telemetry System]
    ARCH --> A3[Fire Control Pipeline]
    ARCH --> A4[Vision System]
    ARCH --> A5[Safety Architecture]
    FB --> F1[AMDA Feedback System]
    FB --> F2[Alliance Strategy]
    FB --> F3[Universal Design]
    DASH --> D1[Elastic Guide]
    DASH --> D2[AdvantageScope Guide]
    DASH --> D3[Quick Reference]
    OPS --> O1[Competition Playbook]
    OPS --> O2[Tuning Reference]
    OPS --> O3[Troubleshooting]
    ENG --> E1[Testing & Quality]
    ENG --> E2[Engineering Process]
    ENG --> E3[FMEA Log]
    ENG --> E4[Ball Physics Simulation]
    ENG --> E5[What We Learned]

    style HOME fill:#7c3aed,stroke:#5b21b6,color:#fff
    style ARCH fill:#2563eb,stroke:#1d4ed8,color:#fff
    style FB fill:#059669,stroke:#047857,color:#fff
    style DASH fill:#d97706,stroke:#b45309,color:#fff
    style OPS fill:#dc2626,stroke:#b91c1c,color:#fff
    style ENG fill:#db2777,stroke:#be185d,color:#fff
    style A1 fill:#60a5fa,stroke:#3b82f6,color:#fff
    style A2 fill:#60a5fa,stroke:#3b82f6,color:#fff
    style A3 fill:#60a5fa,stroke:#3b82f6,color:#fff
    style A4 fill:#60a5fa,stroke:#3b82f6,color:#fff
    style A5 fill:#60a5fa,stroke:#3b82f6,color:#fff
    style F1 fill:#34d399,stroke:#10b981,color:#000
    style F2 fill:#34d399,stroke:#10b981,color:#000
    style F3 fill:#34d399,stroke:#10b981,color:#000
    style D1 fill:#fbbf24,stroke:#f59e0b,color:#000
    style D2 fill:#fbbf24,stroke:#f59e0b,color:#000
    style D3 fill:#fbbf24,stroke:#f59e0b,color:#000
    style O1 fill:#f87171,stroke:#ef4444,color:#fff
    style O2 fill:#f87171,stroke:#ef4444,color:#fff
    style O3 fill:#f87171,stroke:#ef4444,color:#fff
    style E1 fill:#f472b6,stroke:#ec4899,color:#fff
    style E2 fill:#f472b6,stroke:#ec4899,color:#fff
    style E3 fill:#f472b6,stroke:#ec4899,color:#fff
    style E4 fill:#f472b6,stroke:#ec4899,color:#fff
    style E5 fill:#f472b6,stroke:#ec4899,color:#fff
```

## By the Numbers

| Metric | Value |
|--------|-------|
| Telemetry signals monitored in real time | ~500 |
| Mutation testing kill rate | 53% across 10 classes |
| FMEA failure entries tracked | 34 |
| Feedback channels (haptic, LED, HUD, dashboard) | 4 |
| Conditions for automated scoring readiness | 6 |
| Neural network ensemble models for shot prediction | 10 |
| Collision elements in physics simulation | 43 |
| Custom code linter rules | 108 |

## Quick Start by Role

**Drivers & Copilots** \
[Driver Feedback](feedback/driver-feedback.md) | [Competition Playbook](operations/competition-playbook.md) | [Alliance Strategy](feedback/alliance-strategy.md)

**Programmers** \
[System Overview](architecture/system-overview.md) | [Telemetry System](architecture/telemetry-system.md) | [Testing & Quality](engineering/testing-and-quality.md) | [Ball Physics](engineering/fuel-simulation.md)

**Coaches** \
[Competition Playbook](operations/competition-playbook.md) | [Alliance Strategy](feedback/alliance-strategy.md) | [Quick Reference](dashboards/quick-reference.md)

**Judges & Mentors** \
[System Overview](architecture/system-overview.md) | [Fire Control](architecture/fire-control-pipeline.md) | [FMEA Log](engineering/fmea-log.md) | [Engineering Process](engineering/engineering-process.md) | [Universal Design](feedback/universal-design.md) | [What We Learned](engineering/what-we-learned.md)

**Debugging** \
[Troubleshooting](operations/troubleshooting.md) | [Elastic Guide](dashboards/elastic-guide.md) | [AdvantageScope Guide](dashboards/advantagescope-guide.md)

## System Architecture

```mermaid
graph LR
    subgraph Robot
        SUB[Subsystems] --> TM[TelemetryManager]
        TM --> SL[SafeLog]
    end

    SL --> AK[AdvantageKit Logger]
    AK --> NT[NetworkTables]
    AK --> LOG[Log Files]

    NT --> DASH[Dashboards]
    NT --> CC[ChannelCoordinator]

    CC --> HAP[Haptic Feedback]
    CC --> LED_OUT[LED Status]
    CC --> HUD[Camera HUD]
    CC --> DASH

    style SUB fill:#7c3aed,stroke:#5b21b6,color:#fff
    style TM fill:#2563eb,stroke:#1d4ed8,color:#fff
    style SL fill:#059669,stroke:#047857,color:#fff
    style AK fill:#0891b2,stroke:#0e7490,color:#fff
    style NT fill:#d97706,stroke:#b45309,color:#fff
    style LOG fill:#d97706,stroke:#b45309,color:#fff
    style DASH fill:#dc2626,stroke:#b91c1c,color:#fff
    style CC fill:#db2777,stroke:#be185d,color:#fff
    style HAP fill:#f472b6,stroke:#ec4899,color:#fff
    style LED_OUT fill:#34d399,stroke:#10b981,color:#000
    style HUD fill:#fbbf24,stroke:#f59e0b,color:#000
```

---

<sub>FRC Team 5962 perSEVERE, 2026 Season</sub>
