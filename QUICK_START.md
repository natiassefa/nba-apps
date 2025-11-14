# Quick Start Guide

Get up and running in 5 minutes!

## TL;DR

```bash
# 1. Start infrastructure
cd nba-realtime-service
docker compose up -d redis redpanda postgres

# 2. Run migrations
docker compose exec nba-realtime-service npm run migrate

# 3. Start services (in separate terminals)
cd nba-realtime-service && npm run dev
cd nba-api-service && npm run dev
cd nba-client-app && npm run dev

# 4. Open http://localhost:3001
```

## Detailed Steps

### 1. Prerequisites

Make sure you have:
- Node.js 20+
- Docker & Docker Compose
- Python 3.11+ (for nba-realtime-service)

### 2. Clone Repositories

```bash
git clone <repo-url>/nba-realtime-service
git clone <repo-url>/nba-api-service
git clone <repo-url>/nba-client-app
```

### 3. Start Infrastructure

```bash
cd nba-realtime-service
docker compose up -d redis redpanda postgres
```

### 4. Setup Database

```bash
docker compose exec nba-realtime-service npm run migrate
```

### 5. Start Services

**Terminal 1 - Realtime Service:**
```bash
cd nba-realtime-service
npm install
npm run dev
```

**Terminal 2 - API Service:**
```bash
cd nba-api-service
npm install
# Create .env with: KAFKA_BROKERS=localhost:19092
npm run dev
```

**Terminal 3 - Client App:**
```bash
cd nba-client-app
npm install
# Create .env.local with: NEXT_PUBLIC_API_URL=http://localhost:3000
npm run dev
```

### 6. Access Application

Open http://localhost:3001 in your browser!

## Environment Variables Quick Reference

### nba-api-service/.env
```bash
KAFKA_BROKERS=localhost:19092
REDIS_HOST=localhost
REDIS_PORT=6379
API_PORT=3000
CORS_ORIGIN=http://localhost:3001
```

### nba-client-app/.env.local
```bash
NEXT_PUBLIC_API_URL=http://localhost:3000
NEXT_PUBLIC_WS_URL=ws://localhost:3000
```

## Troubleshooting

**Port already in use?**
```bash
lsof -i :3000  # Find process
kill -9 <PID>  # Kill it
```

**Services won't start?**
```bash
# Check infrastructure
docker compose ps

# Check logs
docker compose logs -f nba-realtime-service
```

**No game data?**
- Check if games are scheduled today
- Check nba-realtime-service logs
- Verify Kafka messages in Redpanda Console (http://localhost:8080)

For more details, see [SETUP.md](./SETUP.md) or [README.md](./README.md).

