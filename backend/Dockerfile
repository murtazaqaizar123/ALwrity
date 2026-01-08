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
# CRITICAL FIX: Use Python 3.12 to allow backslashes in f-strings
FROM python:3.12-bookworm
WORKDIR /app

# 1. Install system dependencies required for heavy Python libraries
RUN apt-get update && apt-get install -y \
    curl \
    build-essential \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# 2. Upgrade pip tools for better dependency resolution
RUN pip install --no-cache-dir --upgrade pip setuptools wheel

# 3. Install Backend dependencies
COPY backend/requirements.txt ./backend/
RUN pip install --no-cache-dir -r backend/requirements.txt

# 4. Copy the backend source code
COPY backend/ ./backend/

# 5. Copy the built React frontend from Stage 1
COPY --from=frontend-builder /app/frontend/build ./frontend/build

# 6. Pre-download spaCy model to prevent runtime download delays
RUN python -m spacy download en_core_web_sm

# 7. Expose the port
EXPOSE 8000

# 8. CRITICAL FIX: Run the command from the backend folder 
# This solves the "app.py not found" error
WORKDIR /app/backend
CMD ["python", "start_alwrity_backend.py"]
