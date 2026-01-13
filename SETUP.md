# Project Setup Guide

This guide explains how to start the workflow platform locally with PostgreSQL, backend, and frontend.

## Prerequisites

- **Docker** - for running PostgreSQL and Redis
- **Python 3.12+** and **uv** - for backend
- **Node.js** and **bun** (or npm) - for frontend

## 1. Start PostgreSQL with Docker

Run PostgreSQL in a Docker container:

```bash
docker run -d \
  --name workflows-postgres \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=workflow_platform \
  -p 5432:5432 \
  postgres:16
```

To stop the container later:
```bash
docker stop workflows-postgres
docker rm workflows-postgres
```

## 2. Start Redis with Docker

The backend uses Redis for the background job queue (arq). Run Redis in a Docker container:

```bash
docker run -d \
  --name workflows-redis \
  -p 6379:6379 \
  redis:7
```

To stop the container later:
```bash
docker stop workflows-redis
docker rm workflows-redis
```

## 3. Backend Setup

Navigate to the backend directory:

```bash
cd backend
```

### Install Dependencies

```bash
uv sync --all-extras
```

### Configure Environment

Create a `.env` file in the `backend/` directory:

```env
# Database
DATABASE_URL=postgresql+asyncpg://postgres:password@localhost:5432/workflow_platform

# JWT Secret (generate your own for production)
JWT_SECRET=your-secret-key-here

# Encryption Key (generate your own for production - must be 32 bytes, URL-safe base64 encoded)
ENCRYPTION_KEY=your-32-byte-encryption-key-here

# Google API (for workflow features)
GOOGLE_API_KEY=your-google-api-key
```

### Setting Up Credentials

The platform supports storing API credentials (encrypted) in the database for use in workflows.

#### Option 1: Using the UI/API (Recommended)

1. Start the backend and frontend
2. Sign up/login to the application
3. Navigate to **Credentials** section
4. Create a credential with:
   - **Name**: e.g., "My Telegram Bot"
   - **Type**: `telegram` (or `google_api_key` for LLM nodes)
   - **Value**: Your bot token or API key

5. When creating a workflow node, select the credential to use

#### Option 2: Using `.env.inject` (For Local Testing Only)

The `.env.inject` file is for **forced local testing** - it injects environment variables directly without saving to the database.

Create a `.env.inject` file in the `backend/` directory:

```env
TELEGRAM_API_KEY=your-telegram-bot-token
BOT_ID=your-bot-id
USER_ID=your-telegram-user-id
```

Then source it when running the backend:
```bash
source .env.inject && uv run uvicorn api.main:app --host 0.0.0.0 --port 8000 --reload
```

**Note:** `.env.inject` is NOT loaded automatically. It must be manually sourced. Use this only for quick local testing - for normal usage, credentials should be stored via the UI/API.

### Start the Backend

```bash
uv run uvicorn api.main:app --host 0.0.0.0 --port 8000 --reload
```

The backend will be available at **http://localhost:8000**

## 4. Frontend Setup

Open a new terminal and navigate to the frontend directory:

```bash
cd frontend
```

### Install Dependencies

```bash
bun install
# or: npm install
```

### Start the Frontend

```bash
bun dev
# or: npm run dev
```

The frontend will be available at **http://localhost:8080**

## Access the Application

Once everything is running:

- **Frontend**: http://localhost:8080
- **Backend API**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs

## Running Database Migrations

The backend automatically runs migrations on startup. To manually run migrations:

```bash
cd backend
uv run alembic upgrade head
```

## Troubleshooting

### PostgreSQL connection issues
- Ensure the Docker container is running: `docker ps`
- Check the DATABASE_URL in `.env` matches the Docker credentials

### Redis connection issues
- Ensure the Docker container is running: `docker ps`
- Verify Redis is accessible: `redis-cli ping` (returns `PONG`)

### Backend won't start
- Verify Python 3.12+ is installed
- Ensure all dependencies are installed with `uv sync`
- Check that both PostgreSQL and Redis containers are running

### Frontend can't connect to backend
- Check that the backend is running on port 8000
- Verify the API URL in `frontend/src/lib/api.ts`

### Credentials not working
- For `.env.inject`: Remember to source the file before running backend only in dev mode for testing by injecting secrets
- For DB credentials: Verify the credential type matches what the node expects (e.g., `telegram`, `google_api_key`)
