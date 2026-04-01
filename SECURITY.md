# Security & Compliance: Client-Capture-Automations

This document outlines the security architecture and compliance measures implemented within the **Client-Capture-Automations** ecosystem.

---

## 1. Data Protection & Privacy
Handling Personally Identifiable Information (PII), such as names, phone numbers, and addresses, requires a rigorous security framework.

*   **Database Encryption:** All lead data is persisted in a Supabase (PostgreSQL) database with **AES-256 encryption-at-rest**.
*   **Row-Level Security (RLS):** Supabase RLS policies ensure that dashboard users can only access the data they are authorized to see, preventing cross-tenant data leakage.
*   **TLS/SSL:** All data in transit (API calls from Frontend to Backend) is encrypted using **TLS 1.2+**.

---

## 2. Infrastructure Security
The system is built on modern, cloud-native platforms with enterprise-grade security controls.

*   **FastAPI & Pydantic:** The backend uses strict Pydantic models for data validation, providing native protection against **SQL Injection** and **Buffer Overflow** attacks.
*   **Decoupled Architecture:** By separating the Frontend, Backend, and Database, the platform minimizes the attack surface.
*   **Secure Secrets Management:** All API keys (OpenAI, Twilio, Supabase) are managed via environment variables and never committed to source control.

---

## 3. Communication Security
Integrations with third-party communication providers follow industry best practices.

*   **E.164 Normalization:** Phone numbers are normalized using the **Google libphonenumber** standard before processing, preventing malformed inputs from triggering unexpected logic.
*   **GSM-7 Sanitization:** SMS payloads are sanitized to remove non-GSM-7 characters, ensuring delivery integrity across all carrier types.
*   **Webhook Verification:** Incoming webhooks (e.g., from Twilio) are validated to ensure they originate from the trusted provider.

---

## 4. Operational Compliance
The system's design incorporates principles of **Least Privilege** and **Auditing**.

*   **Audit Logging:** Every automation event (AI response, SMS sent, Email delivered) is logged in a persistent `automation_logs` table for debugging and compliance verification.
*   **Rate Limiting:** The backend implements rate-limiting to protect against **Denial of Service (DoS)** attacks and API abuse.

---

*For technical security inquiries or detailed penetration testing reports, please contact the project maintainer.*
