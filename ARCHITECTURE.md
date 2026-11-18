# System Reconstruction Architecture

This document translates the reverse engineering findings from `Attilio.debug.dylib` into a scalable, modern technical implementation plan.

## 1. Frontend: Component Library & UI/UX
The frontend is designed as a state-driven interface where visibility and interaction are governed by the user's current "Life Stage."

### Design Language
- **Theme**: Dynamic theming (e.g., color palettes that shift based on Life Stage).
- **Aesthetic**: High-density utility interfaces with a focus on "Glassmorphism" to emphasize a premium, optimized feel.

### Core Components
| Component | Responsibility | Logic Trigger |
| :--- | :--- | :--- |
| `StageIndicator` | Displays current Life Stage and credit balance. | `UserSession.stage` |
| `EventCard` | Represents an optimization event; state-aware (Claimable $\rightarrow$ Booked $\rightarrow$ Active). | `Model.load()` filtered list |
| `PersonalizationDashboard`| Grid of available optimization tiles tailored to the user's stage. | `Config.filter_criteria` |
| `AccessGate` | Intercepts actions that exceed current stage permissions; triggers "Level Up" flows. | `Access.validate()` failure |

---

## 2. Backend Architecture: The Personalization Engine
A modular architecture designed to handle the tight loop between identity, configuration, and event visibility.

### Core Services
- **Stage Manager**: Maps the user to a specific `LifeStage` and provides the corresponding `Attilio.Config`.
- **Event Engine**: Implements the filtering logic observed in `Model.load()`, intersecting available events with stage constraints.
- **Access Controller**: A transactional service managing credits and bookings (`book()` and `claim()` logic).

### Data Flow
`Request` $\rightarrow$ `API Gateway` $\rightarrow$ `Stage Manager (Redis Cache)` $\rightarrow$ `Event Engine (SQL Filter)` $\rightarrow$ `Personalized Response`.

---

## 3. Database Schema
A relational schema designed to ensure strict stage-based access control.

### Primary Tables
- **`users`**: `id`, `email`, `current_stage_id`, `credit_balance`.
- **`life_stages`**: `id`, `label`, `priority_level`, `config_blob (JSONB)`. (Stores stage-specific params like `cancelCutoffHours`).
- **`events`**: `id`, `title`, `min_stage_required`, `total_slots`.
- **`bookings`**: `id`, `user_id`, `event_id`, `status (reserved/claimed)`.

---

## 4. Deployment & Operations

### Infrastructure Stack
- **Frontend**: Vercel (Web) / Native iOS.
- **Backend**: AWS Fargate or GCP Cloud Run (Auto-scaling containers).
- **Database**: Managed PostgreSQL with a Redis layer for caching `LifeStage` configurations.

### Scaling & Monitoring
- **Scaling**: Horizontal Pod Autoscaling (HPA) based on request volume and CPU usage.
- **Monitoring**: 
    - **Prometheus/Grafana**: Endpoint latency and error rates.
    - **OpenTelemetry**: Distributed tracing to monitor the latency of the personalization pipeline.
    - **Business Metrics**: Tracking "Stage Migration" (Starter $\rightarrow$ Advanced) as a primary success KPI.
