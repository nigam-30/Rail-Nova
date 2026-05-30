# 🚂 Rail Nova — Indian Train Booking System

> A full-stack, high-performance Indian railway ticket booking web application built with **FastAPI**, **SQLite**, **WebSockets**, and a vanilla **HTML/JS** frontend. Supports real seat allocation (CNF/RAC/WL), Tatkal booking with live queue, eWallet payments, and auto background cleanup tasks.

---

## 📋 Table of Contents

- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Prerequisites](#-prerequisites)
- [Installation & Setup](#-installation--setup)
- [Running the Application](#-running-the-application)
- [API Overview](#-api-overview)
- [Database Schema](#-database-schema)
- [Configuration](#-configuration)
- [Testing](#-testing)
- [Known Limitations](#-known-limitations)

---

## ✨ Features

| Feature | Description |
|---|---|
| **Train Search** | Search trains by source, destination, date, and coach class |
| **Seat Booking** | Real CNF / RAC / Waitlist allocation logic matching IRCTC rules |
| **Tatkal Booking** | Tatkal quota with time-gated window (AC: 10 AM, Non-AC: 11 AM, D-1) |
| **Live Queue** | Real-time Tatkal queue position via WebSocket |
| **Seat Availability** | Live seat availability broadcast via WebSocket on every booking |
| **eWallet** | In-app wallet with top-up and pay-via-wallet flow |
| **Mock Payments** | Simulated Razorpay-style create-order + verify flow |
| **Auth** | JWT-based login/register with PBKDF2 password hashing |
| **PNR Lookup** | Fetch booking status by PNR number |
| **Booking Cancellation** | Cancel booking with automatic refund to eWallet |
| **Auto Cleanup** | Background tasks every 60s to expire seat locks and rotate Tatkal queue |
| **Rate Limiting** | Login endpoint rate-limited to 5 attempts/minute per IP |
| **Request Logging** | All requests logged with method, path, status, and latency |

---

## 🛠 Tech Stack

**Backend**
- [FastAPI](https://fastapi.tiangolo.com/) — async web framework
- [SQLAlchemy](https://www.sqlalchemy.org/) — ORM with SQLite
- [Pydantic v2](https://docs.pydantic.dev/) + `pydantic-settings` — data validation
- [python-jose](https://github.com/mpdavis/python-jose) — JWT tokens
- [passlib](https://passlib.readthedocs.io/) + PBKDF2 — password hashing
- [uvicorn](https://www.uvicorn.org/) — ASGI server
- [aiosqlite](https://aiosqlite.omnilib.dev/) — async SQLite support

**Frontend**
- Vanilla HTML + JavaScript (no framework)
- [Tailwind CSS](https://tailwindcss.com/) via CDN
- Google Fonts (Merriweather + Inter)

**Database**
- SQLite (`train_booking.db`) — zero-config, file-based

**Data**
- `stations.csv` — all Indian railway stations with codes, coordinates, zone, state
- `trains_final.csv` — train master data with coach availability flags
- `stoppages.csv` — intermediate stoppage data for all trains (~30 MB)

---

## 📁 Project Structure

```
Rail Nova -Booking System Project/
├── frontend/
│   ├── index.html              # Single-page frontend app
│   └── app.js                  # All frontend JS (~2500 lines)
│
├── train-booking/
│   └── backend/
│       ├── app/
│       │   ├── main.py         # FastAPI app, middleware, lifespan tasks
│       │   ├── config.py       # Settings via pydantic-settings (.env)
│       │   ├── database.py     # SQLAlchemy engine + session
│       │   ├── dependencies.py # JWT auth dependency
│       │   │
│       │   ├── models/         # SQLAlchemy ORM models
│       │   │   ├── user.py
│       │   │   ├── train.py
│       │   │   ├── station.py
│       │   │   ├── booking.py
│       │   │   ├── passenger.py
│       │   │   ├── seat.py
│       │   │   ├── seat_lock.py
│       │   │   ├── payment.py
│       │   │   ├── tatkal_queue.py
│       │   │   ├── train_stoppage.py
│       │   │   └── wallet_transaction.py
│       │   │
│       │   ├── routers/        # FastAPI route handlers
│       │   │   ├── auth.py
│       │   │   ├── trains.py
│       │   │   ├── bookings.py
│       │   │   ├── passengers.py
│       │   │   ├── payments.py
│       │   │   └── websocket.py
│       │   │
│       │   ├── schemas/        # Pydantic I/O schemas
│       │   │   ├── auth.py
│       │   │   ├── train.py
│       │   │   ├── booking.py
│       │   │   ├── passenger.py
│       │   │   └── payment.py
│       │   │
│       │   ├── services/       # Business logic layer
│       │   │   ├── auth_service.py
│       │   │   ├── booking_service.py
│       │   │   ├── train_service.py
│       │   │   ├── payment_service.py
│       │   │   ├── seat_service.py
│       │   │   └── tatkal_service.py
│       │   │
│       │   ├── middleware/
│       │   │   ├── logging.py  # Request/response latency logging
│       │   │   └── rate_limit.py  # 5 req/min on /api/auth/login
│       │   │
│       │   └── utils/
│       │       ├── security.py    # JWT creation/decoding, password hashing
│       │       ├── helpers.py     # PNR generator
│       │       └── validators.py
│       │
│       ├── seed.py             # DB seeder (stations, trains, stoppages)
│       ├── run.py              # Uvicorn entry point
│       ├── requirements.txt    # Python dependencies
│       ├── .env.example        # Environment variable template
│       └── test_app.py         # API integration tests
│
├── stations.csv                # ~7000 Indian stations
├── trains_final.csv            # Train master data
├── stoppages.csv               # Train stoppage data (~30 MB)
├── railway_schema.sql          # Reference SQL schema
└── integrate_with_booking.py   # DB integration helper script
```

---

## ✅ Prerequisites

Make sure you have the following installed before starting:

| Tool | Minimum Version | Check Command |
|---|---|---|
| Python | 3.10+ | `python --version` |
| pip | Latest | `pip --version` |

> **Windows users:** Python 3.13 was used during development. Use `python` instead of `python3` in all commands below.

---

## 🚀 Installation & Setup

### Step 1 — Clone / Extract the Project

If you downloaded the zip, extract it. Your working directory should be:
```
Rail Nova -Booking System Project/
```

### Step 2 — Navigate to the Backend Directory

```bash
cd "Rail Nova -Booking System Project/train-booking/backend"
```

### Step 3 — Create a Virtual Environment

```bash
# Create venv
python -m venv venv

# Activate — Windows (PowerShell)
venv\Scripts\Activate.ps1

# Activate — Windows (Command Prompt)
venv\Scripts\activate.bat

# Activate — macOS / Linux
source venv/bin/activate
```

> You should see `(venv)` prefix in your terminal after activation.

### Step 4 — Install Dependencies

```bash
pip install -r requirements.txt
```

This installs:
- `fastapi`, `uvicorn[standard]` — web server
- `sqlalchemy`, `aiosqlite` — database ORM
- `pydantic[email]`, `pydantic-settings` — validation
- `passlib[bcrypt]`, `python-jose[cryptography]` — auth/security
- `python-multipart`, `httpx`, `alembic` — supporting libs

### Step 5 — Configure Environment Variables

Copy the example env file and edit it:

```bash
# Windows
copy .env.example .env

# macOS / Linux
cp .env.example .env
```

Open `.env` and update the values:

```env
DATABASE_URL=sqlite:///./train_booking.db
JWT_SECRET_KEY=your-super-secret-key-change-this   # ← CHANGE THIS!
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
FRONTEND_URL=http://localhost:8000
```

> ⚠️ **Important:** Change `JWT_SECRET_KEY` to a long random string before use.

### Step 6 — Seed the Database

The seeder reads the CSV files and populates stations, trains, and stoppages into SQLite.

Make sure you are in the `backend/` directory, then run:

```bash
python seed.py
```

Expected output:
```
Dropping all existing SQLite tables to remove old data...
Initializing SQLite tables...
Seeding stations...
Seeded XXXX new stations.
Seeding trains...
Seeded XXXX new trains.
Seeding train stoppages (this might take a few seconds)...
Seeded XXXXXX stoppages successfully.
Pre-seeding default seat configurations...
Pre-seeded XX seat records for next 5 days.
```

> ⏱ Stoppages seeding may take **15–30 seconds** due to the large dataset (~30 MB CSV).

> ⚠️ The seeder will look for `stations.csv`, `trains_final.csv`, and `stoppages.csv` in the parent directories. Make sure these files are present at the root of the project zip (they should be by default).

---

## ▶️ Running the Application

### Start the Backend Server

From the `backend/` directory (with venv active):

```bash
python run.py
```

Or directly with uvicorn:

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

You should see:
```
Starting background cleanup tasks...
Migration complete!
Static files mounted from: .../frontend
INFO:     Uvicorn running on http://0.0.0.0:8000
```

### Open the Frontend

Open your browser and navigate to:

```
http://localhost:8000
```

The FastAPI backend serves the frontend static files directly — **no separate frontend server needed**.

### Access the API Docs

Interactive Swagger UI is available at:

```
http://localhost:8000/docs
```

ReDoc documentation:

```
http://localhost:8000/redoc
```

---

## 📡 API Overview

All API routes are prefixed with `/api/`.

### Authentication — `/api/auth`

| Method | Endpoint | Description | Auth Required |
|---|---|---|---|
| POST | `/api/auth/register` | Register new user | No |
| POST | `/api/auth/login` | Login, receive JWT | No |
| POST | `/api/auth/logout` | Logout (client discards token) | No |
| GET | `/api/auth/me` | Get current user profile | Yes |
| PUT | `/api/auth/profile` | Update profile | Yes |
| PUT | `/api/auth/change-password` | Change password | Yes |

### Trains & Stations — `/api/trains`, `/api/stations`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/trains/search?from=&to=&date=&class=` | Search trains |
| GET | `/api/trains/{train_number}` | Train details |
| GET | `/api/trains/{train_number}/availability?class=&date=` | Seat availability |
| GET | `/api/stations/search?q=` | Autocomplete station search |

### Bookings — `/api/bookings`

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/bookings` | Create a new booking |
| GET | `/api/bookings` | Get booking history |
| GET | `/api/bookings/{pnr}` | Get booking by PNR |
| DELETE | `/api/bookings/{pnr}` | Cancel a booking |
| POST | `/api/bookings/tatkal/join` | Join Tatkal queue |
| GET | `/api/bookings/tatkal/status` | Get Tatkal queue position |

### Payments — `/api/payments`

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/payments/create-order` | Create mock payment order |
| POST | `/api/payments/verify` | Verify and confirm payment |
| GET | `/api/payments/{booking_id}` | Get payment status |
| GET | `/api/payments/ewallet/balance` | Get eWallet balance + history |
| POST | `/api/payments/ewallet/topup` | Top-up eWallet |
| POST | `/api/payments/pay-via-wallet` | Pay booking from eWallet |

### WebSockets

| Endpoint | Description |
|---|---|
| `ws://localhost:8000/ws/seats/{train_number}/{journey_date}` | Live seat availability updates |
| `ws://localhost:8000/ws/tatkal/{train_number}/{journey_date}/{coach_class}` | Live Tatkal queue updates |
| `ws://localhost:8000/ws/heartbeat` | Browser tab heartbeat (triggers auto-shutdown when all tabs close) |

---

## 🗄 Database Schema

Key tables in `train_booking.db`:

| Table | Description |
|---|---|
| `users` | Registered users with eWallet balance, ID verification fields |
| `trains` | Train master data with coach type flags |
| `stations` | Station codes, names, coordinates, zone, state |
| `train_stoppages` | Intermediate station stops per train |
| `bookings` | Booking records with PNR, status (CNF/RAC/WL/CANCELLED) |
| `passengers` | Passengers attached to each booking with berth/seat assignment |
| `seats` | Dynamic seat availability per train+date+class |
| `seat_locks` | Temporary 10-minute seat locks during payment flow |
| `payments` | Payment records with mock Razorpay order/payment IDs |
| `tatkal_queue` | Tatkal queue entries with WAITING/ACTIVE/EXPIRED status |
| `wallet_transactions` | eWallet credit/debit transaction history |

---

## ⚙️ Configuration

All configuration is loaded from `.env` via `pydantic-settings`:

| Variable | Default | Description |
|---|---|---|
| `DATABASE_URL` | `sqlite:///./train_booking.db` | SQLAlchemy DB connection string |
| `JWT_SECRET_KEY` | `your-super-secret-key-change-this` | Secret for JWT signing — **must be changed** |
| `JWT_ALGORITHM` | `HS256` | JWT signing algorithm |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | `30` | JWT token validity in minutes |
| `FRONTEND_URL` | `http://localhost:5173` | CORS allowed origin for frontend |

---

## 🧪 Testing

A test script is provided at `backend/test_app.py`. Run it while the server is running:

```bash
# Make sure server is running first, then:
python test_app.py
```

For PowerShell-based API testing:

```powershell
# Run the PowerShell test script (Windows)
.\test_app.ps1
```

---

## ⚠️ Known Limitations

- **Payment is mocked** — No real Razorpay integration. Payment verification always succeeds with mock order/payment IDs.
- **SQLite only** — Not suitable for multi-process production deployments. For production, migrate to PostgreSQL by updating `DATABASE_URL`.
- **No email verification** — `is_verified` field exists in DB but verification emails are not sent.
- **In-memory rate limiting** — Login rate limits reset on server restart (not persisted).
- **Auto-shutdown feature** — Server shuts down 5 seconds after all browser tabs close (heartbeat WebSocket). Disable by removing `ws_heartbeat` router in production.
- **Stoppages data** — `stoppages.csv` is ~30 MB. First seed may take 15–30 seconds.

---

## 👤 Author

**Nigam Mehta** — Pursuing Electronics Engineering (VLSI Design And Technology), SAKEC, Mumbai, India  
Project: Rail Nova — Indian Railway Booking System  
Stack: FastAPI · SQLAlchemy · SQLite · Vanilla JS · Tailwind CSS

## ⚠️ License:
This project is intended for **simulation, knowledge, and testing purposes only.**
Not for commercial use or production deployment.
