# 🐳 Module 2: Docker Installation & Setup

## 📚 Learning Objectives (30 minutes)

By the end of this module, you will:
- Install Docker on your operating system
- Verify Docker installation
- Understand Docker Desktop vs Docker Engine
- Set up Docker Hub account
- Run your first Docker container

---

## 🎯 Installation Options

### Option A: Online Docker Playground (Instant Start - Recommended)
### Option B: Docker Desktop (Windows/Mac)
### Option C: Docker Engine (Linux)

---

## 🌐 Option A: Docker Playground (No Installation Required)

**Perfect for learning and quick experimentation!**

### Steps:
1. **Visit:** https://labs.play-with-docker.com/
2. **Login** with Docker Hub account (create one if needed)
3. **Click "Start"** to begin session
4. **Add New Instance** - Get a Linux VM with Docker pre-installed
5. **Session lasts 4 hours** - Perfect for this course

### Advantages:
✅ **No installation required** - Start immediately  
✅ **Pre-configured environment** - Everything ready  
✅ **Multiple instances** - Test multi-container setups  
✅ **Internet accessible** - Share running containers  
✅ **Free to use** - No cost for learning  

### Quick Verification:
```bash
# In Play with Docker terminal
docker --version
docker run hello-world
```

---

## 💻 Option B: Docker Desktop (Windows/Mac)

**Best for development environment with GUI**

### System Requirements

**Windows:**
- Windows 10/11 (64-bit)
- Hyper-V enabled
- 4 GB RAM minimum
- Virtualization enabled in BIOS

**macOS:**
- macOS 10.15 or newer
- 4 GB RAM minimum
- Apple chip (M1/M2) or Intel

### Installation Steps

#### For Windows:
1. **Download Docker Desktop:**
   - Visit: https://www.docker.com/products/docker-desktop
   - Click "Download for Windows"

2. **Run Installer:**
   ```powershell
   # Run as Administrator
   Docker Desktop Installer.exe
   ```

3. **Configuration:**
   - Enable WSL 2 feature (recommended)
   - Enable Hyper-V if prompted
   - Restart computer when required

4. **First Launch:**
   - Start Docker Desktop from Start menu
   - Accept license agreement
   - Complete initial setup

#### For macOS:
1. **Download Docker Desktop:**
   - Visit: https://www.docker.com/products/docker-desktop
   - Choose Apple Chip (M1/M2) or Intel version

2. **Install:**
   ```bash
   # Drag Docker.app to Applications folder
   # Launch Docker Desktop
   ```

3. **Grant Permissions:**
   - Allow privileged access when prompted
   - Enter your macOS password

### Verification:
```bash
# Open Terminal/Command Prompt
docker --version
docker-compose --version
docker run hello-world
```

---

## 🐧 Option C: Docker Engine (Linux)

**Lightweight option for Linux users**

### Ubuntu/Debian Installation:
```bash
# Update package index
sudo apt-get update

# Install prerequisites
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up stable repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group (avoid using sudo)
sudo usermod -aG docker $USER

# Log out and back in, then test
docker --version
docker run hello-world
```

### CentOS/RHEL/Fedora:
```bash
# Install Docker using repository
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group
sudo usermod -aG docker $USER

# Test installation
docker --version
```

---

## 🔧 Post-Installation Setup

### 1. Create Docker Hub Account
```bash
# Visit: https://hub.docker.com/
# Click "Sign up"
# Verify email address
# Choose username (will be used for repositories)
```

### 2. Login to Docker Hub
```bash
# Login from command line
docker login

# Enter username and password
# Success message: "Login Succeeded"
```

### 3. Configure Docker (Optional)
```bash
# Check Docker info
docker info

# View current configuration
docker version

# Set resource limits (Docker Desktop)
# Settings > Resources > Advanced
# Memory: 4GB minimum
# CPUs: 2 minimum
```

---

## ✅ Installation Verification

### Basic Verification Tests:
```bash
# 1. Check Docker version
docker --version
# Expected: Docker version 20.10.x or higher

# 2. Check Docker Compose (if using Docker Desktop)
docker-compose --version
# Expected: docker-compose version 1.29.x or higher

# 3. Check Docker system info
docker info
# Should show server info without errors

# 4. Run hello-world container
docker run hello-world
# Should download and run successfully

# 5. List running containers
docker ps
# Should show empty list (no containers running)

# 6. List all containers (including stopped)
docker ps -a
# Should show hello-world container in exited state
```

### Advanced Verification:
```bash
# Test container networking
docker run -d -p 8080:80 nginx
curl http://localhost:8080
docker stop $(docker ps -q)

# Test volume mounting
docker run -v $(pwd):/workspace ubuntu ls /workspace

# Test image operations
docker images
docker pull alpine
docker images | grep alpine
```

---

## 🏃‍♂️ Your First Docker Container

Let's run a few containers to get familiar:

### 1. Hello World
```bash
# Run the classic hello-world
docker run hello-world

# What happened:
# 1. Docker looked for 'hello-world' image locally
# 2. Didn't find it, so downloaded from Docker Hub
# 3. Created a container from the image
# 4. Ran the container (printed message)
# 5. Container exited
```

### 2. Interactive Ubuntu Container
```bash
# Run Ubuntu container interactively
docker run -it ubuntu bash

# You're now inside the container!
# Try these commands:
whoami          # root
ls              # see filesystem
cat /etc/os-release  # Ubuntu version
apt update && apt install -y curl
curl --version
exit            # leave container
```

### 3. Web Server Container
```bash
# Run Nginx web server
docker run -d -p 8080:80 --name my-nginx nginx

# Verify it's running
docker ps

# Test the web server
curl http://localhost:8080
# Or open browser to http://localhost:8080

# Stop the container
docker stop my-nginx

# Remove the container
docker rm my-nginx
```

### 4. Database Container
```bash
# Run PostgreSQL database
docker run -d \
  --name my-postgres \
  -e POSTGRES_DB=testdb \
  -e POSTGRES_USER=student \
  -e POSTGRES_PASSWORD=password123 \
  -p 5432:5432 \
  postgres:13

# Check if it's running
docker ps

# Connect to database (if you have psql installed)
# psql -h localhost -U student -d testdb

# Clean up
docker stop my-postgres
docker rm my-postgres
```

---

## 🔧 Docker Desktop GUI (Windows/Mac Users)

### Dashboard Overview:
- **Containers/Apps**: View running containers
- **Images**: Manage downloaded images  
- **Volumes**: Handle data persistence
- **Dev Environments**: Development workspaces

### Settings Configuration:
1. **Resources**: Adjust CPU, Memory, Disk
2. **Docker Engine**: Advanced daemon settings
3. **Kubernetes**: Enable local Kubernetes cluster
4. **Extensions**: Add third-party tools

### Useful Features:
- **Container logs**: View output in GUI
- **Terminal access**: Open shell in containers
- **Port forwarding**: Easy port management
- **Volume browser**: Explore mounted volumes

---

## 🛠️ Essential Docker Commands (Quick Reference)

```bash
# System Information
docker --version          # Docker version
docker info               # System info
docker system df          # Disk usage

# Container Management
docker ps                 # List running containers
docker ps -a              # List all containers
docker run <image>        # Run new container
docker start <container>  # Start stopped container
docker stop <container>   # Stop running container
docker rm <container>     # Remove container

# Image Management
docker images             # List local images
docker pull <image>       # Download image
docker rmi <image>        # Remove image
docker build -t <name> .  # Build image from Dockerfile

# Container Interaction
docker exec -it <container> bash  # Open shell in container
docker logs <container>           # View container logs
docker cp <file> <container>:path # Copy files

# Docker Hub
docker login              # Login to Docker Hub
docker push <image>       # Upload image
docker pull <image>       # Download image
```

---

## 🚨 Troubleshooting Common Issues

### Issue 1: "Docker daemon not running"
**Solution:**
```bash
# Windows/Mac: Start Docker Desktop
# Linux: Start Docker service
sudo systemctl start docker
```

### Issue 2: "Permission denied" (Linux)
**Solution:**
```bash
# Add user to docker group
sudo usermod -aG docker $USER
# Log out and back in
```

### Issue 3: "Port already in use"
**Solution:**
```bash
# Find what's using the port
netstat -tulpn | grep :8080
# Use different port
docker run -p 8081:80 nginx
```

### Issue 4: "Cannot connect to Docker daemon"
**Solution:**
```bash
# Check if Docker is running
docker info
# Restart Docker service
sudo systemctl restart docker
```

### Issue 5: WSL 2 issues (Windows)
**Solution:**
```powershell
# Update WSL 2
wsl --update
# Restart Docker Desktop
```

---

## 📋 Environment Checklist

Before proceeding to the next module, ensure:

✅ **Docker installed** and running  
✅ **Docker Hub account** created and logged in  
✅ **hello-world container** runs successfully  
✅ **Basic commands** working (docker ps, docker images)  
✅ **Port forwarding** tested with nginx example  
✅ **Container interaction** tested with ubuntu example  

### Quick Test Script:
```bash
#!/bin/bash
echo "Testing Docker installation..."

# Test 1: Docker version
echo "1. Docker version:"
docker --version || exit 1

# Test 2: Docker info
echo "2. Docker info (check daemon):"
docker info > /dev/null || exit 1

# Test 3: Run container
echo "3. Running test container:"
docker run --rm alpine echo "Docker works!" || exit 1

# Test 4: Port forwarding
echo "4. Testing port forwarding:"
docker run -d -p 8888:80 --name test-nginx nginx:alpine
sleep 5
curl -s http://localhost:8888 > /dev/null && echo "Port forwarding works!"
docker stop test-nginx && docker rm test-nginx

echo "✅ All tests passed! Docker is ready."
```

---

## 🎯 What's Next?

Now that Docker is installed and verified, you're ready for:

1. **Module 3**: Basic Docker Commands & Container Management
2. **Hands-on container operations**  
3. **Understanding container lifecycle**

### Key Achievements:
✅ Docker successfully installed on your system  
✅ Docker Hub account created and configured  
✅ First containers run and tested  
✅ Basic Docker commands mastered  
✅ Environment ready for development  

---

## 💡 Pro Tips for Your Setup

1. **Use Docker Playground** for initial learning - no setup required
2. **Install Docker Desktop** for serious development work
3. **Join Docker Hub** early - you'll need it for image sharing
4. **Bookmark Docker docs** - excellent reference material
5. **Practice commands daily** - muscle memory is important

Ready to dive into container management? Let's move to **Module 3: Basic Docker Commands & Containers**!