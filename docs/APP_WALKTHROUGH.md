# KCSP Platform — Comprehensive Application Walkthrough & Screenshots Guide

A visual, step-by-step tour through the enterprise web console, execution pipeline, and simulation endpoints of the **KCSP (WhatsApp Security Awareness Simulation Platform)**.

---

## 🧭 Application Map & Walkthrough Navigation

| # | Screen / Module | Primary Purpose | Live URL Path | Screenshot |
| :--- | :--- | :--- | :--- | :--- |
| **00** | **Authentication Portal** | Role-based administrator login with bcrypt verification | `/login` | `screenshots/00_login.png` |
| **01** | **Executive Dashboard** | High-level resilience metrics, session health, queue status | `/dashboard` | `screenshots/01_dashboard.png` |
| **02** | **Campaigns Manager** | Overview of all active, scheduled, and completed drills | `/campaigns` | `screenshots/02_campaigns.png` |
| **03** | **Campaign Detail & Drill** | Granular conversion funnel, unique clicks, department stats | `/campaigns/[id]` | `screenshots/03_campaign_detail.png` |
| **04** | **Participant Roster** | Department tagging, CSV import, 1-click GDPR anonymization | `/participants` | `screenshots/04_participants.png` |
| **05** | **Message Templates** | Dynamic WhatsApp templates with variable interpolation | `/templates` | `screenshots/05_templates.png` |
| **06** | **Landing Scenarios** | Controlled simulation pages (Microsoft 365, HR policy) | `/landing-pages` | `screenshots/06_landing_pages.png` |
| **07** | **Dispatch Job Queue** | Asynchronous queue with rate limiting, retries, and jitter | `/queue` | `screenshots/07_queue.png` |
| **08** | **Analytics & Risk Heatmaps** | Cross-campaign telemetry, department vulnerability, CSV export | `/analytics` | `screenshots/08_analytics.png` |
| **09** | **Simulation Landing Page** | Pixel-perfect Microsoft 365 SSO with client DOM interception | `/s/[token]` | `screenshots/09_simulation_landing.png` |
| **10** | **The Teachable Moment** | Instant non-punitive educational debrief card | `/s/[token]` | `screenshots/10_simulation_debrief.png` |

---

## 00. Administrator Authentication Portal
- **Route**: `http://localhost:3000/login`
- **File**: `screenshots/00_login.png`

The access gateway enforces bcrypt password hashing and secure, hardened session cookies. Pre-seeded credentials allow rapid evaluation for internal auditors and security teams.

![Admin Authentication Portal](screenshots/00_login.png)

#### Key Capabilities:
- Role-based authorization (`ADMIN`, `OPERATOR`, `AUDITOR`).
- Automatic logging of `USER_LOGIN` events in the immutable audit ledger.
- Clean, focused interface with security status badges.

---

## 01. Executive Resilience Dashboard
- **Route**: `http://localhost:3000/dashboard`
- **File**: `screenshots/01_dashboard.png`

The central cockpit for security officers and CISOs. Combines live campaign status, human resilience scores, WhatsApp gateway connectivity, and recent audit logs into a unified view.

![Executive Resilience Dashboard](screenshots/01_dashboard.png)

#### Key Capabilities:
- **Resilience Score Meter**: Calculates `100 - Interaction Rate` in real time.
- **WhatsApp Gateway Health**: Probes WASender and WPPConnect connections; displays warning banner if a gateway disconnects.
- **Outbound Queue Health**: Displays pending, processing, and paused jobs.
- **Immutable Audit Feed**: Shows real-time operator actions with timestamps and resource IDs.

---

## 02. Simulation Campaigns Management
- **Route**: `http://localhost:3000/campaigns`
- **File**: `screenshots/02_campaigns.png`

Manages the entire lifecycle of awareness drills across departments.

![Simulation Campaigns Management](screenshots/02_campaigns.png)

#### Key Capabilities:
- Filter campaigns by status (`DRAFT`, `SCHEDULED`, `RUNNING`, `COMPLETED`).
- Quick-launch modal for creating targeted drills with customized templates and scenarios.
- Participant counter, message delivery ratios, and creation timestamps.

---

## 03. Campaign Detail Telemetry & Conversion Funnel
- **Route**: `http://localhost:3000/campaigns/[id]`
- **File**: `screenshots/03_campaign_detail.png`

Provides deep forensic analytics for an individual drill, following participants through each phase of the attack lifecycle.

![Detailed Campaign Telemetry](screenshots/03_campaign_detail.png)

#### Key Capabilities:
- **Conversion Funnel**: Sent → Delivered → Link Opened → Simulation Interaction.
- **Department Vulnerability Breakdown**: e.g., Engineering vs. Finance risk comparisons.
- **Live Test Preview**: Direct link to test the participant's opaque URL and controlled simulation landing page.

---

## 04. Participant Roster & GDPR Privacy Actions
- **Route**: `http://localhost:3000/participants`
- **File**: `screenshots/04_participants.png`

Directory of enrolled employees segmented by department, providing full compliance with European privacy regulations.

![Participant Roster & GDPR Actions](screenshots/04_participants.png)

#### Key Capabilities:
- **Department Segmentation**: Easily target high-risk units (Finance, HR, Executive, IT).
- **1-Click GDPR Anonymization**: Instantly scrubs employee name, phone number, and email upon request while retaining historical statistical risk scores.
- **CSV Roster Import**: Batch upload employee rosters with phone number validation.

---

## 05. Dynamic WhatsApp Message Templates
- **Route**: `http://localhost:3000/templates`
- **File**: `screenshots/05_templates.png`

Library of social engineering simulation templates mimicking real-world attacker techniques.

![Message Templates](screenshots/05_templates.png)

#### Key Capabilities:
- **Dynamic Variable Interpolation**: Smart placeholders: `{{first_name}}`, `{{department}}`, `{{tracking_url}}`.
- **Pre-Built Scenarios**:
  1. *IT Security Urgent Access Alert*
  2. *HR Welfare & Policy Acknowledgment*
  3. *Administration Payroll Verification*
- Extensible custom template builder.

---

## 06. Simulation Landing Scenarios
- **Route**: `http://localhost:3000/landing-pages`
- **File**: `screenshots/06_landing_pages.png`

Pre-configured, high-fidelity landing pages designed to simulate realistic corporate portals safely.

![Simulation Landing Scenarios](screenshots/06_landing_pages.png)

#### Key Capabilities:
- Microsoft 365 Single Sign-On simulation.
- Corporate HR Policy Acknowledgment portal.
- Custom HTML/CSS landing scenario editor with live syntax preview.

---

## 07. Outbound Dispatch Job Queue
- **Route**: `http://localhost:3000/queue`
- **File**: `screenshots/07_queue.png`

The heartbeat of the message execution engine, ensuring high deliverability while protecting WhatsApp phone numbers from spam flagging.

![Outbound Dispatch Job Queue](screenshots/07_queue.png)

#### Key Capabilities:
- **Rate-Limiting & Jitter**: Randomized delays between dispatches to mimic human behavior.
- **Automated Fail-Safe Pause**: Pauses outbound jobs if the WhatsApp gateway reports a logged-out state.
- **Exponential Backoff**: Automatic retry scheduling on transient network errors.
- **Cryptographic Idempotency**: Strict database constraints prevent duplicate messages to any recipient.

---

## 08. Advanced Analytics & Risk Heatmaps
- **Route**: `http://localhost:3000/analytics`
- **File**: `screenshots/08_analytics.png`

Executive-level intelligence transforming raw interaction telemetry into boardroom-ready risk reports.

![Advanced Analytics & Risk Heatmaps](screenshots/08_analytics.png)

#### Key Capabilities:
- **Department Vulnerability Heatmap**: Identifies which business units require prioritized security training.
- **Unique vs. Repeated Click Telemetry**: Prevents single-user repeated clicks from skewing unique vulnerability metrics.
- **Dual CSV Export**:
  - *Full Administrative CSV*: For security analysts and internal audits.
  - *Anonymized CSV*: PII-free statistical report for board presentations.

---

## 09. Controlled Simulation Landing Page (Microsoft 365)
- **Route**: `http://localhost:3000/s/[token]`
- **File**: `screenshots/09_simulation_landing.png`

A pixel-perfect Microsoft 365 Single Sign-On portal where the **Zero-Credential Safe Simulation Engine** operates.

![Controlled Simulation Landing Page](screenshots/09_simulation_landing.png)

#### Key Capabilities:
- **Client-Side Form Interception**: JavaScript hooks intercept form submissions directly in the browser DOM.
- **Zero Credential Transmission**: Passwords, pins, and MFA tokens are permanently erased and replaced with `USER_ENTERED_VALUE` before network transit.
- **Opaque URL**: High-entropy 32-byte token with zero employee personal data in the link.

---

## 10. The Teachable Moment — Immediate Educational Debrief
- **Route**: `http://localhost:3000/s/[token]` (Post-interaction)
- **File**: `screenshots/10_simulation_debrief.png`

The instant an interaction occurs, the login page morphs into an educational debrief card delivering immediate learning feedback.

![The Teachable Moment Debrief Card](screenshots/10_simulation_debrief.png)

#### Key Capabilities:
- **Zero-Latency Feedback**: Delivered at the exact moment of cognitive engagement (4x higher learning retention).
- **Three Warning Signs Highlighted**:
  1. *Anomalous Channel*: Legitimate IT security never requests passwords via WhatsApp.
  2. *Domain Discrepancy*: Link domain does not match official corporate infrastructure.
  3. *Artificial Urgency*: High-pressure language designed to induce hurried compliance.
- **Positive Culture**: Non-punitive, empowering employees to become active defenders.

---

## 🔗 Quick Links to Presentation Assets

- 🖥️ **Interactive Web Presentation Deck**: [`docs/presentation/index.html`](index.html)
- 🎥 **Video Pitch Player**: [`docs/presentation/video_pitch_player.html`](video_pitch_player.html)
- 🖼️ **Interactive Screenshot Walkthrough**: [`docs/presentation/app_walkthrough.html`](app_walkthrough.html)
- 📹 **Rendered MP4 Video Pitch**: [`docs/presentation/kcsp_platform_pitch.mp4`](kcsp_platform_pitch.mp4)
