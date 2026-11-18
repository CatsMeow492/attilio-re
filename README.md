# Attilio Reverse Engineering

This repository documents the reverse engineering findings for **Attilio**, a life optimization app.

## High-Level Findings
The core "optimization" logic in Attilio is implemented as an **Access-as-a-Service** model. The application leverages user "Life Stages" to personalize experience and filter available services.

### Key Architectural Insights:
- **Personalization**: User identity determines a "Life Stage," which loads a specific `Attilio.Config`.
- **Filtering**: The `Attilio.Model` uses this configuration to filter the global pool of events into a personalized list of claimable items.
- **Access Control**: Reservations and claims are validated against the user's current access credits and stage permissions.

For detailed technical analysis, including binary addresses and method logic, see [REVERSE_ENGINEERING.md](./REVERSE_ENGINEERING.md).
For the proposed system reconstruction plan, see [ARCHITECTURE.md](./ARCHITECTURE.md).

