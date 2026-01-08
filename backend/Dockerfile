# --- Stage 1: Build the Frontend ---
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

# Copy the built React frontend
COPY --from=frontend-builder /app/frontend/build ./frontend/build

# Pre-download spaCy and NLTK data to prevent runtime crashes/delays
RUN python -m spacy download en_core_web_sm
RUN python -m nltk.downloader punkt averaged_perceptron_tagger

# Expose the internal port mentioned in your logs
EXPOSE 3085

# Run from the backend directory
WORKDIR /app/backend
# We use 'python app.py' because start_alwrity_backend.py seems to be failing
CMD ["python", "app.py"]
