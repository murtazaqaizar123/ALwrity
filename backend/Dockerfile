# --- Stage 1: Build the Frontend ---
FROM node:20-slim AS frontend-builder
WORKDIR /app/frontend
COPY frontend/package*.json ./
RUN npm install
COPY frontend/ ./
# This creates the 'dist' folder
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

# Copy the built frontend from Stage 1
# We'll put it inside the backend folder so FastAPI can serve it, 
# or just keep it separate if you're running two processes.
COPY --from=frontend-builder /app/frontend/dist ./frontend/dist

# Expose the port (Alwrity backend usually runs on 8000)
EXPOSE 8000

# Start script: We need to run the backend. 
# Note: To run both in one container, we'll use a simple shell command.
CMD ["python", "backend/start_alwrity_backend.py"]
