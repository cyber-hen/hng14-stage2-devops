# Stage 1: Build
FROM node:20-slim AS builder
WORKDIR /build
COPY package*.json ./
RUN npm install
COPY . .

# Stage 2: Final Runtime
FROM node:20-slim
ENV NODE_ENV=production
WORKDIR /app

# Install curl and create non-root user
RUN apt-get update && apt-get install -y --no-install-recommends curl \
    && groupadd -r hnggroup && useradd -r -g hnggroup hnguser

# Copy only production dependencies
COPY package*.json ./
RUN npm install --only=production
COPY --from=builder /build .

RUN chown -R hnguser:hnggroup /app
USER hnguser

# Healthcheck (Requirement #2)
HEALTHCHECK --interval=10s --timeout=5s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

EXPOSE 3000
CMD ["node", "app.js"]