# AI Hotel Receptionist (Node.js + n8n + Neon)

Conversational hotel assistant that talks like a real receptionist, understands what the guest wants (ask about rooms, check availability, make a booking, cancel a booking, ask general hotel info) and triggers n8n workflows + Neon Postgres in the background.

This repository is mainly **frontend + documentation**.  
The full backend logic (Node.js agent + LangChain prompts + n8n/Neon details) lives in a **private repository**.

---

## ✨ Features

The assistant can:

- **Small talk**  
  “Hi, how are you?”, “Thanks, bye”, etc.

- **Check availability**  
  - Detects when the guest asks about available rooms for specific dates  
  - If dates are missing → asks follow-up questions  
  - Calls an n8n workflow (SQL query against Neon `rooms` / `bookings`)  
  - Returns a natural summary of available room types, capacity and price

- **Create a booking**  
  - Detects booking intent (“I want to book…”, “Please reserve…”)  
  - Collects: guest name, email, dates; chooses a `room_number` from the available rooms  
  - Calls a booking workflow in n8n → `INSERT` into Neon `bookings`  
  - Returns a confirmation message (optionally with booking ID)

- **Cancel a booking**  
  - Detects cancellation intent  
  - Asks for guest name and room number (or other ID)  
  - n8n checks if the booking exists; if yes → removes it;  
    if not → assistant explains that no matching booking was found.

- **Hotel & surroundings info (INFO intent)**  
  - Questions like: “When is breakfast?”, “Do you have a spa?”,  
    “Which restaurants are nearby?”, “Where can I park?”, etc.  
  - Assistant sends the question to an `info` workflow in n8n  
  - n8n returns structured text (or content extracted from a “hotel guide” PDF)  
  - Assistant answers briefly and naturally based only on that information.

- **Context switching**  
  - Can switch topics mid-conversation:  
    rooms → booking → general info → cancel, without losing the flow.

---

## 🧱 Architecture

High-level overview:

```text
Browser (index.html + JS)
        │  HTTP (JSON)
        ▼
Node.js Agent (Express + ChatOpenAI)
  - Intent detection (availability / booking / cancellation / info / other)
  - Per-session state (context, room_number, dates, guest data…)
  - Orchestration towards n8n
        │  Webhooks (JSON)
        ▼
n8n Workflows
  - check_availability → Neon SELECT rooms/availability
  - booking           → Neon INSERT bookings (+ pricing)
  - cancel_booking    → Neon DELETE / UPDATE bookings
  - hotel_info        → return hotel guide / surroundings info
        │
        ▼
Neon Postgres (rooms, bookings, …)
