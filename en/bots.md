# Bots and Components

This document describes the bots and components used by MeetLines.

- **Reception Bot**: Understands user intents, answers general questions using the Knowledge Base, and routes booking or purchase flows to the Transactional Bot.
- **Transactional Bot**: Manages bookings and sales, checks availability, creates pending records, and handles payment flows.
- **Notifier Bot**: Sends confirmations and notifications to users (via WhatsApp, push, email) and handles event-driven alerts.
- **Reputation Bot**: Gathers feedback and reviews post-service.

Components:
- **Dashboard / Admin app**: Business owners manage bookings, schedules, and statuses.
- **Mobile App**: Customers browse businesses, services, and book appointments.
- **Backend services**: API, Application logic, Domain model, Infrastructure services (DB, queues, external integrations).
