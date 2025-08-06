# 🐳 Module 4: Docker Images & Dockerfiles

## 📚 Learning Objectives (45 minutes)

By the end of this module, you will:
- Understand Docker images and layers
- Write effective Dockerfiles
- Build custom Docker images
- Optimize image size and build time
- Tag and version images
- Implement multi-stage builds
- Share images via registries

---

## 🖼️ Understanding Docker Images

### What are Docker Images?

**Docker images are read-only templates** used to create containers. Think of them as:
- **Blueprints** for containers
- **Snapshots** of a filesystem and application
- **Layered filesystems** stacked on top of each other
- **Immutable artifacts** that can be versioned and shared

### Image Layers

Docker images are built in **layers**, where each layer represents a change to the filesystem:

```
┌─────────────────────────────────────┐ ← Container Layer (writable)
├─────────────────────────────────────┤ ← Application Layer
├─────────────────────────────────────┤ ← Dependencies Layer  
├─────────────────────────────────────┤ ← Runtime Layer
├─────────────────────────────────────┤ ← OS Layer
└─────────────────────────────────────┘ ← Base Layer
```

**Benefits of Layered Architecture:**
- **Storage efficiency**: Shared layers between images
- **Network efficiency**: Only changed layers are downloaded
- **Build cache**: Unchanged layers are reused
- **Rollback capability**: Easy to revert changes

### Examining Image Layers

```bash
# View image layers
docker history <image>

# Example: Inspect Ubuntu image layers
docker history ubuntu:20.04

# View detailed image information
docker inspect ubuntu:20.04

# Check image size and layers
docker images
```

---

## 📝 Introduction to Dockerfiles

### What is a Dockerfile?

A **Dockerfile is a text file** containing instructions to build a Docker image automatically. It defines:
- **Base image** to start from
- **Commands** to run during build
- **Files** to copy into the image
- **Environment variables** to set
- **Ports** to expose
- **Default command** to run

### Dockerfile Syntax

```dockerfile
# Comment
INSTRUCTION arguments
```

**Key Rules:**
- Instructions are **case-insensitive** (but convention is UPPERCASE)
- Each instruction creates a **new layer**
- Instructions are executed in **order**
- First instruction must be `FROM`

---

## 🔧 Essential Dockerfile Instructions

### 1. FROM - Base Image
```dockerfile
# Specify base image
FROM ubuntu:20.04
FROM node:16
FROM python:3.9

# Multi-stage builds
FROM node:16 AS builder
FROM nginx:alpine AS production
```

### 2. RUN - Execute Commands
```dockerfile
# Run commands during build
RUN apt-get update && apt-get install -y curl

# Multiple commands (recommended format)
RUN apt-get update \
    && apt-get install -y \
       curl \
       wget \
       git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Each RUN creates a new layer
RUN echo "Layer 1"
RUN echo "Layer 2"  # Creates separate layer
```

### 3. COPY & ADD - Copy Files
```dockerfile
# Copy files from host to image
COPY source destination
COPY app.js /app/
COPY . /app/

# ADD has additional features (URLs, tar extraction)
ADD https://example.com/file.tar.gz /tmp/
ADD archive.tar.gz /app/  # Auto-extracts

# Best practice: Use COPY unless you need ADD features
COPY requirements.txt .
COPY src/ /app/src/
```

### 4. WORKDIR - Set Working Directory
```dockerfile
# Set working directory (creates if doesn't exist)
WORKDIR /app

# Equivalent to: RUN mkdir -p /app && cd /app
# All subsequent instructions use this directory
COPY . .
RUN npm install
```

### 5. ENV - Environment Variables
```dockerfile
# Set environment variables
ENV NODE_ENV=production
ENV PORT=3000
ENV PATH="/app/bin:$PATH"

# Multiple variables
ENV NODE_ENV=production \
    PORT=3000 \
    DEBUG=false
```

### 6. EXPOSE - Document Ports
```dockerfile
# Document which ports the container will listen on
EXPOSE 3000
EXPOSE 8080 8443

# This doesn't actually publish ports (use -p flag with docker run)
# It's documentation for developers
```

### 7. USER - Set User
```dockerfile
# Run commands as specific user
USER node
USER 1000
USER appuser

# Create user first
RUN adduser --disabled-password --gecos '' appuser
USER appuser
```

### 8. CMD & ENTRYPOINT - Default Commands
```dockerfile
# CMD - Default command (can be overridden)
CMD ["node", "app.js"]
CMD ["python", "app.py"]
CMD npm start

# ENTRYPOINT - Fixed command (args can be added)
ENTRYPOINT ["python", "app.py"]

# Combined usage
ENTRYPOINT ["python", "app.py"]
CMD ["--help"]  # Default argument

# Shell form vs Exec form
CMD echo "Hello World"           # Shell form
CMD ["echo", "Hello World"]      # Exec form (preferred)
```

### 9. VOLUME - Mount Points
```dockerfile
# Create mount points
VOLUME ["/data"]
VOLUME ["/var/log", "/var/db"]

# Data persists even after container removal
```

### 10. LABEL - Metadata
```dockerfile
# Add metadata to image
LABEL version="1.0"
LABEL description="This is a web application"
LABEL maintainer="student@university.edu"

# Multiple labels
LABEL version="1.0" \
      description="Web application" \
      maintainer="student@university.edu"
```

---

## 🏗️ Building Docker Images

### Basic Build Command
```bash
# Build image from current directory
docker build .

# Build with tag
docker build -t myapp:latest .

# Build with specific Dockerfile
docker build -f Dockerfile.dev -t myapp:dev .

# Build with build arguments
docker build --build-arg NODE_VERSION=16 -t myapp .

# Build with no cache
docker build --no-cache -t myapp .
```

### Build Context
The **build context** is the directory sent to Docker daemon:
```bash
# Current directory is build context
docker build .

# Specify different context
docker build /path/to/context

# Use .dockerignore to exclude files
echo "node_modules" > .dockerignore
echo "*.log" >> .dockerignore
```

---

## 🧪 Hands-on Lab: Building Your First Image

### Lab 1: Simple Node.js Application

**Step 1: Create Application Files**
```bash
# Create project directory
mkdir my-node-app
cd my-node-app

# Create package.json
cat > package.json << 'EOF'
{
  "name": "my-node-app",
  "version": "1.0.0",
  "description": "Simple Node.js app",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
EOF

# Create application file
cat > app.js << 'EOF'
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Docker!',
    timestamp: new Date().toISOString(),
    version: process.env.APP_VERSION || '1.0.0'
  });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
EOF
```

**Step 2: Create Dockerfile**
```dockerfile
# Use official Node.js runtime as base image
FROM node:16

# Set working directory in container
WORKDIR /app

# Copy package.json first (for better caching)
COPY package.json .

# Install dependencies
RUN npm install

# Copy application files
COPY . .

# Set environment variables
ENV NODE_ENV=production
ENV APP_VERSION=1.0.0

# Expose port
EXPOSE 3000

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser && \
    chown -R appuser:appuser /app
USER appuser

# Define default command
CMD ["npm", "start"]
```

**Step 3: Create .dockerignore**
```dockerignore
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
coverage
.nyc_output
```

**Step 4: Build and Test**
```bash
# Build the image
docker build -t my-node-app:1.0.0 .

# Run the container
docker run -d -p 3000:3000 --name node-app my-node-app:1.0.0

# Test the application
curl http://localhost:3000
curl http://localhost:3000/health

# Check logs
docker logs node-app

# Clean up
docker stop node-app && docker rm node-app
```

---

## 🐍 Lab 2: Python Flask Application

**Step 1: Create Python Application**
```bash
mkdir my-flask-app
cd my-flask-app

# Create requirements.txt
cat > requirements.txt << 'EOF'
Flask==2.3.0
gunicorn==21.2.0
EOF

# Create Flask app
cat > app.py << 'EOF'
from flask import Flask, jsonify
import os
from datetime import datetime

app = Flask(__name__)

@app.route('/')
def hello():
    return jsonify({
        'message': 'Hello from Flask in Docker!',
        'timestamp': datetime.now().isoformat(),
        'version': os.getenv('APP_VERSION', '1.0.0'),
        'environment': os.getenv('FLASK_ENV', 'production')
    })

@app.route('/health')
def health():
    return jsonify({'status': 'healthy'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=int(os.getenv('PORT', 5000)))
EOF
```

**Step 2: Create Optimized Dockerfile**
```dockerfile
# Use Python slim image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Create non-root user early
RUN adduser --disabled-password --gecos '' appuser

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Copy and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application files
COPY . .

# Change ownership to non-root user
RUN chown -R appuser:appuser /app
USER appuser

# Set environment variables
ENV FLASK_APP=app.py
ENV FLASK_ENV=production
ENV PORT=5000

# Expose port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1

# Use gunicorn for production
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

**Step 3: Build Multi-Architecture Image**
```bash
# Build the image
docker build -t my-flask-app:1.0.0 .

# Test locally
docker run -d -p 5000:5000 --name flask-app my-flask-app:1.0.0

# Test endpoints
curl http://localhost:5000
curl http://localhost:5000/health

# Check health status
docker inspect --format='{{.State.Health.Status}}' flask-app

# Clean up
docker stop flask-app && docker rm flask-app
```

---

## 🏷️ Image Tagging and Versioning

### Tagging Strategies
```bash
# Semantic versioning
docker build -t myapp:1.0.0 .
docker build -t myapp:1.0 .
docker build -t myapp:1 .
docker build -t myapp:latest .

# Environment-based tags
docker build -t myapp:dev .
docker build -t myapp:staging .
docker build -t myapp:prod .

# Git-based tags
docker build -t myapp:$(git rev-parse --short HEAD) .
docker build -t myapp:$(git describe --tags) .

# Date-based tags
docker build -t myapp:$(date +%Y%m%d) .
```

### Tag Management
```bash
# List images
docker images

# Tag existing image
docker tag myapp:1.0.0 myapp:latest
docker tag myapp:1.0.0 registry.example.com/myapp:1.0.0

# Remove tags
docker rmi myapp:old-tag

# View image tags
docker images myapp
```

---

## 🚀 Multi-Stage Builds

### Why Multi-Stage Builds?
- **Reduce image size**: Exclude build tools and dependencies
- **Security**: Fewer components in production image
- **Separation of concerns**: Build vs runtime environments

### Example: Node.js Multi-Stage Build
```dockerfile
# Stage 1: Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Production stage
FROM node:16-alpine AS production
WORKDIR /app

# Create user
RUN adduser --disabled-password --gecos '' appuser

# Copy only production dependencies and app
COPY --from=builder /app/node_modules ./node_modules
COPY . .

# Change ownership and switch user
RUN chown -R appuser:appuser /app
USER appuser

# Expose port and start app
EXPOSE 3000
CMD ["node", "app.js"]
```

### Example: Python Multi-Stage Build
```dockerfile
# Stage 1: Build dependencies
FROM python:3.9 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Production image
FROM python:3.9-slim AS production
WORKDIR /app

# Copy user packages from builder stage
COPY --from=builder /root/.local /root/.local

# Copy application
COPY . .

# Make sure scripts in .local are usable
ENV PATH=/root/.local/bin:$PATH

# Run application
CMD ["python", "app.py"]
```

### Build Specific Stage
```bash
# Build only the builder stage
docker build --target builder -t myapp:builder .

# Build production stage (default)
docker build -t myapp:prod .

# Compare image sizes
docker images myapp
```

---

## 📏 Image Optimization Best Practices

### 1. Use Appropriate Base Images
```dockerfile
# Heavy base image (avoid for production)
FROM ubuntu:20.04  # ~72MB

# Better alternatives
FROM alpine:3.16   # ~5MB
FROM python:3.9-slim  # ~45MB vs python:3.9 (~350MB)
FROM node:16-alpine   # ~40MB vs node:16 (~350MB)
```

### 2. Minimize Layers
```dockerfile
# Bad: Multiple RUN layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get clean

# Good: Single RUN layer
RUN apt-get update && \
    apt-get install -y curl wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 3. Order Instructions by Change Frequency
```dockerfile
# Rarely changing instructions first
FROM node:16-alpine
WORKDIR /app

# Dependencies (change less frequently)
COPY package*.json ./
RUN npm ci --only=production

# Application code (changes frequently)
COPY . .

# Runtime configuration
EXPOSE 3000
CMD ["node", "app.js"]
```

### 4. Use .dockerignore
```dockerignore
# Version control
.git
.gitignore

# Dependencies
node_modules
__pycache__
*.pyc

# Development files
.env
docker-compose.yml
Dockerfile
README.md

# IDE files
.vscode
.idea

# OS files
.DS_Store
Thumbs.db
```

### 5. Remove Unnecessary Files
```dockerfile
# Clean package manager cache
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Remove build dependencies
RUN apk add --no-cache --virtual .build-deps \
    gcc musl-dev && \
    pip install some-package && \
    apk del .build-deps
```

---

## 🔍 Image Analysis and Security

### Analyze Image Size
```bash
# Check image size
docker images

# Analyze layers
docker history myapp:latest

# Use dive tool for detailed analysis
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive:latest myapp:latest
```

### Security Scanning
```bash
# Docker Scout (built-in)
docker scout cves myapp:latest
docker scout recommendations myapp:latest

# Trivy scanner
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image myapp:latest
```

### Security Best Practices
```dockerfile
# Use specific versions
FROM node:16.20.0-alpine

# Don't run as root
RUN adduser --disabled-password appuser
USER appuser

# Use secrets for sensitive data
RUN --mount=type=secret,id=api_key \
    API_KEY=$(cat /run/secrets/api_key) && \
    # Use API_KEY without storing in layer

# Scan for vulnerabilities regularly
```

---

## 📋 Module Summary

### What You've Learned:
✅ **Docker image concepts** and layer architecture  
✅ **Dockerfile instructions** and best practices  
✅ **Image building process** and optimization  
✅ **Multi-stage builds** for production images  
✅ **Tagging strategies** and versioning  
✅ **Security considerations** and scanning  

### Key Skills Acquired:
- Write efficient Dockerfiles
- Build optimized Docker images
- Implement multi-stage builds
- Apply security best practices
- Manage image tags and versions
- Analyze and optimize image size

### Dockerfile Best Practices Summary:
1. **Use minimal base images** (Alpine, slim variants)
2. **Order instructions** by change frequency
3. **Combine RUN commands** to minimize layers
4. **Use .dockerignore** to exclude unnecessary files
5. **Run as non-root user** for security
6. **Use multi-stage builds** for production images
7. **Clean up** package caches and temporary files
8. **Pin versions** for reproducible builds

---

## 🚀 What's Next?

You're now ready for **Module 5: Docker Compose & Multi-Container Applications** where you'll learn to:
- Orchestrate multiple containers
- Define application stacks
- Configure networking between services
- Manage volumes and environment variables
- Scale applications horizontally

### Practice Exercises:
1. Create a Dockerfile for your favorite programming language
2. Build a multi-stage image for a web application
3. Optimize an existing Dockerfile for size and security
4. Implement health checks and proper tagging strategy

Excellent work on mastering Docker images and Dockerfiles! 🎉