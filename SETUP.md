# Setup Guide - Step by Step

This guide walks you through setting up the entire NBA Apps platform locally.

## Prerequisites Checklist

- [ ] Node.js 20+ installed (`node --version`)
- [ ] pnpm installed (`pnpm --version`) or npm/yarn
- [ ] Python 3.11+ installed (`python3 --version`)
- [ ] Docker and Docker Compose installed (`docker --version`, `docker compose version`)
- [ ] Git installed (`git --version`)

## Step-by-Step Setup

### Step 1: Clone Repositories

```bash
# Clone each repository
git clone <your-github>/nba-realtime-service
git clone <your-github>/nba-api-service
git clone <your-github>/nba-client-app

# Or if using a monorepo:
git clone <your-github>/nba-apps
cd nba-apps
```

### Step 2: Start Infrastructure Services

Open a terminal and navigate to `nba-realtime-service`:

```bash
cd nba-realtime-service
docker compose up -d redis redpanda postgres
```

**Wait for services to be healthy** (about 30 seconds). Verify:

```bash
# Check Redis
docker compose exec redis redis-cli ping
# Should return: PONG

# Check PostgreSQL
docker compose exec postgres pg_isready -U nba
# Should return: postgres:5432 - accepting connections

# Check Redpanda (Kafka)
curl http://localhost:9644/v1/status/ready
# Should return: {"status":"ready"}
```

### Step 3: Run Database Migrations

```bash
# Still in nba-realtime-service directory
docker compose exec nba-realtime-service npm run migrate

# Or if running locally:
npm install
npm run migrate
```

### Step 4: Configure and Start nba-realtime-service

```bash
# In nba-realtime-service directory
# Create .env file (see Environment Variables section in main README)

# Start the service
docker compose up -d nba-realtime-service

# Or run locally:
npm install
npm run dev
```

**Verify it's working:**
```bash
curl http://localhost:8000/health
# Should return: {"status":"healthy",...}
```

### Step 5: Configure and Start nba-api-service

Open a **new terminal**:

```bash
cd nba-api-service

# Create .env file
cat > .env << EOF
REDIS_HOST=localhost
REDIS_PORT=6379
KAFKA_BROKERS=localhost:19092
API_PORT=3000
CORS_ORIGIN=http://localhost:3001
EOF

# Install dependencies
npm install

# Start the service
npm run dev
```

**Verify it's working:**
```bash
curl http://localhost:3000/health
# Should return: {"status":"ok",...}
```

### Step 6: Configure and Start nba-client-app

Open a **new terminal**:

```bash
cd nba-client-app

# Create .env.local file
cat > .env.local << EOF
NEXT_PUBLIC_API_URL=http://localhost:3000
NEXT_PUBLIC_WS_URL=ws://localhost:3000
EOF

# Install dependencies
npm install

# Start the development server
npm run dev
```

**Verify it's working:**
- Open browser to http://localhost:3001
- You should see the NBA games schedule page

## Verification Checklist

- [ ] Infrastructure services running (Redis, Kafka, PostgreSQL)
- [ ] Database migrations completed
- [ ] nba-realtime-service running on port 8000
- [ ] nba-api-service running on port 3000
- [ ] nba-client-app running on port 3001
- [ ] Can access frontend at http://localhost:3001
- [ ] Can see game schedule (if games are scheduled today)

## Common Issues

### Port Already in Use

If you get "port already in use" errors:

```bash
# Find what's using the port
lsof -i :3000  # or :3001, :8000, etc.

# Kill the process
kill -9 <PID>
```

### Docker Services Won't Start

```bash
# Check Docker is running
docker ps

# Check service logs
docker compose logs redis
docker compose logs postgres
docker compose logs redpanda

# Restart services
docker compose restart
```

### No Game Data Appearing

1. **Check if games are scheduled today:**
   ```bash
   curl http://localhost:8000/schedule/$(date +%Y/%m/%d)
   ```

2. **Check nba-realtime-service logs:**
   ```bash
   docker compose logs -f nba-realtime-service
   ```

3. **Check Kafka messages:**
   - Open Redpanda Console: http://localhost:8080
   - Check topic: `nba.game.updates`

### WebSocket Connection Fails

1. **Check API service health:**
   ```bash
   curl http://localhost:3000/health
   ```

2. **Verify CORS configuration:**
   - Check `.env` file has `CORS_ORIGIN=http://localhost:3001`

3. **Check browser console** for WebSocket errors

## Next Steps

Once everything is running:

1. **Explore the API:**
   - View OpenAPI docs: `nba-api-service/openapi.yaml`
   - Test endpoints: `curl http://localhost:3000/api/schedules/2025-01-08`

2. **Test WebSocket:**
   - Open browser console on http://localhost:3001
   - Connect to WebSocket and subscribe to games

3. **Check Real-time Updates:**
   - Watch a live game (if in progress)
   - See updates appear in real-time

## Stopping Services

```bash
# Stop all Docker services
cd nba-realtime-service
docker compose down

# Stop Node.js services
# Press Ctrl+C in each terminal running npm run dev
```

## Clean Slate

To start completely fresh:

```bash
# Stop all services
docker compose down

# Remove volumes (WARNING: deletes all data)
docker compose down -v

# Remove node_modules (optional)
cd nba-realtime-service && rm -rf node_modules
cd ../nba-api-service && rm -rf node_modules
cd ../nba-client-app && rm -rf node_modules
```

Then start from Step 2 again.

