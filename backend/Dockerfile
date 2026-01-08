# --- Stage 1: Build the Frontend (React/CRA) ---
FROM node:20-slim AS frontend-builder
WORKDIR /app/frontend
COPY frontend/package*.json ./
ENV CI=false
RUN npm install
COPY frontend/ ./
RUN npm run build

# --- Stage 2: Final Image (Python 3.12) ---
FROM python:3.12-bookworm
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl build-essential gcc \
    && rm -rf /var/lib/apt/lists/*

# Upgrade pip and install requirements
RUN pip install --no-cache-dir --upgrade pip setuptools wheel
COPY backend/requirements.txt ./backend/
RUN pip install --no-cache-dir -r backend/requirements.txt

# Copy backend code
COPY backend/ ./backend/

# Copy the build from Stage 1 to the backend's static folder
# Note: Ensure your FastAPI/Flask app is configured to serve this directory
COPY --from=frontend-builder /app/frontend/build ./frontend/build

# Pre-download spaCy
RUN python -m spacy download en_core_web_sm

# Expose the internal port
EXPOSE 8000

# Set environment to ignore the duplicate index errors if possible
ENV PYTHONUNBUFFERED=1

# Run from the backend directory
WORKDIR /app/backend
CMD ["python", "app.py"]
