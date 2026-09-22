# LockGuard 🛡️ — Concurrency-Safe Resource Booking Platform

> **Hackathon Pitch:** A resource booking platform that **proves, live on stage**, that double-booking is impossible — even under 100 simultaneous requests.

---

## 📽️ Demo Architecture

```
                       RESOURCE BOOKING PLATFORM
                                  │
               ┌──────────────────┴──────────────────┐
               │                                     │
         Normal User Flow                     Adversarial / Chaos Flow
               │                                     │
      Calendar Slot View                     100 Concurrent Requests
               │                                     │
       Create Booking Request                Chaos Simulation Engine
               │                                     │
               └──────────────────┬──────────────────┘
                                  ▼
                      CONCURRENCY PROTECTION ENGINE
                                  │
            ┌─────────────────────┼─────────────────────┐
            ▼                     ▼                     ▼
      Postgres Row Lock      Transaction          Idempotency Key
     (SELECT FOR UPDATE)   Isolation Level        Header / Unique Key
            │                     │                     │
            └─────────────────────┼─────────────────────┘
                                  ▼
                           Single Source of Truth
                                  │
               ┌──────────────────┴──────────────────┐
               ▼                                     ▼
       [200 OK] CONFIRMED                   [409 CONFLICT] REJECTED
               │                                     │
               ├─ Socket.io broadcast                ├─ Alternative slot suggestion
               ├─ Audit log entry                    ├─ Cell flashes RED with toast
               └─ Live calendar update               └─ Live chaos counter update
```

---

## ✨ Features & Money Shots

1. **PostgreSQL Row-Lock Concurrency Engine (`SELECT FOR UPDATE`)**:
   - Strictly serializes concurrent booking attempts for the same resource slot inside a database transaction.
   - Guaranteed **first writer wins**: 1 request confirmed (`200 OK`), all concurrent overlapping requests rejected (`409 Conflict`).

2. **The 100-Request Chaos Button (Stage Money Shot)**:
   - Live stress-test runner firing 100 simultaneous HTTP requests at a single time slot.
   - Displays real-time result breakdown (**1 confirmed / 99 rejected**) with sub-millisecond execution latency audit.

3. **Structured 409 Conflict Response + Smart Alternative Suggestions**:
   - Returns structured JSON payload containing conflict details.
   - Calculates 3 nearby available time slots for instant 1-click alternative booking.

4. **Idempotency Guarantee (`Idempotency-Key` Header)**:
   - Retried requests with identical headers return existing booking without creating duplicate DB entries.

5. **Real-Time Socket.io Synchronization & Flash Alerts**:
   - Instantly updates calendar grid across all connected browser clients without manual refresh.
   - Losing clients flash red with live toast: *"Booked 0.3s ago by another user"*.

6. **Dual Browser Side-by-Side Simulator**:
   - Integrated split-screen browser mode to simulate User A vs User B competing for the same slot on a single screen.

7. **Live Streaming Audit Feed**:
   - Append-only transactional audit trail streaming all `BOOKING_CREATED`, `CONFLICT_DETECTED`, and `IDEMPOTENT_RETRY` events.

---

## 🛠️ Tech Stack

- **Backend:** Node.js, Express, TypeScript, Socket.io
- **Database:** PostgreSQL 16 (Row-level locking `FOR UPDATE`)
- **ORM:** Prisma ORM
- **Frontend:** React, TypeScript, Vite, Tailwind CSS, Lucide Icons, Framer Motion, Canvas Confetti

---

## 🚀 Quick Start & Local Execution

### Prerequisites
- Node.js 18+
- Docker Desktop

### 1. Start PostgreSQL Container
```bash
docker compose up -d
```

### 2. Install & Start Backend
```bash
cd backend
npm install
npx prisma db push
npm start
```

### 3. Start Frontend (Optional Dev Server)
```bash
cd frontend
npm install
npm run dev
```

Open `http://localhost:5000` (or `http://localhost:3000` with dev proxy) to run the app.

---

## 🎭 Live Stage Demo Script (for Judges)

1. Open `http://localhost:5000` in browser.
2. Select resource **"3D Printer #1 (Ultra 4K)"**.
3. Toggle **"Dual Browser Simulator"** in navbar to show side-by-side windows.
4. Click **"Fire Simultaneous Booking (Alice vs Bob)"** for 11:00 AM.
   - Observe **Alice -> 🟢 CONFIRMED**, **Bob -> 🔴 409 CONFLICT**.
5. Hit **"Launch 100 Concurrent Requests"** Chaos Button.
   - Watch live counter: **1 Confirmed / 99 Rejected**.
6. Open **"Live Audit Stream"** drawer to show immutable database events.
