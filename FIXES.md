# Application Fixes Log - HNG Stage 2

This document details the bugs and architectural issues identified in the starter code and the specific steps taken to resolve them.

---

### 1. Hardcoded Infrastructure Hosts
- **File:** `api/main.py` (Line 7), `worker/worker.py` (Line 6), `frontend/app.js` (Line 7)
- **Problem:** All services used `localhost` or `127.0.0.1` to communicate. In a Docker environment, `localhost` refers to the container itself, not the shared network, causing `ConnectionRefused` errors.
- **Fix:** Replaced hardcoded strings with `os.getenv` (Python) and `process.env` (Node.js). Services now dynamically resolve hosts using Docker's internal DNS names (e.g., `redis` or `api`).

### 2. Unsafe Redis Data Decoding
- **File:** `api/main.py` (Line 18)
- **Problem:** The code attempted to `.decode()` a Redis response without checking if the key existed. If a job ID was invalid, the API would crash with an `AttributeError: 'NoneType' object has no attribute 'decode'`.
- **Fix:** Added a conditional check `if status:` before decoding. Also initialized the Redis client with `decode_responses=True` to handle string conversion at the driver level.

### 3. Missing API Observability (Healthchecks)
- **File:** `api/main.py`, `frontend/app.js`
- **Problem:** The services lacked a dedicated health endpoint. Docker and Orchestrators cannot verify if the application logic is "ready" or just "zombie" (running but unresponsive).
- **Fix:** Implemented a GET `/health` endpoint in both the FastAPI and Express services that returns a `200 OK` status.

### 4. Lack of Redis Connection Resilience
- **File:** `worker/worker.py`
- **Problem:** The worker script attempted to connect to Redis immediately upon execution. If the Redis container took longer to initialize than the Python script, the worker would crash and exit.
- **Fix:** Wrapped the Redis initialization in a retry loop with a backoff strategy, ensuring the worker waits for its dependency to be healthy before starting the job loop.

### 5. Dependency Version Instability
- **File:** `api/requirements.txt`, `worker/requirements.txt`
- **Problem:** Packages were listed without version pins (e.g., just `fastapi`). This leads to "Non-Deterministic Builds," where a future update to a library could break the production environment.
- **Fix:** Pinned all dependencies to specific, verified versions (e.g., `fastapi==0.109.0`) to ensure build consistency across all environments.

### 6. Missing CORS Middleware
- **File:** `api/main.py`
- **Problem:** In a multi-service architecture, the Frontend (port 3000) will be blocked by modern browsers when trying to request data from the API (port 8000) due to Cross-Origin Resource Sharing (CORS) policies.
- **Fix:** Imported `CORSMiddleware` from FastAPI and configured it to allow requests from the designated frontend origin.