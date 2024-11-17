```
# Stage 1: Build the Python application
FROM python:3.9-slim AS builder
# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
# Create a working directory
WORKDIR /app
# Install build dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc libpq-dev
# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt
# Copy the application source code
COPY . .
# Stage 2: Create the final runtime image
FROM python:3.9-slim
# Set environment variables for production
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
# Create a user to run the application
RUN adduser --disabled-password appuser
# Set working directory
WORKDIR /app
# Copy only necessary files from the builder stage
COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin
COPY --from=builder /app /app

# Change ownership to the appuser
RUN chown -R appuser:appuser /app
# Switch to the appuser
USER appuser
# Expose the application port
EXPOSE 8000
# Run the application
CMD ["python", "app.py"].dockerignore -> create this file within Dockerfile directory, it ignores files for command COPY . . 

# Before building create requriements.txt
vi requriements.txt
Flask==2.0.3
gunicorn==20.1.0
requests==2.26.0
psycopg2==2.9.3
```
