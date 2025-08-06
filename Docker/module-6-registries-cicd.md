# 🐳 Module 6: Docker Hub, Registries & CI/CD

## 📚 Learning Objectives (45 minutes)

By the end of this module, you will:
- Understand Docker Hub and container registries
- Create and manage Docker Hub repositories
- Push and pull images from registries
- Implement automated builds and webhooks
- Set up CI/CD pipelines with GitHub Actions
- Deploy applications automatically
- Manage image versioning and tagging strategies
- Share images and collaborate with teams

---

## 🏢 Understanding Docker Registries

### What is a Container Registry?

A **container registry** is a repository for storing and distributing Docker images. It's like GitHub for Docker images - a centralized place to store, version, and share container images.

### Types of Registries

```
┌─────────────────────────────────────┐
│            REGISTRY TYPES           │
├─────────────────────────────────────┤
│  PUBLIC REGISTRIES                  │
│  ├─ Docker Hub (docker.io)         │
│  ├─ GitHub Container Registry       │
│  ├─ Quay.io                        │
│  └─ Google Container Registry       │
├─────────────────────────────────────┤
│  PRIVATE REGISTRIES                 │
│  ├─ AWS ECR                        │
│  ├─ Azure Container Registry       │
│  ├─ Harbor                         │
│  └─ Self-hosted Registry           │
├─────────────────────────────────────┤
│  HYBRID SOLUTIONS                   │
│  ├─ Docker Hub (private repos)     │
│  └─ Cloud provider registries      │
└─────────────────────────────────────┘
```

### Docker Hub - The Default Registry

**Docker Hub** is the world's largest container registry with:
- 📦 **100+ billion** image pulls
- 🏢 **13+ million** repositories
- 👥 **9+ million** developers
- 🆓 **Free tier** with unlimited public repositories
- 💰 **Paid plans** for private repositories and teams

---

## 🚀 Getting Started with Docker Hub

### Step 1: Create Docker Hub Account

1. **Visit**: https://hub.docker.com/
2. **Sign Up** with email or GitHub account
3. **Verify** your email address
4. **Choose** a memorable username (becomes part of image names)

### Step 2: Login from Command Line

```bash
# Login to Docker Hub
docker login

# Login with specific username
docker login -u your-username

# Login to specific registry
docker login registry.example.com

# Check logged-in user
docker info | grep Username
```

### Step 3: Create Your First Repository

**Via Web Interface:**
1. Go to https://hub.docker.com/
2. Click **"Create Repository"**
3. Choose **repository name** (e.g., `my-first-app`)
4. Set **visibility** (Public/Private)
5. Add **description** and README
6. Click **"Create"**

**Repository Naming Convention:**
```
[registry]/[namespace]/[repository]:[tag]

Examples:
docker.io/student/my-app:latest
docker.io/student/my-app:v1.0.0
ghcr.io/student/my-app:main
```

---

## 📤 Pushing Images to Docker Hub

### Building and Pushing Your First Image

**Step 1: Create a Simple Application**
```bash
mkdir my-first-app
cd my-first-app

# Create a simple Node.js app
cat > package.json << 'EOF'
{
  "name": "my-first-app",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
EOF

cat > app.js << 'EOF'
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Docker Hub!',
    version: '1.0.0',
    timestamp: new Date().toISOString(),
    hostname: require('os').hostname()
  });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
EOF

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application files
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodeuser -u 1001 && \
    chown -R nodeuser:nodejs /app

USER nodeuser

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (res) => process.exit(res.statusCode === 200 ? 0 : 1))"

CMD ["npm", "start"]
EOF
```

**Step 2: Build and Tag the Image**
```bash
# Build the image (replace 'yourusername' with your Docker Hub username)
docker build -t yourusername/my-first-app:1.0.0 .

# Tag for latest as well
docker tag yourusername/my-first-app:1.0.0 yourusername/my-first-app:latest

# Verify images
docker images yourusername/my-first-app
```

**Step 3: Push to Docker Hub**
```bash
# Push specific version
docker push yourusername/my-first-app:1.0.0

# Push latest
docker push yourusername/my-first-app:latest

# Push all tags
docker push yourusername/my-first-app --all-tags
```

**Step 4: Verify and Test**
```bash
# Remove local images
docker rmi yourusername/my-first-app:1.0.0 yourusername/my-first-app:latest

# Pull and run from Docker Hub
docker run -d -p 3000:3000 --name test-app yourusername/my-first-app:latest

# Test the application
curl http://localhost:3000

# Clean up
docker stop test-app && docker rm test-app
```

---

## 🏷️ Image Tagging Strategies

### Semantic Versioning Strategy

```bash
# Major.Minor.Patch versioning
docker tag myapp:latest yourusername/myapp:1.0.0
docker tag myapp:latest yourusername/myapp:1.0
docker tag myapp:latest yourusername/myapp:1
docker tag myapp:latest yourusername/myapp:latest

# Push all versions
docker push yourusername/myapp:1.0.0
docker push yourusername/myapp:1.0
docker push yourusername/myapp:1
docker push yourusername/myapp:latest
```

### Environment-Based Tagging

```bash
# Environment-specific tags
docker tag myapp yourusername/myapp:dev
docker tag myapp yourusername/myapp:staging
docker tag myapp yourusername/myapp:production

# Feature branch tags
docker tag myapp yourusername/myapp:feature-auth
docker tag myapp yourusername/myapp:bugfix-login
```

### Git-Based Tagging

```bash
# Git commit hash
export GIT_COMMIT=$(git rev-parse --short HEAD)
docker tag myapp yourusername/myapp:$GIT_COMMIT

# Git branch name
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker tag myapp yourusername/myapp:$GIT_BRANCH

# Git tag
export GIT_TAG=$(git describe --tags --exact-match 2>/dev/null || echo "")
if [ ! -z "$GIT_TAG" ]; then
  docker tag myapp yourusername/myapp:$GIT_TAG
fi
```

---

## 🤖 Automated Builds with Docker Hub

### Setting Up Automated Builds

**Step 1: Connect GitHub Repository**
1. Go to Docker Hub → **Account Settings** → **Linked Accounts**
2. Connect your **GitHub account**
3. Authorize Docker Hub to access your repositories

**Step 2: Create Automated Build**
1. Go to **Create** → **Create Automated Build**
2. Choose **GitHub repository**
3. Configure **build settings**:
   - **Dockerfile location**: `/`
   - **Build context**: `/`
   - **Autobuild**: Enable
   - **Build caching**: Enable

**Step 3: Configure Build Rules**
```yaml
# Example build rules
Source Type: Branch
Source: main
Docker Tag: latest
Dockerfile location: Dockerfile
Build Context: /

Source Type: Tag
Source: /^v[0-9.]+$/
Docker Tag: {sourceref}
Dockerfile location: Dockerfile
Build Context: /
```

### Build Hooks (Advanced)

**Create `hooks/build` file:**
```bash
#!/bin/bash
echo "Building image with custom arguments"
docker build \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  --build-arg VCS_REF=$SOURCE_COMMIT \
  --build-arg VERSION=$SOURCE_BRANCH \
  -t $IMAGE_NAME .
```

**Create `hooks/post_build` file:**
```bash
#!/bin/bash
echo "Running post-build tasks"
docker run --rm $IMAGE_NAME npm test
```

---

## 🔄 CI/CD with GitHub Actions

### Basic GitHub Actions Workflow

**Create `.github/workflows/docker.yml`:**
```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [ main, develop ]
    tags: [ 'v*' ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: docker.io
  IMAGE_NAME: yourusername/my-app

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Log in to Docker Hub
      if: github.event_name != 'pull_request'
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
          type=semver,pattern={{major}}
          type=raw,value=latest,enable={{is_default_branch}}

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        platforms: linux/amd64,linux/arm64
        push: ${{ github.event_name != 'pull_request' }}
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

### Advanced CI/CD Pipeline

**Create `.github/workflows/ci-cd.yml`:**
```yaml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: docker.io
  IMAGE_NAME: yourusername/my-app

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run tests
      run: npm test
    
    - name: Run security audit
      run: npm audit --audit-level high

  build:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      image: ${{ steps.image.outputs.image }}
      digest: ${{ steps.build.outputs.digest }}
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Log in to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}

    - name: Build and push
      id: build
      uses: docker/build-push-action@v5
      with:
        context: .
        platforms: linux/amd64,linux/arm64
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

    - name: Output image
      id: image
      run: echo "image=${{ env.IMAGE_NAME }}:latest" >> $GITHUB_OUTPUT

  security-scan:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: ${{ needs.build.outputs.image }}
        format: 'sarif'
        output: 'trivy-results.sarif'

    - name: Upload Trivy scan results to GitHub Security tab
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: 'trivy-results.sarif'

  deploy:
    if: github.ref == 'refs/heads/main'
    needs: [build, security-scan]
    runs-on: ubuntu-latest
    environment: production
    
    steps:
    - name: Deploy to production
      run: |
        echo "Deploying ${{ needs.build.outputs.image }}@${{ needs.build.outputs.digest }}"
        # Add your deployment commands here
        # For example, update Kubernetes deployment, trigger webhook, etc.
```

### Setting Up GitHub Secrets

**Required secrets for Docker Hub:**
1. Go to your GitHub repository
2. **Settings** → **Secrets and variables** → **Actions**
3. Add repository secrets:
   - `DOCKER_USERNAME`: Your Docker Hub username
   - `DOCKER_PASSWORD`: Your Docker Hub password or access token

**Creating Docker Hub Access Token:**
1. Go to Docker Hub → **Account Settings** → **Security**
2. Click **New Access Token**
3. Give it a descriptive name (e.g., "GitHub Actions")
4. Copy the token and use it as `DOCKER_PASSWORD`

---

## 🌐 Alternative Registries

### GitHub Container Registry

```yaml
name: Build and Push to GHCR

on:
  push:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: |
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

### AWS Elastic Container Registry (ECR)

```yaml
name: Build and Push to ECR

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-west-2

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build, tag, and push image to Amazon ECR
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: my-app
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest
```

---

## 🔄 Multi-Registry Strategy

### Building for Multiple Registries

```yaml
name: Multi-Registry Build

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Login to GitHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and push to multiple registries
      uses: docker/build-push-action@v5
      with:
        context: .
        platforms: linux/amd64,linux/arm64
        push: true
        tags: |
          ${{ github.repository_owner }}/my-app:latest
          ${{ github.repository_owner }}/my-app:${{ github.sha }}
          ghcr.io/${{ github.repository }}:latest
          ghcr.io/${{ github.repository }}:${{ github.sha }}
```

---

## 📊 Real-world Example: Complete CI/CD for Web App

### Project Structure
```
web-app/
├── .github/workflows/
│   ├── ci.yml
│   └── deploy.yml
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app/
├── docker-compose.yml
├── docker-compose.prod.yml
└── README.md
```

### CI Pipeline (`.github/workflows/ci.yml`)

```yaml
name: Continuous Integration

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
  DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
  
jobs:
  test-frontend:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: frontend/package-lock.json
    
    - name: Install dependencies
      run: |
        cd frontend
        npm ci
    
    - name: Run tests
      run: |
        cd frontend
        npm run test:ci
    
    - name: Build frontend
      run: |
        cd frontend
        npm run build

  test-backend:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'
    
    - name: Install dependencies
      run: |
        cd backend
        pip install -r requirements.txt
        pip install pytest
    
    - name: Run tests
      run: |
        cd backend
        pytest tests/

  build-images:
    needs: [test-frontend, test-backend]
    runs-on: ubuntu-latest
    outputs:
      frontend-image: ${{ steps.meta-frontend.outputs.tags }}
      backend-image: ${{ steps.meta-backend.outputs.tags }}
    
    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ env.DOCKER_USERNAME }}
        password: ${{ env.DOCKER_PASSWORD }}

    - name: Extract metadata for frontend
      id: meta-frontend
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.DOCKER_USERNAME }}/webapp-frontend
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}

    - name: Build and push frontend
      uses: docker/build-push-action@v5
      with:
        context: ./frontend
        push: true
        tags: ${{ steps.meta-frontend.outputs.tags }}
        labels: ${{ steps.meta-frontend.outputs.labels }}
        cache-from: type=gha,scope=frontend
        cache-to: type=gha,mode=max,scope=frontend

    - name: Extract metadata for backend
      id: meta-backend
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.DOCKER_USERNAME }}/webapp-backend
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}

    - name: Build and push backend
      uses: docker/build-push-action@v5
      with:
        context: ./backend
        push: true
        tags: ${{ steps.meta-backend.outputs.tags }}
        labels: ${{ steps.meta-backend.outputs.labels }}
        cache-from: type=gha,scope=backend
        cache-to: type=gha,mode=max,scope=backend

  integration-test:
    needs: build-images
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Run integration tests
      run: |
        # Start services with latest images
        docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d
        
        # Wait for services to be ready
        sleep 30
        
        # Run integration tests
        docker-compose exec -T backend pytest tests/integration/
        
        # Cleanup
        docker-compose down
```

### Deployment Pipeline (`.github/workflows/deploy.yml`)

```yaml
name: Deploy to Production

on:
  workflow_run:
    workflows: ["Continuous Integration"]
    types:
      - completed
    branches: [main]

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    environment: production
    
    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Deploy to production server
      uses: appleboy/ssh-action@v1.0.0
      with:
        host: ${{ secrets.PROD_HOST }}
        username: ${{ secrets.PROD_USER }}
        key: ${{ secrets.PROD_SSH_KEY }}
        script: |
          cd /opt/webapp
          
          # Pull latest images
          docker-compose -f docker-compose.yml -f docker-compose.prod.yml pull
          
          # Rolling update
          docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
          
          # Health check
          sleep 30
          curl -f http://localhost/health || exit 1
          
          # Cleanup old images
          docker image prune -f

    - name: Send notification
      if: always()
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        text: |
          Deployment to production: ${{ job.status }}
          Commit: ${{ github.sha }}
          Author: ${{ github.actor }}
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## 🛡️ Security Best Practices

### Image Security Scanning

```yaml
name: Security Scan

on:
  schedule:
    - cron: '0 2 * * 1'  # Weekly scan
  workflow_dispatch:

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'yourusername/webapp:latest'
        format: 'table'
        exit-code: '1'  # Fail on HIGH/CRITICAL vulnerabilities
        severity: 'HIGH,CRITICAL'
```

### Secrets Management

```yaml
# Don't do this - secrets in plain text
environment:
  - DB_PASSWORD=supersecret123

# Do this - use secrets
environment:
  - DB_PASSWORD_FILE=/run/secrets/db_password
secrets:
  - db_password
```

### Base Image Security

```dockerfile
# Use specific versions, not latest
FROM node:18.17.0-alpine

# Use official images
FROM postgres:15.3-alpine

# Scan for vulnerabilities
FROM alpine:3.18
RUN apk add --no-cache --upgrade apk-tools
```

---

## 📋 Best Practices Summary

### 1. Image Naming and Tagging
```bash
# Good naming convention
yourusername/project-component:version
yourusername/webapp-frontend:v1.2.3
yourusername/webapp-backend:v1.2.3

# Consistent tagging strategy
:latest      # Latest stable version
:v1.2.3      # Specific version
:main        # Main branch build
:pr-123      # Pull request build
```

### 2. Multi-stage Builds for Production
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:18-alpine AS production
COPY --from=builder /app/node_modules ./node_modules
COPY . .
USER node
CMD ["npm", "start"]
```

### 3. Repository Organization
```
# Organize repositories logically
company/project-frontend
company/project-backend
company/project-database
company/shared-utils
```

### 4. Automated Quality Gates
- ✅ Run tests before building images
- ✅ Scan images for vulnerabilities
- ✅ Validate Dockerfile best practices
- ✅ Check for secrets in images
- ✅ Ensure multi-platform builds

---

## 📊 Module Summary

### What You've Learned:
✅ **Docker Hub fundamentals** and account setup  
✅ **Image pushing and pulling** workflows  
✅ **Automated builds** and webhooks  
✅ **CI/CD pipelines** with GitHub Actions  
✅ **Multi-registry strategies** and alternatives  
✅ **Security best practices** and scanning  
✅ **Real-world deployment** patterns  

### Key Skills Acquired:
- Create and manage Docker Hub repositories
- Implement automated CI/CD pipelines
- Configure multi-platform builds
- Set up security scanning and quality gates
- Deploy applications automatically
- Manage image versioning and tagging
- Integrate with multiple container registries

### CI/CD Workflow Mastery:
```yaml
# You now understand this complete flow:
Code Push → Tests → Build → Security Scan → Deploy → Monitor
```

---

## 🚀 What's Next?

You're now ready for the **final module**: **Module 7: Complete Project - AI-Powered Voting App** where you'll apply everything you've learned to build and deploy a real-world application.

### Practice Exercises:
1. Set up automated builds for multiple projects
2. Create a multi-stage CI/CD pipeline
3. Implement blue-green deployment strategy
4. Set up monitoring and alerting for deployments

### Advanced Topics to Explore:
- **Container orchestration** with Kubernetes
- **Service mesh** architecture
- **GitOps** deployment patterns
- **Infrastructure as Code** with Terraform

Excellent work! You now have the skills to share your Docker images with the world and implement professional CI/CD workflows! 🎉