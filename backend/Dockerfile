# --- Stage 1: Build the Frontend (React/CRA) ---
FROM node:20-slim AS frontend-builder
WORKDIR /app/frontend
COPY frontend/package*.json ./

# Prevent build failure on ESLint warnings
ENV CI=false

RUN npm install
COPY frontend/ ./
# Build React app (outputs to /app/frontend/build)
RUN npm run build

# --- Stage 2: Final Image (Python Backend) ---
FROM python:3.11-bookworm
WORKDIR /app

# 1. Install system dependencies required for heavy Python libraries (numpy, pandas, etc.)
RUN apt-get update && apt-get install -y \
    curl \
    build-essential \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# 2. Upgrade pip tools for better dependency resolution
RUN pip install --no-cache-dir --upgrade pip setuptools wheel

# 3. Install Backend dependencies (done before copying code to use Docker cache)
COPY backend/requirements.txt ./backend/
RUN pip install --no-cache-dir -r backend/requirements.txt

# 4. Copy the backend source code
COPY backend/ ./backend/

# 5. Copy the built React frontend from Stage 1
# We keep it in /app/frontend/build so the backend can find it if configured
COPY --from=frontend-builder /app/frontend/build ./frontend/build

# 6. Expose the port used by Alwrity
EXPOSE 8000

# 7. CRITICAL FIX: Run the command from the backend folder
# This solves the "app.py not found" error
WORKDIR /app/backend
CMD ["python", "start_alwrity_backend.py"]
