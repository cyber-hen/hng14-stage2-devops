# HNG Stage 2: Microservices Orchestration

## 🚀 Project Scope
This project is a high-availability microservices architecture designed to handle asynchronous job processing. It leverages **Docker Compose** for orchestration and **GitHub Actions** for CI/CD. Unlike basic setups, this architecture focuses on service isolation and reliable communication between containers.

- **Frontend:** Node.js/Express (Port 3000) - Serves the interface and proxies user requests.
- **API:** FastAPI (Port 8000) - Manages job logic and communicates with the message broker.
- **Worker:** Python Background Processor - Consumes and executes long-running tasks from the queue.
- **Data Store:** Redis (Internal) - Acts as the high-speed message broker and task queue.

## 🛠️ Errors & Resolutions

### 1. Networking: "Connection Refused"
- **Issue:** The frontend could not reach the API because it was hardcoded to point to `localhost`. In a Dockerized environment, `localhost` refers to the container's own internal loopback interface, meaning the Frontend was searching for the API inside itself.
- **Resolution:** Implemented Docker Service Discovery. Refactored `app.js` to use an environment variable `API_URL`. By setting this to `http://api:8000`, Docker’s internal DNS resolves the request to the correct container across the bridge network.

### 2. Infrastructure: "Unhealthy Worker"
- **Issue:** The worker container was consistently flagged as `unhealthy` by Docker's healthcheck. Investigation revealed that the `python:3.12-slim` base image does not include `procps`, making the `pgrep` command unavailable.
- **Resolution:** Refactored the healthcheck in `docker-compose.yml` to use a native `CMD-SHELL` with `ps aux | grep python`, which is compatible with the slim image environment without requiring additional package overhead.

### 3. Security: CORS Policy
- **Issue:** Cross-Origin Resource Sharing (CORS) errors occurred when the browser attempted to facilitate requests between the Frontend (Port 3000) and the API (Port 8000).
- **Resolution:** Configured `CORSMiddleware` within the FastAPI application. I explicitly whitelisted the frontend origin and allowed the necessary HTTP methods (POST, GET) to ensure secure, authorized communication.

### 4. Dependency Sequencing: Redis Race Condition
- **Issue:** The API service would occasionally crash on initial startup because it attempted to connect to Redis before the Redis service was fully initialized.
- **Resolution:** Optimized the `docker-compose.yml` by using long-form `depends_on` syntax with `condition: service_healthy`. This ensures the API only attempts to boot once Redis has passed its own internal healthchecks.

## 🏁 How to Run
```bash
# Build and launch the orchestrated stack
docker-compose up -d --build

# Verify all containers have reached a (healthy) status
docker ps

# Perform an integration test
curl -X POST http://localhost:3000/submit
```