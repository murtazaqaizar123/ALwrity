# --- Stage 1: Build the Frontend ---
FROM node:20-slim AS frontend-builder
WORKDIR /app/frontend
COPY frontend/package*.json ./
ENV CI=false
RUN npm install
COPY frontend/ ./
RUN npm run build

# --- Stage 2: Setup the Backend & Final Image ---
# Using 'bookworm' instead of 'slim' to avoid compilation errors on small servers
FROM python:3.11-bookworm
WORKDIR /app

# Install system dependencies needed for heavy Python packages
RUN apt-get update && apt-get install -y \
    curl \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Upgrade pip to the latest version to handle complex dependency resolution
RUN pip install --upgrade pip setuptools wheel

# Install Backend dependencies
COPY backend/requirements.txt ./backend/
# We install these one by one or with a higher timeout if needed
RUN pip install --no-cache-dir -r backend/requirements.txt

# Copy backend code
COPY backend/ ./backend/

# Copy the build from Stage 1
COPY --from=frontend-builder /app/frontend/build ./frontend/build

EXPOSE 8000

CMD ["python", "backend/start_alwrity_backend.py"]
