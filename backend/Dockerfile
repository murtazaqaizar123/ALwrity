# --- Stage 1: Build the Frontend ---
FROM node:20-slim AS frontend-builder
WORKDIR /app/frontend
COPY frontend/package*.json ./

# Disable 'CI=true' so the build doesn't fail on ESLint warnings
ENV CI=false

RUN npm install
COPY frontend/ ./
# CRA outputs to 'build', not 'dist'
RUN npm run build

# --- Stage 2: Setup the Backend & Final Image ---
FROM python:3.11-slim
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Install Backend dependencies
COPY backend/requirements.txt ./backend/
RUN pip install --no-cache-dir -r backend/requirements.txt

# Copy backend code
COPY backend/ ./backend/

# FIX: Copy from 'build' instead of 'dist'
COPY --from=frontend-builder /app/frontend/build ./frontend/build

# Expose the port
EXPOSE 8000

# Start the application
CMD ["python", "backend/start_alwrity_backend.py"]
