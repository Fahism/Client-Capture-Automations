# Automation Use Cases: Business Value & Technical Logic

This document details the six core automation use cases implemented in the **Client-Capture-Automations** platform, explaining the business problem they solve and the high-level logic behind them.

---

## 1. 🤖 AI-Powered Chatbot (FAQ + Qualification)
*   **Business Problem:** Customers often ask the same questions (Pricing? Hours? Services?) and leave if they don't get an immediate answer.
*   **The Solution:** An intelligent chatbot that provides instant answers while simultaneously qualifying the lead through natural conversation.
*   **Key Logic:**
    1.  User input is processed via OpenAI with a "persona-locked" system prompt.
    2.  The engine extracts entities (Name, Phone, Service Type) using JSON-mode function calling.
    3.  Once all required entities are present, the bot acknowledges the request and auto-syncs the lead data to the database.

---

## 2. 📞 Missed Call → SMS (Safety Net)
*   **Business Problem:** Missed calls are missed revenue. Most callers won't leave a voicemail; they simply call the next business on Google.
*   **The Solution:** The system automatically detects a missed call and immediately sends a text message to engage the prospect.
*   **Key Logic:**
    1.  The backend listens for a "Missed Call" event (via webhook or manual simulation for demo).
    2.  The number is normalized and sanitized to avoid carrier-side blocking.
    3.  A personalized SMS is sent, and the "conversation thread" is initialized in the CRM.

---

## 3. ✉️ Instant Email Follow-Up
*   **Business Problem:** Leads expect professional confirmation. Lack of a follow-up email makes a business look disorganized.
*   **The Solution:** A high-quality, branded email is sent within seconds of any lead-capture event.
*   **Key Logic:**
    1.  The backend's `leads.py` router triggers an asynchronous task (`asyncio.create_task`) upon lead creation.
    2.  The email service renders a template with the lead's name and requested service.
    3.  The email is dispatched via SMTP/API, and the status is logged for tracking.

---

## 4. ⏰ Appointment Reminders (No-Show Protection)
*   **Business Problem:** Forgotten appointments waste time and resources.
*   **The Solution:** Automated SMS reminders ensure the customer remembers the booking and has an easy way to confirm or reschedule.
*   **Key Logic:**
    1.  When an appointment is booked, the system calculates the "Reminder Window" (e.g., 24h prior).
    2.  An automated SMS task is queued or triggered immediately for the demo.
    3.  The message includes the booking details and the business's contact info.

---

## 5. ⭐ Reputation Management (Review Request)
*   **Business Problem:** Online reviews are critical for local SEO, but businesses often forget to ask for them.
*   **The Solution:** The system automatically requests a review the moment a job is marked "Completed" in the CRM.
*   **Key Logic:**
    1.  The CRM Dashboard sends a `PATCH` request to the `/api/appointments/{id}` endpoint with `status: "Completed"`.
    2.  The backend identifies the linked `lead_id` and triggers the `review_request` service.
    3.  A personalized SMS is sent with a direct link to the business's Google/Review page.

---

## 6. 🛒 Abandoned Booking Recovery
*   **Business Problem:** Many users start filling out a booking form but get distracted and leave before clicking "Submit."
*   **The Solution:** If a user enters their name and phone but doesn't complete the booking within 30 seconds, the system sends an "Abandoned Booking" SMS.
*   **Key Logic:**
    1.  The frontend monitors input field activity.
    2.  If the form remains incomplete for a set duration, an "Abandoned Booking" event is sent to the backend.
    3.  The backend sends a "Gentle Nudge" SMS with a link to return and finish the booking.

---

*These six automations work together to create a "frictionless" experience for the customer while maximizing efficiency for the business.*
