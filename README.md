# Score Pulse

A real-time sports match tracking app with live scores and commentary. The project consists of a **React/Vite client** and a **Node.js/Express server** with WebSocket support and PostgreSQL.

---

## Prerequisites

- **Node.js** (v18+)
- **PostgreSQL** (for the server database)
- **npm** or **pnpm**

---

## Quick Start

### 1. Database Setup

Create a PostgreSQL database and run migrations:

```bash
# Create database (psql or pgAdmin)
createdb score_pulse

# Navigate to server
cd server

# Copy env example and set DATABASE_URL
cp .env.example .env
# Edit .env and set: DATABASE_URL=postgresql://user:password@localhost:5432/score_pulse

# Install dependencies
npm install

# Run migrations
npm run db:migrate
```

### 2. Start the Server

```bash
cd server
npm install
npm run dev
```

Server runs at **http://localhost:8000** (REST API and WebSocket at `ws://localhost:8000/ws`).

### 3. Start the Client

In a new terminal:

```bash
cd client
npm install
npm run dev
```

Client runs at **http://localhost:3000** (or the port shown in the terminal).

### 4. Connect Client to Server

The client is pre-configured to talk to `http://localhost:8000`. In `client/.env`:

```
VITE_API_BASE_URL="http://localhost:8000"
VITE_WS_BASE_URL="ws://localhost:8000/ws"
```

These are already set for local development.

---

## Project Structure

```
score-pulse/
├── client/          # React + Vite frontend
│   ├── .env         # API/WS URLs (points to server)
│   ├── src/
│   └── package.json
├── server/          # Node.js + Express + WebSocket backend
│   ├── .env         # DATABASE_URL (required)
│   ├── src/
│   └── package.json
└── README.md
```

---

## API Endpoints

| Method | Endpoint                            | Description                               |
| ------ | ----------------------------------- | ----------------------------------------- |
| GET    | `/matches?limit=50`                 | List matches                              |
| POST   | `/matches`                          | Create match                              |
| GET    | `/matches/:id/commentary?limit=100` | Get commentary for a match                |
| POST   | `/matches/:id/commentary`           | Add commentary (broadcasts via WebSocket) |

## WebSocket

- **URL:** `ws://localhost:8000/ws`
- **Messages (client → server):** `subscribe`, `unsubscribe`, `setSubscriptions`
- **Events (server → client):** `welcome`, `subscribed`, `commentary`, `match_created`, etc.

---

## Environment Variables

### Server (`server/.env`)

| Variable       | Required | Description                  |
| -------------- | -------- | ---------------------------- |
| `DATABASE_URL` | Yes      | PostgreSQL connection string |
| `PORT`         | No       | Server port (default: 8000)  |
| `HOST`         | No       | Bind host (default: 0.0.0.0) |

### Client (`client/.env`)

| Variable            | Required | Description                                     |
| ------------------- | -------- | ----------------------------------------------- |
| `VITE_API_BASE_URL` | No       | REST API base (default: http://localhost:8000)  |
| `VITE_WS_BASE_URL`  | No       | WebSocket URL (default: ws://localhost:8000/ws) |

---

## Seed Data (Live Demo)

Seed matches and commentary from `server/data.json` so data updates in real time over WebSocket. **Start the server first**, then run:

```bash
cd server
# Ensure API_URL is set in .env (e.g. API_URL=http://localhost:8000)
npm run seed
```

This will:

1. Create matches from `data.json` (adjusts times so they appear "live" when `SEED_FORCE_LIVE=1`)
2. Stream commentary entries every ~250ms (`DELAY_MS`)
3. Update scores in real time when goals/points occur (via `score_update` WebSocket events)

Tune in `server/.env`:

- `API_URL` – server base URL (required for seed)
- `SEED_FORCE_LIVE=1` – make matches appear live
- `DELAY_MS=250` – delay between commentary updates (lower = faster)

---

## Production / Remote Server

To connect the client to a remote server (e.g. ngrok or hosted API):

1. Update `client/.env`:
   ```
   VITE_API_BASE_URL="https://your-api.example.com"
   VITE_WS_BASE_URL="wss://your-api.example.com/ws"
   ```
2. Rebuild the client: `cd client && npm run build`

---

## Troubleshooting

| Issue                         | Solution                                                   |
| ----------------------------- | ---------------------------------------------------------- |
| `DATABASE_URL is not defined` | Add `DATABASE_URL` to `server/.env`                        |
| CORS errors                   | Server uses `cors`; ensure client URL is correct           |
| WebSocket won't connect       | Check `VITE_WS_BASE_URL` and that the server is running    |
| No matches shown              | Run migrations, then add matches via API or Drizzle Studio |
