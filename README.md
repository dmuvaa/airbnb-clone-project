# airbnb-clone-project

## Team Roles

| Role            | Owners              | Responsibilities                                  | Backup  |
|-----------------|---------------------|---------------------------------------------------|---------|
| Project Lead    | @Dennis             | Roadmap, priorities, releases, stakeholder comms  | @bob    |

## Technology Stack

- **Django** – Web framework for building the backend and RESTful APIs (routing, auth, validation).
- **PostgreSQL** – Primary relational database for persistent application data.
- **GraphQL** – API query language enabling flexible, typed client–server data fetching.
- **Django REST Framework (DRF)** – Toolkit for building REST APIs on top of Django.
- **Graphene-Django** – GraphQL schema & view integration for Django.
- **Redis** – In-memory cache and task broker.
- **Celery** – Background jobs and scheduled tasks.
- **Docker / Docker Compose** – Reproducible local dev and containerized deployment.
- **Nginx** – Reverse proxy and static asset serving in production.
- **GitHub Actions** – CI/CD for tests and deployments.

## Database Design

### Key Entities & Fields

- **Users**
  - `id` (PK, UUID)
  - `full_name` (string)
  - `email` (string, unique)
  - `role` (enum: `guest`, `host`, `admin`)
  - `created_at` (timestamp)

- **Properties**
  - `id` (PK, UUID)
  - `host_id` (FK → Users.id)
  - `title` (string)
  - `nightly_price` (decimal)
  - `location` (string / JSON with city, country, lat/lng)
  - `created_at` (timestamp)

- **Bookings**
  - `id` (PK, UUID)
  - `property_id` (FK → Properties.id)
  - `guest_id` (FK → Users.id)
  - `check_in` / `check_out` (date)
  - `status` (enum: `pending`, `confirmed`, `cancelled`, `completed`)
  - `total_price` (decimal)

- **Reviews**
  - `id` (PK, UUID)
  - `property_id` (FK → Properties.id)
  - `author_id` (FK → Users.id)
  - `rating` (int 1–5)
  - `comment` (text)
  - `created_at` (timestamp)

- **Payments**
  - `id` (PK, UUID)
  - `booking_id` (FK → Bookings.id)
  - `provider` (string, e.g., Stripe/M-Pesa/etc.)
  - `amount` (decimal), `currency` (string)
  - `status` (enum: `initiated`, `succeeded`, `failed`, `refunded`)
  - `transaction_ref` (string)

### Relationships (high level)

- A **User (host)** can have many **Properties**; a **Property** belongs to one host.
- A **User (guest)** can make many **Bookings**; a **Booking** belongs to one guest and one property.
- A **Property** can have many **Reviews**; a **Review** belongs to one property and one author (user).
- A **Booking** has one or more **Payments** (initial charge, refunds/partials).
- Common extensions (optional): `PropertyImages`, `Amenities` (many-to-many), `Favorites` (User ↔ Property), `Messages` (guest ↔ host).

---

## Feature Breakdown

- **User Management (Auth & Profiles)**  
  Users can sign up/in, manage profiles, and assume roles (guest/host). Session or token auth secures access; profile data powers bookings and hosting features.

- **Property Management (Hosting)**  
  Hosts can create, edit, and publish listings with pricing, availability, photos, and amenities. Validation and draft/publish states ensure quality listings.

- **Search & Discovery**  
  Guests can filter by location, dates, price, capacity, and amenities. Sorting, pagination, and map/radius search improve findability and performance.

- **Booking System**  
  Real-time availability checks, date validation, and price calculation (incl. fees/taxes). Bookings move through states (pending → confirmed → completed/cancelled) with clear user feedback.

- **Payments**  
  Secure payment flows tied to bookings, with idempotent charge creation and webhook handling for asynchronous confirmations/refunds. Clear receipts and error handling reduce disputes.

- **Reviews & Ratings**  
  Guests can rate and review properties after completed stays. Anti-spam rules and optional “review must match a completed booking” keep feedback trustworthy.

- **Notifications**  
  Email/in-app notifications for booking requests, confirmations, cancellations, and review reminders. Batched or real-time delivery to avoid noise.

- **Admin Console (Moderation & Support)**  
  Admins can view users, listings, bookings, and payments; they can moderate content and assist with disputes, refunds, and account issues.

---

## API Security

- **Authentication**  
  Use secure session or JWT/OAuth flows, strong password policies, and MFA where appropriate. Purpose: verify identity to protect accounts and prevent unauthorized actions.

- **Authorization (RBAC + Record-level)**  
  Enforce role-based and ownership checks (e.g., only a host can edit their listing; only a guest can cancel their booking). Purpose: least privilege and data isolation.

- **Input Validation & Sanitization**  
  Validate all payloads (types, ranges, enums) and sanitize user content to prevent injection/XSS. Purpose: preserve data integrity and app stability.

- **Rate Limiting & Throttling**  
  Limit login, booking, and payment endpoints to mitigate brute-force and abuse. Purpose: protect availability and reduce fraud.

- **Transport & Data Encryption**  
  Enforce HTTPS/TLS; encrypt sensitive data at rest (e.g., tokens, secrets). Purpose: protect user data and prevent interception.

- **Secrets Management**  
  Store API keys and credentials in a secure vault/manager; rotate regularly. Purpose: reduce blast radius from leaks.

- **Audit Logging**  
  Log auth events, admin actions, and payment lifecycle events. Purpose: traceability for debugging, compliance, and fraud investigation.

- **Payments Hardening**  
  Use provider webhooks with signature verification and idempotency keys; never store raw card/PIN data. Purpose: secure money flows and prevent duplicate charges.

- **CORS/CSRF**  
  Configure CORS carefully for API consumers; apply CSRF protections to cookie-based sessions. Purpose: protect against cross-origin attacks.

---

## CI/CD Pipeline

**What & Why:**  
Continuous Integration (CI) automatically tests and validates each change (lint, unit/integration tests, type checks). Continuous Delivery/Deployment (CD) builds artifacts (e.g., Docker images), runs migrations, and deploys to staging/production with health checks and rollbacks. This shortens feedback loops, improves quality, and reduces deployment risk.

**Tools (typical setup):**  
- **GitHub Actions** for pipelines (on PRs and merges to `main`).  
- **Docker** to build consistent images; **Docker Compose** for local parity.  
- Test & quality gates (e.g., `pytest`/`jest`, linters, formatters, type checks).  
- Deploy to your target (e.g., Fly.io, Render, AWS, Railway, or self-hosted), run DB migrations, then smoke tests.
