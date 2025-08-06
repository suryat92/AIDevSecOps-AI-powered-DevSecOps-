# 🐳 Module 3: Basic Docker Commands & Container Management

## 📚 Learning Objectives (60 minutes)

By the end of this module, you will:
- Master essential Docker commands
- Understand container lifecycle
- Manage running and stopped containers
- Work with container logs and debugging
- Handle environment variables and ports
- Practice container resource management

---

## 🔄 Container Lifecycle

Understanding the container lifecycle is crucial for effective Docker usage:

```
┌─────────────┐    docker run     ┌─────────────┐
│             │ ─────────────────>│             │
│   IMAGE     │                   │  CONTAINER  │
│ (template)  │<──────────────────│ (running)   │
└─────────────┘   docker commit   └─────────────┘
                                         │
                                         │ docker stop
                                         ▼
                                  ┌─────────────┐
                                  │  CONTAINER  │
                                  │ (stopped)   │
                                  └─────────────┘
                                         │
                                         │ docker rm
                                         ▼
                                    [DELETED]
```

### Lifecycle States:
1. **Created**: Container exists but not started
2. **Running**: Container is actively running
3. **Paused**: Container processes are paused
4. **Stopped**: Container has exited
5. **Deleted**: Container no longer exists

---

## 🚀 Essential Docker Commands

### 1. Container Creation and Execution

#### `docker run` - Create and start container
```bash
# Basic syntax
docker run [OPTIONS] IMAGE [COMMAND]

# Simple examples
docker run ubuntu echo "Hello World"
docker run -d nginx                    # Detached mode
docker run -it ubuntu bash            # Interactive terminal
docker run --name my-container nginx  # Named container
docker run -p 8080:80 nginx          # Port mapping
docker run --rm alpine ls            # Auto-remove after exit
```

#### Common `docker run` Options:
```bash
-d, --detach          # Run in background
-i, --interactive     # Keep STDIN open
-t, --tty            # Allocate pseudo-TTY
--name               # Assign container name
-p, --publish        # Publish ports (host:container)
-v, --volume         # Mount volumes
-e, --env            # Set environment variables
--rm                 # Automatically remove container when it exits
-w, --workdir        # Working directory inside container
--restart            # Restart policy (no, always, on-failure)
```

### 2. Container Management Commands

#### List containers
```bash
# List running containers
docker ps

# List all containers (running + stopped)
docker ps -a

# List with custom format
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# List only container IDs
docker ps -q
```

#### Start, stop, restart containers
```bash
# Start stopped container
docker start <container_name_or_id>

# Stop running container
docker stop <container_name_or_id>

# Restart container
docker restart <container_name_or_id>

# Pause container processes
docker pause <container_name_or_id>

# Unpause container processes
docker unpause <container_name_or_id>

# Kill container (force stop)
docker kill <container_name_or_id>
```

#### Remove containers
```bash
# Remove stopped container
docker rm <container_name_or_id>

# Remove running container (force)
docker rm -f <container_name_or_id>

# Remove all stopped containers
docker container prune

# Remove multiple containers
docker rm container1 container2 container3
```

---

## 🔍 Container Inspection and Debugging

### Container Logs
```bash
# View container logs
docker logs <container_name_or_id>

# Follow logs in real-time
docker logs -f <container_name_or_id>

# Show last N lines
docker logs --tail 50 <container_name_or_id>

# Show logs with timestamps
docker logs -t <container_name_or_id>

# Show logs since specific time
docker logs --since "2024-01-01T00:00:00" <container_name_or_id>
```

### Container Statistics
```bash
# Show live resource usage
docker stats

# Show stats for specific container
docker stats <container_name_or_id>

# Show stats once (no streaming)
docker stats --no-stream
```

### Container Inspection
```bash
# Detailed container information
docker inspect <container_name_or_id>

# Get specific information using format
docker inspect --format='{{.State.Status}}' <container>
docker inspect --format='{{.NetworkSettings.IPAddress}}' <container>
docker inspect --format='{{.Config.Env}}' <container>
```

### Execute Commands in Running Containers
```bash
# Execute command in running container
docker exec <container> <command>

# Interactive shell in running container
docker exec -it <container> bash
docker exec -it <container> sh        # If bash not available

# Run command as specific user
docker exec -u root -it <container> bash

# Examples
docker exec my-nginx cat /etc/nginx/nginx.conf
docker exec -it my-postgres psql -U postgres
```

---

## 🌐 Container Networking

### Port Mapping
```bash
# Map single port
docker run -p 8080:80 nginx

# Map multiple ports
docker run -p 8080:80 -p 8443:443 nginx

# Map to specific host interface
docker run -p 127.0.0.1:8080:80 nginx

# Map random host port
docker run -P nginx  # Maps all exposed ports to random host ports

# View port mappings
docker port <container>
```

### Network Management
```bash
# List networks
docker network ls

# Inspect network
docker network inspect bridge

# Create custom network
docker network create my-network

# Run container on specific network
docker run -d --network my-network --name web nginx

# Connect running container to network
docker network connect my-network <container>

# Disconnect container from network
docker network disconnect my-network <container>
```

---

## 💾 Container Data Management

### Environment Variables
```bash
# Set single environment variable
docker run -e DATABASE_URL="postgresql://user:pass@host:5432/db" myapp

# Set multiple environment variables
docker run -e NODE_ENV=production -e PORT=3000 myapp

# Load environment variables from file
docker run --env-file .env myapp

# Example .env file content:
# NODE_ENV=production
# DATABASE_URL=postgresql://localhost:5432/mydb
# API_KEY=your-secret-key
```

### Volume Mounting
```bash
# Mount host directory to container
docker run -v /host/path:/container/path nginx

# Mount current directory
docker run -v $(pwd):/app node:16 ls /app

# Named volumes
docker volume create my-volume
docker run -v my-volume:/data postgres

# Anonymous volumes
docker run -v /data postgres  # Docker creates anonymous volume

# Read-only mounts
docker run -v $(pwd):/app:ro nginx
```

---

## 🔧 Resource Management

### Memory Limits
```bash
# Limit memory usage
docker run -m 512m nginx          # 512 MB limit
docker run --memory=1g nginx      # 1 GB limit

# Memory + swap limit
docker run -m 512m --memory-swap=1g nginx
```

### CPU Limits
```bash
# Limit CPU usage
docker run --cpus="1.5" nginx     # 1.5 CPU cores
docker run --cpu-shares=512 nginx # Relative weight

# Assign specific CPUs
docker run --cpuset-cpus="0,1" nginx  # Use only CPU 0 and 1
```

### Disk I/O Limits
```bash
# Limit read/write operations per second
docker run --device-read-iops /dev/sda:1000 nginx
docker run --device-write-iops /dev/sda:1000 nginx
```

---

## 🧪 Hands-on Lab Exercises

### Lab 1: Basic Container Operations

```bash
# 1. Run a simple container
docker run hello-world

# 2. Run interactive Ubuntu container
docker run -it --name my-ubuntu ubuntu bash
# Inside container:
apt update
apt install -y curl
curl --version
exit

# 3. Start the stopped container
docker start my-ubuntu

# 4. Execute command in running container
docker exec my-ubuntu curl --version

# 5. View container logs
docker logs my-ubuntu

# 6. Stop and remove container
docker stop my-ubuntu
docker rm my-ubuntu
```

### Lab 2: Web Server Management

```bash
# 1. Run Nginx web server
docker run -d -p 8080:80 --name web-server nginx

# 2. Test the web server
curl http://localhost:8080

# 3. View server logs
docker logs web-server

# 4. Execute command in container
docker exec web-server nginx -v

# 5. Copy custom HTML file
echo "<h1>Hello Docker!</h1>" > index.html
docker cp index.html web-server:/usr/share/nginx/html/

# 6. Test custom page
curl http://localhost:8080

# 7. View container processes
docker exec web-server ps aux

# 8. Clean up
docker stop web-server
docker rm web-server
```

### Lab 3: Database Container

```bash
# 1. Run PostgreSQL database
docker run -d \
  --name my-postgres \
  -e POSTGRES_DB=sampledb \
  -e POSTGRES_USER=student \
  -e POSTGRES_PASSWORD=password123 \
  -p 5432:5432 \
  postgres:13

# 2. Wait for database to start
sleep 10

# 3. Connect to database
docker exec -it my-postgres psql -U student -d sampledb

# Inside PostgreSQL:
\l                          # List databases
CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(50));
INSERT INTO users (name) VALUES ('Alice'), ('Bob');
SELECT * FROM users;
\q                          # Quit

# 4. View database logs
docker logs my-postgres

# 5. Backup database
docker exec my-postgres pg_dump -U student sampledb > backup.sql

# 6. Check container resources
docker stats my-postgres --no-stream

# 7. Clean up
docker stop my-postgres
docker rm my-postgres
```

### Lab 4: Multi-Container Communication

```bash
# 1. Create custom network
docker network create app-network

# 2. Run database on custom network
docker run -d \
  --name app-db \
  --network app-network \
  -e POSTGRES_DB=appdb \
  -e POSTGRES_USER=app \
  -e POSTGRES_PASSWORD=secret \
  postgres:13

# 3. Run web application that connects to database
docker run -d \
  --name app-web \
  --network app-network \
  -p 3000:3000 \
  -e DATABASE_URL=postgresql://app:secret@app-db:5432/appdb \
  node:16 \
  sh -c "npm install express pg && node -e '
    const express = require(\"express\");
    const app = express();
    app.get(\"/\", (req, res) => res.send(\"App connected to database!\"));
    app.listen(3000, () => console.log(\"Server running on port 3000\"));
  '"

# 4. Test application
sleep 5
curl http://localhost:3000

# 5. Inspect network
docker network inspect app-network

# 6. Clean up
docker stop app-web app-db
docker rm app-web app-db
docker network rm app-network
```

---

## 🐛 Troubleshooting Common Issues

### Issue 1: Container Exits Immediately
```bash
# Check exit code and logs
docker ps -a
docker logs <container>

# Common causes:
# - No long-running process
# - Application error
# - Missing files/dependencies

# Solution: Run with interactive mode to debug
docker run -it <image> bash
```

### Issue 2: Port Already in Use
```bash
# Find what's using the port
netstat -tulpn | grep :8080  # Linux
lsof -i :8080                # Mac

# Solutions:
docker run -p 8081:80 nginx  # Use different port
docker stop <container>      # Stop conflicting container
```

### Issue 3: Permission Denied
```bash
# Check container user
docker exec <container> whoami
docker exec <container> id

# Run as specific user
docker run -u $(id -u):$(id -g) <image>

# Fix file permissions
docker exec -u root <container> chown -R user:group /app
```

### Issue 4: Container Not Responding
```bash
# Check if container is running
docker ps

# Check container processes
docker exec <container> ps aux

# Check resource usage
docker stats <container>

# Force kill if necessary
docker kill <container>
```

---

## 📊 Container Monitoring and Management

### Real-time Monitoring
```bash
# Monitor all containers
docker stats

# Monitor specific containers
docker stats web-server db-server

# Container processes
docker exec <container> top

# Container filesystem usage
docker exec <container> df -h
```

### Container Health Checks
```bash
# Run container with health check
docker run -d \
  --name healthy-nginx \
  --health-cmd="curl -f http://localhost || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  nginx

# Check health status
docker inspect --format='{{.State.Health.Status}}' healthy-nginx
```

### Bulk Operations
```bash
# Stop all running containers
docker stop $(docker ps -q)

# Remove all stopped containers
docker container prune

# Remove all containers (running and stopped)
docker rm -f $(docker ps -aq)

# Remove containers by pattern
docker ps -a --filter "name=test-*" -q | xargs docker rm
```

---

## 📝 Command Cheat Sheet

### Essential Commands
```bash
# Container Lifecycle
docker run <image>           # Create and start
docker start <container>     # Start stopped container
docker stop <container>      # Stop running container
docker restart <container>   # Restart container
docker rm <container>        # Remove container

# Container Information
docker ps                   # List running containers
docker ps -a               # List all containers
docker logs <container>    # View container logs
docker inspect <container> # Detailed information
docker stats <container>   # Resource usage

# Container Interaction
docker exec -it <container> bash  # Interactive shell
docker exec <container> <command> # Execute command
docker cp <file> <container>:path # Copy files
```

### Advanced Commands
```bash
# Networking
docker run -p <host>:<container> <image>  # Port mapping
docker network create <network>           # Create network
docker network connect <net> <container> # Connect to network

# Resource Management
docker run -m <memory> <image>           # Memory limit
docker run --cpus <count> <image>       # CPU limit

# Environment & Volumes
docker run -e <VAR>=<value> <image>     # Environment variable
docker run -v <host>:<container> <image> # Volume mount
```

---

## 🎯 Module Summary

### What You've Learned:
✅ **Container lifecycle** and state management  
✅ **Essential Docker commands** for daily operations  
✅ **Container debugging** with logs and inspection  
✅ **Networking concepts** and port mapping  
✅ **Data management** with volumes and environment variables  
✅ **Resource management** and monitoring  
✅ **Troubleshooting techniques** for common issues  

### Key Skills Acquired:
- Run and manage containers effectively
- Debug container issues using logs and stats
- Configure container networking and ports
- Handle data persistence and environment variables
- Monitor container resource usage
- Perform bulk container operations

---

## 🚀 What's Next?

You're now ready for **Module 4: Docker Images & Dockerfiles** where you'll learn to:
- Create custom Docker images
- Write Dockerfiles
- Understand image layers and optimization
- Build and tag images
- Share images via registries

### Practice Exercises:
1. Create a development environment with database and web server
2. Set up container monitoring for multiple services  
3. Practice troubleshooting different container scenarios
4. Experiment with different resource limits and configurations

Great job mastering Docker container fundamentals! 🎉