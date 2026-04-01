# System Architecture: Client-Capture-Automations

This document provides a technical deep dive into the architectural decisions and data flows that power the **Client-Capture-Automations** ecosystem.

---

## 1. High-Level System Design

The platform is built as a **decoupled micro-service architecture**, ensuring that the high-traffic demo frontends remain responsive while the backend handles heavy-lifting tasks like AI inference and external API integrations.

### Core Components:
*   **The Frontend Layer:** Three distinct Next.js applications (Showcase, Demo, CRM) optimized for speed and SEO.
*   **The Orchestration Layer:** A FastAPI-based Python server that acts as the brain, routing requests to AI services, SMS gateways, and the database.
*   **The Persistence Layer:** A Supabase-managed PostgreSQL database utilizing Row-Level Security (RLS) and real-time triggers.

---

## 2. Automation Lifecycle & Data Flow

### UC-1: The AI Qualification Flow
1.  **Ingress:** User sends a message via the Next.js Chatbot widget.
2.  **State Management:** The backend (FastAPI) retrieves the `session_id` to maintain conversation context.
3.  **Inference:** The message is sent to OpenAI with a custom-tuned system prompt for the "ABC Home Services" persona.
4.  **Qualification:** If the AI detects a "Qualification Event" (name, phone, and service type extracted), it returns a specific `lead_captured: true` flag.
5.  **Commit:** The backend instantly writes the new lead to the `leads` table.
6.  **Real-time Notification:** Supabase's real-time engine pushes the new lead to the CRM Dashboard without a page refresh.

### UC-2: The Missed-Call Recovery Loop
1.  **Trigger:** A simulated missed call occurs (or an actual Twilio webhook is received).
2.  **Normalization:** The FastAPI server normalizes the phone number to E.164 format.
3.  **Sanitization:** The outbound message is sanitized (removing non-GSM-7 characters) to ensure 100% delivery across global carriers.
4.  **Audit Trail:** The SMS is logged in both `automation_logs` (for system health) and `messages` (for the CRM conversation history).

---

## 3. Tech Stack Deep Dive

### **Backend: FastAPI (Python)**
*   **Why?** Asynchronous processing (async/await) allows the server to handle concurrent API calls (OpenAI, Twilio, Supabase) without blocking.
*   **Validation:** Uses Pydantic models for strict type safety and auto-generation of OpenAPI (Swagger) documentation.

### **Database: Supabase / PostgreSQL**
*   **Real-time:** Utilizes PostgreSQL's WAL (Write-Ahead Log) to broadcast changes to the frontend.
*   **Relational Integrity:** Strict foreign key constraints between `leads`, `messages`, `appointments`, and `automation_logs` ensure data consistency.

### **Frontend: Next.js (TypeScript)**
*   **API Rewrites:** Configured to act as a reverse proxy, avoiding CORS issues and hiding the backend API URL from the client's browser.
*   **Tailwind CSS:** Ensures a unified design system across all three platforms with a custom color palette for the "ABC Home Services" brand.

---

## 4. Scalability & Performance
*   **Cold Start Management:** Optimized for the Render "Free Tier" with lightweight container images.
*   **AI Latency:** Implements a 3-second timeout strategy with fallback responses to ensure the user never experiences a "hanging" chat window.
*   **Error Handling:** Every automation trigger is wrapped in a dedicated error-handling service that logs failures to a persistent audit trail without interrupting the customer's journey.

---

## 5. Integration Summary
| Service | Role | Integration Method |
| :--- | :--- | :--- |
| **OpenAI** | Conversation AI | REST API (POST) |
| **Twilio** | SMS Delivery | Twilio Python SDK |
| **Brevo / SendGrid** | Email Automation | SMTP / REST API |
| **Supabase** | Database & Auth | Supabase Client SDK |

---

*This architecture demonstrates a production-ready approach to business automation, prioritizing reliability and real-time feedback.*
