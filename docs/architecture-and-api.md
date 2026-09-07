# WhatsApp Security Awareness Simulation Platform - Architecture & API Specification

## 1. System Overview

The KCSP platform is an enterprise-ready awareness simulation platform engineered as a modular monolith with clean domain boundaries.

```
                  ┌───────────────────────────────┐
                  │       Admin Web Console       │
                  │  (Next.js App Router / React) │
                  └───────────────┬───────────────┘
                                  │
    ┌─────────────────────────────┼─────────────────────────────┐
    │                             │                             │
    ▼                             ▼                             ▼
┌──────────────┐         ┌─────────────────┐           ┌──────────────────┐
│  Campaigns   │         │   Engine Queue  │           │    Analytics     │
│ & Templates  │         │   & Dispatch    │           │    & Reports     │
└───────┬──────┘         └────────┬────────┘           └────────┬─────────┘
        │                         │                             │
        └─────────────────────────┼─────────────────────────────┘
                                  │
                                  ▼
               ┌──────────────────────────────────────┐
               │    WhatsApp Provider Abstraction     │
               │        (WhatsAppProvider)            │
               └──────────┬────────────────┬──────────┘
                          │                │
                          ▼                ▼
                  ┌──────────────┐ ┌──────────────┐
                  │ MockProvider │ │ WASender API │
                  └──────────────┘ └──────────────┘
```

---

## 2. Domain Modules

### 2.1 `@kcsp/shared`
- Shared data types, enums (`CampaignStatus`, `MessageStatus`, `EventType`, `AuditAction`).
- Variable interpolation engine (`{{first_name}}`, `{{last_name}}`, `{{department}}`, `{{tracking_url}}`).
- Privacy utilities: HMAC SHA-256 IP/User-Agent hashing with dynamic salt, phone number masking.
- Zero-credential input sanitization (`sanitizeSimulationPayload`).

### 2.2 `@kcsp/database`
- Centralized Prisma schema for SQLite / PostgreSQL.
- Foreign keys, cascading deletions, and composite unique constraints.
- Optimized indexes on `campaignId`, `participantId`, `trackingToken`, `eventType`, `timestamp`, `providerMessageId`, `phone`.

### 2.3 `@kcsp/whatsapp`
- Provider interface:
  ```ts
  interface WhatsAppProvider {
    sendMessage(request: SendMessageRequest): Promise<SendMessageResponse>;
    getMessageStatus(providerMessageId: string): Promise<MessageStatusResponse>;
    validateConfiguration(): Promise<boolean>;
    parseWebhook(payload: unknown, headers?: Record<string, string>): NormalizedWebhookEvent | null;
  }
  ```
- `MockWhatsAppProvider`: Local in-memory / zero-external-call simulation with latency, simulated delivery failures, and status transitions.
- `WASenderApiProvider`: Production REST API adapter for WASender with bearer authentication and webhook validation.
- `ProviderRegistry`: Dynamic provider resolver.

### 2.4 `@kcsp/tracking`
- Generates 32-byte opaque tokens.
- Internal token resolution mapping `token -> (campaignId, participantId)`.
- Event logger recording:
  - `LINK_OPENED`
  - `LANDING_PAGE_VIEWED`
  - `SIMULATION_INTERACTION`

### 2.5 `@kcsp/engine`
- Asynchronous campaign execution queue.
- Concurrency pool, rate-limiting delays between messages.
- Exponential backoff retries on transient errors.
- Idempotency per `(campaignId, participantId)`.
- Provider webhook ingestion and message status synchronization.
- GDPR privacy retention cleanup (anonymization & deletion).

### 2.6 `@kcsp/analytics`
- Metric calculation: Delivery rate, click rate, unique click rate, interaction rate, vulnerability rate.
- Department segmentation and time-series aggregations.
- CSV report generator (full administrative and privacy-anonymized variants).

---

## 3. Public Endpoints Specification

### `GET /t/[token]`
- Resolves opaque token.
- Hashes client IP and User-Agent using SHA-256 HMAC.
- Appends `LINK_OPENED` event and sets participant status to `CLICKED`.
- 302 Redirects to `/s/[token]`.

### `GET /s/[token]`
- Resolves campaign landing page HTML/CSS.
- Appends `LANDING_PAGE_VIEWED` event.
- Renders controlled simulation page.

### `POST /api/simulation/interact`
- Invoked when participant submits the simulation form.
- Payload is strictly sanitized: real passwords, pins, and secrets are discarded immediately.
- Appends `SIMULATION_INTERACTION` event and updates participant status to `INTERACTED`.
- Returns interactive security debrief with warning indicators and educational tips.

### `POST /api/webhooks/whatsapp/[provider]`
- Accepts delivery callbacks from WhatsApp providers.
- Validates webhook secret or HMAC signature.
- Updates internal message status (`DELIVERED` / `FAILED`).
- Appends normalized `MESSAGE_DELIVERED` or `MESSAGE_FAILED` event.
