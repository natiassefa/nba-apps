# NBA Apps - Real-Time Game Data Platform

A microservices-based platform for real-time NBA game data, built with TypeScript, Python, Next.js, and modern infrastructure.

## 🏗️ Architecture Overview

This platform consists of three independent microservices that work together to provide real-time NBA game data:

```
┌───────────────────────┐
│  nba-realtime-service │
│  (TypeScript + Python)│
│  Port: 8000           │
└──────────┬────────────┘
           │
           │ Publishes updates
           ▼
    ┌───────────────┐
    │ Kafka/Redpanda│
    │  Port: 9092   │
    └──────┬────────┘
           │
           │ Consumes messages
           ▼
┌─────────────────────┐      ┌──────────────────┐
│  nba-api-service    │◄─────┤   Redis Cache    │
│  (TypeScript)       │      │   Port: 6379     │
│  Port: 3000         │      └──────────────────┘
└──────────┬──────────┘
           │
           │ REST API + WebSocket
           ▼
┌──────────────────────┐
│  nba-client-app      │
│  (Next.js/React)     │
│  Port: 3001          │
└──────────────────────┘
```

### Data Flow

1. **nba-realtime-service** polls NBA.com APIs via Python `nba_api` library
2. Detects changes using SHA256 hash comparison
3. Publishes updates to **Kafka** when data changes
4. **Kafka consumer** persists data to **Redis** (cache) and **PostgreSQL** (database)
5. **nba-api-service** consumes Kafka messages and broadcasts to WebSocket clients
6. **nba-client-app** displays data via REST API and receives real-time updates via WebSocket

## 📦 Repositories

This monorepo contains three main services, each designed to be deployed independently:

### 1. [nba-realtime-service](https://github.com/natiassefa/nba-realtime-service)

**Purpose**: Polls NBA.com APIs and publishes game updates to Kafka

**Tech Stack**:
- TypeScript/Node.js (main service)
- Python/FastAPI (bridge service wrapping `nba_api` library)
- Redis (caching)
- Kafka/Redpanda (message broker)
- PostgreSQL (persistent storage)

**Key Features**:
- Polls game summaries and play-by-play data
- ETag-based conditional requests
- Change detection via SHA256 hashing
- Dual persistence (Redis + PostgreSQL)
- Graceful shutdown and health checks

**Repository**: [`nba-realtime-service`](https://github.com/natiassefa/nba-realtime-service)

### 2. [nba-api-service](https://github.com/natiassefa/nba-api-service)

**Purpose**: REST API and WebSocket service for game data

**Tech Stack**:
- TypeScript/Node.js
- Express.js (REST API)
- WebSocket (real-time updates)
- Redis (data cache)
- Kafka (message consumption)

**Key Features**:
- REST endpoints for schedules, games, summaries, play-by-play
- WebSocket server for real-time updates
- Kafka consumer for game update messages
- Cache-first with database fallback
- OpenAPI documentation

**Repository**: [`nba-api-service`](../nba-api-service)

### 3. [nba-client-app](https://github.com/natiassefa/nba-client-app)

**Purpose**: Frontend web application for viewing NBA games

**Tech Stack**:
- Next.js 16 (App Router)
- React 19
- TypeScript
- Tailwind CSS
- WebSocket client

**Key Features**:
- Game schedule viewer
- Real-time game updates
- Game detail pages with statistics
- Play-by-play event display
- Responsive design

**Repository**: [`nba-client-app`](https://github.com/natiassefa/nba-client-app)

## 🚀 Quick Start

### Prerequisites

- **Node.js** 20+ and **pnpm** (or npm/yarn)
- **Python** 3.11+ (for nba-realtime-service)
- **Docker** and **Docker Compose** (for infrastructure services)
- **Git**

### Option 1: Run Everything with Docker Compose (Recommended)

The easiest way to get started is using Docker Compose for infrastructure services:

#### Step 1: Start Infrastructure Services

From `nba-realtime-service` directory:

```bash
cd nba-realtime-service
docker compose up -d redis redpanda postgres
```

This starts:
- **Redis** on port `6379`
- **Redpanda** (Kafka) on ports `9092` (internal) and `19092` (external)
- **PostgreSQL** on port `5433`

#### Step 2: Run Database Migrations

```bash
cd nba-realtime-service
docker compose exec nba-realtime-service pnpm run migrate

```

Or if running locally:

```bash
cd nba-realtime-service
pnpm run migrate

```

#### Step 3: Start nba-realtime-service

```bash
cd nba-realtime-service
docker compose up -d nba-realtime-service

# Or run locally:
pnpm install
pnpm run dev

```

#### Step 4: Start nba-api-service

In a new terminal:

```bash
cd nba-api-service
pnpm install


# Create .env file (see Environment Variables below)
pnpm run dev

```

#### Step 5: Start nba-client-app

In a new terminal:

```bash
cd nba-client-app
pnpm install
pnpm run dev

```

#### Step 6: Access the Application

- **Frontend**: http://localhost:3001
- **API Service**: http://localhost:3000
- **API Health**: http://localhost:3000/health
- **Realtime Service**: http://localhost:8000/health

### Option 2: Run Everything Locally (Advanced)

If you prefer to run services locally without Docker:

#### Infrastructure Setup

You'll need to install and run:
- Redis (port 6379)
- Kafka/Redpanda (port 9092)
- PostgreSQL (port 5433)

#### Service Setup

Each service can be run independently. See individual READMEs for details:
- [nba-realtime-service README](../nba-realtime-service/README.md)
- [nba-api-service README](../nba-api-service/README.md)
- [nba-client-app README](../nba-client-app/README.md)

## ⚙️ Environment Variables

### nba-realtime-service

Create `.env` file in `nba-realtime-service/`:

```bash
# NBA API Bridge (runs in same container)
NBA_API_BRIDGE_URL=http://localhost:8000

# Redis
REDIS_URL=redis://localhost:6379

# Kafka/Redpanda
KAFKA_BROKERS=localhost:19092  # Use 19092 for external access

# PostgreSQL
DB_HOST=localhost
DB_PORT=5433
DB_USER=nba
DB_PASSWORD=nba
DB_NAME=nba

# Polling
POLLING_BASE_INTERVAL_MS=20000
POLLING_SCHEDULE_DATE= # Optional: YYYY-MM-DD, empty = today

```

### nba-api-service

Create `.env` file in `nba-api-service/`:

```bash
# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# Kafka
KAFKA_BROKERS=localhost:19092  # Use 19092 for external access
KAFKA_CLIENT_ID=nba-api-service
KAFKA_TOPIC_UPDATES=nba.game.updates
KAFKA_CONSUMER_GROUP_ID=nba-api-consumer-group

# API
API_PORT=3000
CORS_ORIGIN=http://localhost:3001

# Database (optional - for fallback)
DB_HOST=localhost
DB_PORT=5433
DB_USER=nba
DB_PASSWORD=nba
DB_NAME=nba

# Logging
LOG_LEVEL=info
```

### nba-client-app

Create `.env.local` file in `nba-client-app/`:

```bash
NEXT_PUBLIC_API_URL=http://localhost:3000
NEXT_PUBLIC_WS_URL=ws://localhost:3000
```

## 🧪 Testing

### Test Infrastructure

```bash
# Test Redis
redis-cli ping

# Test Kafka (requires kafka client tools)
# Or use Redpanda Console at http://localhost:8080

# Test PostgreSQL
psql -h localhost -p 5433 -U nba -d nba -c "SELECT 1"
```

### Test Services

```bash
# Test nba-realtime-service Python bridge
curl http://localhost:8000/health

# Test nba-api-service
curl http://localhost:3000/health

# Test WebSocket (use a WebSocket client or browser console)
# Connect to: ws://localhost:3000/ws
```

## 📚 Documentation

### API Documentation

- **OpenAPI Spec**: [`nba-api-service/openapi.yaml`](../nba-api-service/openapi.yaml)
- View with [Swagger Editor](https://editor.swagger.io/) or [Redoc](https://redocly.github.io/redoc/)

### WebSocket API

Connect to `ws://localhost:3000/ws`:

**Subscribe to game:**
```json
{"type": "subscribe", "gameId": "game-uuid"}
```

**Subscribe to all games:**
```json
{"type": "subscribe", "all": true}
```

**Receive updates:**
```json
{
  "type": "gameUpdate",
  "gameId": "game-uuid",
  "eventType": "summary",
  "payload": {...},
  "timestamp": "2025-01-08T12:00:00.000Z"
}
```

## 🏛️ Architecture Decisions

### Why Microservices?

- **Independent Scaling**: Each service can scale based on demand
- **Technology Flexibility**: Use best tool for each job (Python for NBA API, TypeScript for services)
- **Independent Deployment**: Deploy services separately without affecting others
- **Fault Isolation**: Failures in one service don't cascade

### Why Kafka?

- **Decoupling**: Services communicate asynchronously
- **Reliability**: Messages are persisted and can be replayed
- **Scalability**: Handle high message throughput
- **Event Sourcing**: All game updates are events

### Why Redis + PostgreSQL?

- **Redis**: Fast cache for hot data (current games)
- **PostgreSQL**: Persistent storage for historical data
- **Cache-Aside Pattern**: Check cache first, fallback to database

## 🔧 Development

### Project Structure

```
nba-apps/
├── nba-realtime-service/    # Data polling service
│   ├── src/                  # TypeScript source
│   ├── python-bridge/        # Python FastAPI service
│   ├── docker-compose.yml    # Infrastructure services
│   └── README.md
├── nba-api-service/          # REST API + WebSocket service
│   ├── src/                  # TypeScript source
│   ├── docker-compose.yml    # Optional: standalone setup
│   └── README.md
├── nba-client-app/           # Next.js frontend
│   ├── app/                  # Next.js App Router
│   ├── components/           # React components
│   └── README.md
└── README.md                 # This file
```

### Common Commands

```bash
# Install dependencies (each service)
pnpm install

# Run in development mode
pnpm run dev

# Build for production
pnpm run build

# Run tests
pnpm test

# Run linting
pnpm run lint
```

## 🐛 Troubleshooting

### Services Won't Start

1. **Check infrastructure services are running:**
   ```bash
   docker compose ps  # In nba-realtime-service directory
   ```

2. **Check ports are available:**
   - Redis: 6379
   - Kafka: 9092, 19092
   - PostgreSQL: 5433
   - API Service: 3000
   - Client App: 3001
   - Realtime Service: 8000

3. **Check environment variables:**
   - Ensure `.env` files are configured correctly
   - Use `localhost:19092` for Kafka when running locally (not `redpanda:9092`)

### No Game Data

1. **Check nba-realtime-service logs:**
   ```bash
   docker compose logs -f nba-realtime-service
   ```

2. **Verify NBA API bridge is working:**
   ```bash
   curl http://localhost:8000/health/detailed
   ```

3. **Check Kafka messages:**
   - Use Redpanda Console at http://localhost:8080
   - Or check Kafka topic: `nba.game.updates`

### WebSocket Not Connecting

1. **Check API service is running:**
   ```bash
   curl http://localhost:3000/health
   ```

2. **Verify CORS is configured:**
   - Check `CORS_ORIGIN` in `nba-api-service/.env`
   - Should match client app URL (http://localhost:3001)

3. **Check browser console** for WebSocket connection errors

## 📝 License

See individual repository READMEs for license information.

## ⚠️ Important Legal Notice

**This software uses NBA.com data via the `https://github.com/swar/nba_api` Python library.**


**Commercial Use**: NBA.com's Terms of Use prohibit commercial use of their data without written permission. If you plan to use this software commercially, you must:
1. Obtain written permission from the NBA, OR
2. Use a licensed data provider (e.g., Sportradar, Stats Perform)


## 🤝 Contributing

Each service is in its own repository. Please contribute to the individual repositories:
- [nba-realtime-service](https://github.com/natiassefa/nba-realtime-service)
- [nba-api-service](https://github.com/natiassefa/nba-api-service)
- [nba-client-app](https://github.com/natiassefa/nba-client-app)

## 📞 Support

For issues or questions:
1. Check the individual service READMEs
2. Review the troubleshooting section above
3. Open an issue in the relevant repository

---

**Built with**: TypeScript, Python, Next.js, Redis, Kafka, PostgreSQL, Docker

