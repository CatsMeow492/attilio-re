# Technical Reverse Engineering Analysis

**Target Artifact**: `Attilio.debug.dylib`
**Architecture**: ARM64 (iOS Simulator)

## Core Business Logic Mapping (`Attilio.Model`)

The following methods define the optimization and access loop:

| Method | Binary Address | Description |
| :--- | :--- | :--- |
| `load()` | `0x3a978` | Synchronizes state; applies life-stage filtering to event visibility. |
| `book()` | `0x3ff38` | Handles slot reservation based on access/credit validation. |
| `claim()` | `0x409f4` | Finalizes event acquisition and triggers credit deduction. |

## The "Optimization" Workflow

The app optimizes life stages through a configuration-driven pipeline:

1. **Identity Resolution**: The system identifies the user's current Life Stage (e.g., Starter, Advanced).
2. **Configuration Loading**: An `Attilio.Config` object is loaded with parameters specific to that stage (e.g., `cancelCutoffHours`).
3. **Event Filtering**: `Model.load()` filters the available event universe based on the loaded config, creating a personalized "claimable" list for the user's dashboard.
4. **Access Validation**: Actions (`book`, `claim`) are gated by `Attilio.Access`, ensuring that only users with sufficient credits or correct stage-permissions can execute optimization actions.

## Life Stage Personalization Logic

Personalization is not hardcoded but is instead data-driven via the `Attilio.Config` and `Attilio.Access` classes:
- **Dynamic Rebalancing**: The app monitors triggers (e.g., physiological markers) to suggest or force shifts in life stage configurations.
- **Strictness Scaling**: Higher life stages typically grant wider booking windows and access to a broader set of optimization events.
