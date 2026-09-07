# Security & GDPR Compliance Architecture

The WhatsApp Security Awareness Platform handles authorized security assessments. It was designed under the principles of **Privacy by Design**, **Zero-Trust Credential Handling**, and **GDPR Data Minimization**.

---

## 1. Zero-Credential Harvesting Guarantee

In legitimate corporate security awareness programs, collecting actual employee passwords or secret tokens is dangerous, unnecessary, and legally hazardous.

### Technical Implementation:
- Public simulation endpoints (`/s/[token]`) intercept all form submissions with client-side JavaScript (`client-wrapper.tsx`).
- Before sending the interaction signal to the backend (`/api/simulation/interact`), input values are sanitized using `sanitizeSimulationPayload`.
- Real passwords, authentication tokens, MFA codes, credit card numbers, and API keys are **never transmitted, logged, or written to disk**.
- The backend records only a binary telemetry event: `SIMULATION_INTERACTION`, plus the metadata field names (e.g. `["username", "password"]`) to prove interaction without holding the secrets.
- The user is immediately transitioned to an educational learning moment debrief.

---

## 2. Opaque Tracking Tokens

- Standard phishing simulators often leak internal user IDs or email addresses in the URL (e.g. `?email=target@corp.com&uid=123`).
- In this platform, each participant is assigned a cryptographically random, collision-resistant opaque token (`generateOpaqueToken(32)` -> 256 bits of entropy).
- URLs take the form: `https://domain.tld/t/{opaque-token}`.
- Opaque tokens are resolved only internally within the platform. A third-party observer or network intermediate cannot derive the participant's identity or department from the URL.

---

## 3. Privacy-Preserving Technical Identifiers

- In accordance with GDPR requirements for IP address minimization, the platform **never stores raw IP addresses or raw User-Agents** in the event table.
- All technical identifiers are converted into HMAC SHA-256 hashes using a private, rotated server salt:
  ```ts
  hashIdentifier(ip, PRIVACY_SALT)
  ```
- This allows calculating unique clicks and device patterns without storing actionable PII.

---

## 4. Configurable Data Retention & Anonymization

- Administrators can configure automatic data retention policies (e.g., purge or anonymize records after 30, 90, or 180 days).
- When anonymization is executed (`CampaignExecutionEngine.runRetentionCleanup`):
  - Participant first names are changed to `Anon_XXXXXX`.
  - Last names are set to `ANONYMIZED`.
  - Phone numbers are masked (`+000000XXXX`).
  - Emails and metadata are erased (`null`).
  - Aggregated statistics, department risk rankings, and campaign completion rates remain intact for historical compliance reporting.

---

## 5. Administrative Authentication & Audit Logging

- Admin sessions utilize cryptographically signed, HTTP-only, SameSite cookies.
- Passwords are encrypted using salted bcrypt hashes (`10 rounds`).
- Every critical administrative action (`USER_LOGIN`, `CAMPAIGN_CREATED`, `CAMPAIGN_STARTED`, `PARTICIPANTS_IMPORTED`, `DATA_DELETED`) generates an immutable entry in the `AuditLog` table.
