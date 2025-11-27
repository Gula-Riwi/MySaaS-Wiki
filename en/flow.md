# MeetLines Application Flow

This document describes the **full flow** the application follows from business registration to customer retention.

---

## Phase 1 – Business Onboarding (Brain Configuration)
1. **Registration and trial** – The business owner registers on the website and receives a 14-day trial.
2. **Context injection (Orchestrator)** – The owner completes a form with key information (what they sell, schedules, pricing, cancellation policy).
   - The system generates the **Knowledge Base** (so the Reception Bot can answer general questions) and the **Business Rules** (so the Transactional Bot understands availability).

---

## Phase 2 – End Customer Entry
### A. Mobile App (Marketplace/Directory)
- The user opens the app, sees a list of businesses, and selects one.
- The user can view the calendar or catalog and press **Book** or **Buy**.
- The system stores the request in the database and sends a push notification to the business owner.

### B. Chat (WhatsApp / Instagram / Meta)
- The user starts a conversation with the business' number.
- Bots come into play here to handle the flow.

---

## Phase 3 – Conversational Flow (Bots Interaction)
1. **Reception and Context Bot** – Analyzes the message, queries the Knowledge Base, and answers generic questions. Detects booking or purchase intents and delegates to the next bot.
2. **Transactional Bot / Setter** – Manages bookings or sales:
   - **Service**: checks the real schedule, proposes times, and blocks the slot.
   - **Sale**: captures order details, rates the customer, sends a payment link, or notifies an advisor.
   - Creates a record in the Dashboard with status **Pending**.
3. **Notifier Bot** – Sends confirmation to the customer via WhatsApp: *"Your appointment/order has been confirmed for day X"*.

---

## Phase 4 – Execution and Management (Dashboard)
- The business owner receives an alert in the app/web.
- The service is performed or the product delivered.
- The owner marks the transaction as **Completed** in the Dashboard, which triggers post-sale processes.

---

## Phase 5 – Post-Sale and Retention
1. **Reputation Bot (Feedback)** – After a while, asks the customer for feedback and requests a review or logs any complaints.
2. **Reactivation Bot (Silent Seller)** – Detects inactivity (30 days, 1 year, etc.) and sends proactive messages to drive re-scheduling.

---

## Summary of required components
- **Dashboard/App (Businesses)** – Configuration and state management.
- **Mobile App (Customers)** – Marketplace without chat.
- **Reception Bot** – Generative AI + RAG.
- **Transactional Bot** – Rigid logic + DB.
- **Notifier / Reputation Bot** – Events and confirmations.
- **Reactivation Bot** – Cron jobs/time-based triggers.

This flow ensures any type of business (lawyer, barber, retail store) can operate using the same modular and extensible architecture.
