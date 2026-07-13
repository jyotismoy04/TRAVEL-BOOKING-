# Wayfare — AI-powered travel booking site

A demo travel booking site built with **HTML/CSS/JS** on the front end, a **Node.js + Express** API backed by **MySQL**, and an **AI travel co-pilot** powered by the Claude API.

## What's inside

```
travel-ai/
├── frontend/          Static site — open directly or serve with any static server
│   ├── index.html
│   ├── style.css
│   └── script.js
├── backend/            Express API
│   ├── server.js
│   ├── routes/          catalog.js, search.js, bookings.js, ai.js
│   ├── services/         aiService.js  (Claude API integration)
│   ├── config/db.js      MySQL connection pool
│   ├── package.json
│   └── .env.example
└── database/
    └── schema.sql       Tables + seed data
```

## Features

- **Flight search** with a from/to/date form, results pulled from MySQL (falls back to generated mock results if the backend isn't running, so the front end works standalone too).
- **Destination and deals browsing**, driven by the `destinations` and `flights` tables.
- **Booking flow** with a confirmation modal that writes a row to `bookings` and returns a reference code.
- **AI co-pilot chat widget**: describe a trip in plain language ("beach trip, 4 days, under ₹35k") and it replies with a suggested destination, a short day-by-day shape, and a budget note — powered by the Claude API (`/api/ai/chat`), with conversation history optionally persisted to `ai_conversations`.
- Graceful offline fallbacks everywhere: no MySQL connection or no API key won't crash the demo, it just degrades to mock data / a rule-based co-pilot reply.

## Setup

### 1. Database

```bash
mysql -u root -p < database/schema.sql
```

This creates the `wayfare` database with tables for users, destinations, airports, flights, hotels, bookings, and AI conversation history, plus seed rows so search/browse work immediately.

### 2. Backend

```bash
cd backend
npm install
cp .env.example .env
# edit .env: set DB_PASSWORD and ANTHROPIC_API_KEY
npm start
```

The API runs on `http://localhost:4000` by default. Health check: `GET /api/health`.

**Get a Claude API key** at https://console.anthropic.com — without one, `/api/ai/chat` still works using a rule-based offline fallback, just without real language understanding.

### 3. Frontend

Just open `frontend/index.html` in a browser, or serve it with any static server, e.g.:

```bash
cd frontend
npx serve .
```

If your backend runs somewhere other than `http://localhost:4000/api`, set it before `script.js` loads:

```html
<script>window.WAYFARE_API_BASE = "https://your-api.example.com/api";</script>
<script src="script.js"></script>
```

## API reference

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/destinations` | List curated destinations |
| GET | `/api/deals` | Cheapest current flight deals |
| POST | `/api/search/flights` | `{ from, to, depart }` → matching flights |
| POST | `/api/bookings` | Create a booking, returns `{ bookingRef }` |
| GET | `/api/bookings/mine?email=` | List a traveler's bookings |
| POST | `/api/ai/chat` | `{ message, sessionId }` → AI co-pilot reply |

## Extending it

- Swap the mock flight generator for a real GDS/flight API (Amadeus, Skyscanner) in `routes/search.js`.
- Add auth (JWT) and wire bookings to `user_id` instead of email lookups.
- Add a hotels search endpoint mirroring `search/flights` using the `hotels` table.
- Turn the co-pilot's structured `itinerary` array into a rendered day-by-day card in the front end.
- Add streaming to `/api/ai/chat` (Anthropic supports SSE streaming) for a typing effect instead of a single reply.

## Notes

- Frontend is plain HTML/CSS/JS — no build step, no framework.
- All prices are illustrative demo data, not live fares.
- This is a learning/demo scaffold, not production-hardened: add input validation, rate limiting, and HTTPS/auth before deploying for real.
