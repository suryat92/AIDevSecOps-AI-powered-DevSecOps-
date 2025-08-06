# 🐳 Module 5: Docker Compose & Multi-Container Applications

## 📚 Learning Objectives (75 minutes)

By the end of this module, you will:
- Understand Docker Compose and multi-container applications
- Write Docker Compose files with proper syntax
- Manage multi-container applications with compose commands
- Configure networking between containers
- Handle volumes and data persistence in compose
- Manage environment variables across services
- Scale services horizontally
- Implement service dependencies and health checks

---

## 🔍 What is Docker Compose?

**Docker Compose is a tool for defining and running multi-container Docker applications.** With Compose, you use a YAML file to configure your application's services, networks, and volumes, then create and start all services with a single command.

### The Problem Compose Solves

**Without Docker Compose:**
```bash
# Manual container management (painful!)
docker network create app-network
docker run -d --name database --network app-network postgres
docker run -d --name backend --network app-network -p 5000:5000 myapi
docker run -d --name frontend --network app-network -p 3000:3000 myui
```

**With Docker Compose:**
```bash
# Simple orchestration
docker-compose up
```

### Key Benefits
✅ **Declarative configuration** - Define desired state in YAML  
✅ **Single command management** - Start/stop entire stack  
✅ **Environment consistency** - Same config across environments  
✅ **Service discovery** - Automatic container networking  
✅ **Volume management** - Persistent data handling  
✅ **Scaling capabilities** - Horizontal service scaling  

---

## 📝 Docker Compose File Syntax

### Basic Structure

```yaml
version: '3.8'  # Compose file format version

services:        # Container definitions
  web:
    # Service configuration
  database:
    # Service configuration

networks:        # Custom networks (optional)
  # Network definitions

volumes:         # Named volumes (optional)
  # Volume definitions
```

### Version Compatibility

| Compose Version | Docker Engine | Features |
|-----------------|---------------|----------|
| 3.8 | 19.03.0+ | Latest features, buildkit support |
| 3.7 | 18.06.0+ | External networks, configs |
| 3.6 | 18.02.0+ | Device mappings |
| 3.5 | 17.12.0+ | Isolation, external volumes |

**Recommendation:** Use version 3.8 for maximum compatibility.

---

## 🏗️ Service Configuration

### 1. Basic Service Definition

```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    restart: unless-stopped
```

### 2. Building from Dockerfile

```yaml
services:
  app:
    build:
      context: ./app          # Build context
      dockerfile: Dockerfile  # Dockerfile name
      args:                   # Build arguments
        NODE_ENV: production
      target: production      # Multi-stage build target
    ports:
      - "3000:3000"
```

### 3. Advanced Build Configuration

```yaml
services:
  webapp:
    build:
      context: .
      dockerfile: Dockerfile.prod
      args:
        - BUILD_DATE=${BUILD_DATE}
        - VERSION=${VERSION}
      cache_from:
        - myapp:latest
      labels:
        - "com.example.version=1.0.0"
    image: myapp:latest  # Tag the built image
```

---

## 🌐 Container Networking in Compose

### Default Network Behavior

Docker Compose automatically:
- Creates a **default network** named `{project}_default`
- Connects all services to this network
- Enables **service discovery** by service name

```yaml
version: '3.8'

services:
  web:
    image: nginx
    # Can reach database as 'http://database:5432'
  
  database:
    image: postgres
    # Can reach web as 'http://web:80'
```

### Custom Networks

```yaml
version: '3.8'

services:
  frontend:
    image: nginx
    networks:
      - public
      - internal
  
  backend:
    image: myapi
    networks:
      - internal
      - database
  
  database:
    image: postgres
    networks:
      - database

networks:
  public:
    driver: bridge
  internal:
    driver: bridge
    internal: true    # No external access
  database:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

### Network Configuration Options

```yaml
networks:
  custom:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: custom-bridge
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
          ip_range: 172.28.5.0/24
          gateway: 172.28.5.254
    external: false  # Compose will create this network
  
  existing:
    external: true   # Use existing network
    name: my-existing-network
```

---

## 💾 Volume Management in Compose

### Named Volumes

```yaml
version: '3.8'

services:
  database:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: secret

  web:
    image: nginx
    volumes:
      - web_content:/usr/share/nginx/html

volumes:
  postgres_data:
    driver: local
  web_content:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.100,rw
      device: ":/path/to/nfs/share"
```

### Bind Mounts

```yaml
services:
  development:
    image: node:16
    volumes:
      # Bind mount source code for development
      - ./src:/app/src
      # Named volume for node_modules
      - node_modules:/app/node_modules
    working_dir: /app
    command: npm run dev

volumes:
  node_modules:
```

### Volume Configuration Options

```yaml
volumes:
  database:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /opt/database
    
  backup:
    external: true
    name: backup-volume
  
  logs:
    driver: local
    driver_opts:
      type: tmpfs
      device: tmpfs
```

---

## 🔧 Environment Variables in Compose

### 1. Environment Section

```yaml
services:
  web:
    image: myapp
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
      - API_KEY=${API_KEY}  # From host environment
      - DEBUG=false
```

### 2. Environment Files

**Create `.env` file:**
```bash
# Database configuration
POSTGRES_DB=myapp
POSTGRES_USER=admin
POSTGRES_PASSWORD=secretpassword

# Application settings
NODE_ENV=production
API_KEY=your-secret-api-key
DEBUG=false
```

**Use in compose:**
```yaml
services:
  database:
    image: postgres:13
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
  
  app:
    image: myapp
    env_file:
      - .env
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@database:5432/${POSTGRES_DB}
```

### 3. Multiple Environment Files

```yaml
services:
  app:
    image: myapp
    env_file:
      - .env          # Base environment
      - .env.local    # Local overrides
      - .env.secrets  # Sensitive data
```

### Environment Variable Precedence

1. **Compose file environment** (highest priority)
2. **Shell environment variables**
3. **Environment files** (.env)
4. **Dockerfile ENV** (lowest priority)

---

## 🚀 Docker Compose Commands

### Basic Commands

```bash
# Start services in foreground
docker-compose up

# Start services in background (detached)
docker-compose up -d

# Build images before starting
docker-compose up --build

# Start specific services
docker-compose up web database

# Stop services
docker-compose stop

# Stop and remove containers, networks
docker-compose down

# Stop and remove everything including volumes
docker-compose down -v

# Remove everything including images
docker-compose down --rmi all
```

### Service Management

```bash
# View running services
docker-compose ps

# View service logs
docker-compose logs
docker-compose logs web
docker-compose logs -f --tail=50 web

# Scale services
docker-compose up --scale web=3
docker-compose scale web=5

# Restart services
docker-compose restart
docker-compose restart web

# Execute commands in service containers
docker-compose exec web bash
docker-compose exec database psql -U postgres

# Run one-off commands
docker-compose run web python manage.py migrate
```

### Development Commands

```bash
# Rebuild specific service
docker-compose build web

# Pull latest images
docker-compose pull

# Validate compose file
docker-compose config

# View resolved configuration
docker-compose config --resolve-envvars

# Create and start containers without starting services
docker-compose create
```

---

## 🧪 Hands-on Lab: Building a Multi-Container Web Application

### Lab 1: LAMP Stack with Docker Compose

**Step 1: Create Project Structure**
```bash
mkdir lamp-stack
cd lamp-stack
mkdir -p {html,mysql,logs}

# Create sample PHP application
cat > html/index.php << 'EOF'
<?php
$servername = "mysql";
$username = "root";
$password = "rootpassword";
$dbname = "webapp";

try {
    $pdo = new PDO("mysql:host=$servername;dbname=$dbname", $username, $password);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    echo "<h1>LAMP Stack with Docker Compose</h1>";
    echo "<p>Database connection: <span style='color: green;'>SUCCESS</span></p>";
    
    // Create sample table and data
    $pdo->exec("CREATE TABLE IF NOT EXISTS visitors (
        id INT AUTO_INCREMENT PRIMARY KEY,
        visit_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        ip_address VARCHAR(45)
    )");
    
    // Insert visit record
    $ip = $_SERVER['REMOTE_ADDR'] ?? 'unknown';
    $stmt = $pdo->prepare("INSERT INTO visitors (ip_address) VALUES (?)");
    $stmt->execute([$ip]);
    
    // Display visitor count
    $stmt = $pdo->query("SELECT COUNT(*) as count FROM visitors");
    $count = $stmt->fetch()['count'];
    echo "<p>Total visits: <strong>$count</strong></p>";
    
    echo "<p>Current time: " . date('Y-m-d H:i:s') . "</p>";
    echo "<p>PHP Version: " . phpversion() . "</p>";
    
} catch(PDOException $e) {
    echo "<h1>LAMP Stack</h1>";
    echo "<p>Database connection: <span style='color: red;'>FAILED</span></p>";
    echo "<p>Error: " . $e->getMessage() . "</p>";
}
?>
EOF

# Create database initialization script
cat > mysql/init.sql << 'EOF'
CREATE DATABASE IF NOT EXISTS webapp;
USE webapp;

CREATE TABLE IF NOT EXISTS visitors (
    id INT AUTO_INCREMENT PRIMARY KEY,
    visit_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address VARCHAR(45)
);

INSERT INTO visitors (ip_address) VALUES ('127.0.0.1');
EOF
```

**Step 2: Create Docker Compose Configuration**
```yaml
version: '3.8'

services:
  # Web Server (Apache + PHP)
  web:
    image: php:8.1-apache
    container_name: lamp_web
    ports:
      - "8080:80"
    volumes:
      - ./html:/var/www/html
      - ./logs/apache:/var/log/apache2
    depends_on:
      - mysql
    environment:
      - APACHE_RUN_USER=www-data
      - APACHE_RUN_GROUP=www-data
    command: >
      bash -c "
        docker-php-ext-install pdo pdo_mysql mysqli &&
        apache2-foreground
      "
    networks:
      - lamp-network
    restart: unless-stopped

  # Database (MySQL)
  mysql:
    image: mysql:8.0
    container_name: lamp_mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: webapp
      MYSQL_USER: webuser
      MYSQL_PASSWORD: webpassword
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./mysql/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./logs/mysql:/var/log/mysql
    networks:
      - lamp-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 30s
      timeout: 10s
      retries: 5

  # Database Administration (phpMyAdmin)
  phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest
    container_name: lamp_phpmyadmin
    ports:
      - "8081:80"
    environment:
      PMA_HOST: mysql
      PMA_PORT: 3306
      PMA_USER: root
      PMA_PASSWORD: rootpassword
    depends_on:
      - mysql
    networks:
      - lamp-network
    restart: unless-stopped

volumes:
  mysql_data:
    driver: local

networks:
  lamp-network:
    driver: bridge
```

**Step 3: Environment Configuration**
```bash
# Create .env file
cat > .env << 'EOF'
# MySQL Configuration
MYSQL_ROOT_PASSWORD=rootpassword
MYSQL_DATABASE=webapp
MYSQL_USER=webuser
MYSQL_PASSWORD=webpassword

# Application Configuration
APP_ENV=development
DEBUG=true

# Port Configuration
WEB_PORT=8080
DB_PORT=3306
PHPMYADMIN_PORT=8081
EOF
```

**Step 4: Start and Test the Application**
```bash
# Start all services
docker-compose up -d

# Check service status
docker-compose ps

# View logs
docker-compose logs -f web

# Test the application
curl http://localhost:8080

# Access phpMyAdmin
# Open browser to http://localhost:8081
# Login: root / rootpassword

# Scale web servers
docker-compose up -d --scale web=3

# Clean up
docker-compose down -v
```

### Lab 2: Node.js + MongoDB + Redis Stack

**Step 1: Create Node.js Application**
```bash
mkdir node-mongo-redis
cd node-mongo-redis

# Create package.json
cat > package.json << 'EOF'
{
  "name": "node-mongo-redis-app",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  },
  "dependencies": {
    "express": "^4.18.0",
    "mongoose": "^7.0.0",
    "redis": "^4.6.0",
    "cors": "^2.8.5",
    "helmet": "^7.0.0",
    "morgan": "^1.10.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.0"
  }
}
EOF

# Create main application
cat > app.js << 'EOF'
const express = require('express');
const mongoose = require('mongoose');
const redis = require('redis');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(helmet());
app.use(cors());
app.use(morgan('combined'));
app.use(express.json());

// MongoDB connection
const MONGO_URL = process.env.MONGO_URL || 'mongodb://mongo:27017/nodeapp';
mongoose.connect(MONGO_URL)
  .then(() => console.log('Connected to MongoDB'))
  .catch(err => console.error('MongoDB connection error:', err));

// Redis connection
const REDIS_URL = process.env.REDIS_URL || 'redis://redis:6379';
const redisClient = redis.createClient({ url: REDIS_URL });

redisClient.on('error', (err) => console.error('Redis Client Error', err));
redisClient.on('connect', () => console.log('Connected to Redis'));

// Initialize Redis connection
(async () => {
  await redisClient.connect();
})();

// MongoDB Schema
const messageSchema = new mongoose.Schema({
  text: { type: String, required: true },
  author: { type: String, default: 'Anonymous' },
  createdAt: { type: Date, default: Date.now }
});

const Message = mongoose.model('Message', messageSchema);

// Routes
app.get('/', (req, res) => {
  res.json({
    message: 'Node.js + MongoDB + Redis Application',
    timestamp: new Date().toISOString(),
    environment: process.env.NODE_ENV || 'development'
  });
});

app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    mongodb: mongoose.connection.readyState === 1 ? 'connected' : 'disconnected',
    redis: redisClient.isOpen ? 'connected' : 'disconnected'
  });
});

// Messages endpoint with Redis caching
app.get('/messages', async (req, res) => {
  try {
    // Try to get from cache first
    const cacheKey = 'messages:all';
    const cached = await redisClient.get(cacheKey);
    
    if (cached) {
      return res.json({
        source: 'cache',
        data: JSON.parse(cached)
      });
    }
    
    // Get from database
    const messages = await Message.find().sort({ createdAt: -1 }).limit(10);
    
    // Cache the results for 30 seconds
    await redisClient.setEx(cacheKey, 30, JSON.stringify(messages));
    
    res.json({
      source: 'database',
      data: messages
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.post('/messages', async (req, res) => {
  try {
    const { text, author } = req.body;
    
    const message = new Message({ text, author });
    await message.save();
    
    // Clear cache when new message is added
    await redisClient.del('messages:all');
    
    res.status(201).json(message);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Stats endpoint
app.get('/stats', async (req, res) => {
  try {
    const messageCount = await Message.countDocuments();
    const latestMessage = await Message.findOne().sort({ createdAt: -1 });
    
    // Store stats in Redis
    const stats = {
      totalMessages: messageCount,
      latestMessage: latestMessage,
      timestamp: new Date().toISOString()
    };
    
    await redisClient.setEx('stats', 60, JSON.stringify(stats));
    
    res.json(stats);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
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

**Step 2: Create Docker Compose Configuration**
```yaml
version: '3.8'

services:
  # Node.js Application
  app:
    build: .
    ports:
      - "${APP_PORT:-3000}:3000"
    environment:
      NODE_ENV: ${NODE_ENV:-development}
      MONGO_URL: mongodb://mongo:27017/${MONGO_DB:-nodeapp}
      REDIS_URL: redis://redis:6379
    depends_on:
      mongo:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - app-network
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 256M
        reservations:
          memory: 128M

  # MongoDB Database
  mongo:
    image: mongo:6.0
    ports:
      - "${MONGO_PORT:-27017}:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_ROOT_USER:-admin}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD:-adminpass}
      MONGO_INITDB_DATABASE: ${MONGO_DB:-nodeapp}
    volumes:
      - mongo_data:/data/db
      - ./mongo-init:/docker-entrypoint-initdb.d
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 40s

  # Redis Cache
  redis:
    image: redis:7-alpine
    ports:
      - "${REDIS_PORT:-6379}:6379"
    volumes:
      - redis_data:/data
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 3s
      retries: 5
    command: redis-server --appendonly yes --maxmemory 256mb --maxmemory-policy allkeys-lru

  # MongoDB Admin Interface
  mongo-express:
    image: mongo-express:latest
    ports:
      - "${MONGO_EXPRESS_PORT:-8081}:8081"
    environment:
      ME_CONFIG_MONGODB_SERVER: mongo
      ME_CONFIG_MONGODB_PORT: 27017
      ME_CONFIG_MONGODB_ADMINUSERNAME: ${MONGO_ROOT_USER:-admin}
      ME_CONFIG_MONGODB_ADMINPASSWORD: ${MONGO_ROOT_PASSWORD:-adminpass}
      ME_CONFIG_BASICAUTH_USERNAME: ${MONGO_EXPRESS_USER:-admin}
      ME_CONFIG_BASICAUTH_PASSWORD: ${MONGO_EXPRESS_PASS:-adminpass}
    depends_on:
      - mongo
    networks:
      - app-network
    restart: unless-stopped

  # Redis Admin Interface
  redis-commander:
    image: rediscommander/redis-commander:latest
    ports:
      - "${REDIS_COMMANDER_PORT:-8082}:8081"
    environment:
      REDIS_HOSTS: local:redis:6379
    depends_on:
      - redis
    networks:
      - app-network
    restart: unless-stopped

volumes:
  mongo_data:
    driver: local
  redis_data:
    driver: local

networks:
  app-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

**Step 3: Environment and Testing**
```bash
# Create environment file
cat > .env << 'EOF'
# Application Configuration
NODE_ENV=development
APP_PORT=3000

# MongoDB Configuration
MONGO_DB=nodeapp
MONGO_PORT=27017
MONGO_ROOT_USER=admin
MONGO_ROOT_PASSWORD=adminpass

# Redis Configuration
REDIS_PORT=6379

# Admin Interface Configuration
MONGO_EXPRESS_PORT=8081
MONGO_EXPRESS_USER=admin
MONGO_EXPRESS_PASS=adminpass
REDIS_COMMANDER_PORT=8082
EOF

# Start all services
docker-compose up -d

# Check service health
docker-compose ps

# Test the API
curl http://localhost:3000/health

# Add some test data
curl -X POST http://localhost:3000/messages \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello from Docker Compose!", "author": "DevOps Student"}'

# Get messages (should return cached result on second call)
curl http://localhost:3000/messages

# Check stats
curl http://localhost:3000/stats

# Access admin interfaces
# MongoDB: http://localhost:8081 (admin/adminpass)
# Redis: http://localhost:8082
```

---

## 📈 Scaling Services with Compose

### Horizontal Scaling

```bash
# Scale specific service
docker-compose up -d --scale web=3

# Scale multiple services
docker-compose up -d --scale web=3 --scale worker=2

# View scaled services
docker-compose ps
```

### Load Balancing with Nginx

**Create nginx configuration:**
```nginx
upstream backend {
    server web_1:3000;
    server web_2:3000;
    server web_3:3000;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**Add to compose:**
```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - web
```

### Auto-scaling with Resource Limits

```yaml
services:
  web:
    image: myapp
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
        reservations:
          cpus: '0.2'
          memory: 128M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
```

---

## 🔍 Service Dependencies and Health Checks

### Dependency Management

```yaml
services:
  database:
    image: postgres:13
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 30s
      timeout: 10s
      retries: 5
  
  web:
    image: myapp
    depends_on:
      database:
        condition: service_healthy  # Wait for healthy database
    restart: on-failure
```

### Advanced Health Checks

```yaml
services:
  api:
    image: myapi
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s  # Grace period for startup
  
  worker:
    image: myworker
    healthcheck:
      test: ["CMD", "python", "health_check.py"]
      interval: 60s
      timeout: 30s
      retries: 3
    depends_on:
      api:
        condition: service_healthy
      database:
        condition: service_started
```

### Restart Policies

```yaml
services:
  web:
    image: nginx
    restart: always          # Always restart
  
  database:
    image: postgres
    restart: unless-stopped  # Restart unless manually stopped
  
  worker:
    image: myworker
    restart: on-failure:3    # Restart on failure, max 3 attempts
  
  cache:
    image: redis
    restart: "no"           # Never restart automatically
```

---

## 🔧 Development vs Production Configurations

### Override Files

**docker-compose.yml (base):**
```yaml
version: '3.8'

services:
  web:
    build: .
    environment:
      NODE_ENV: development
  
  database:
    image: postgres:13
```

**docker-compose.override.yml (development - automatic):**
```yaml
version: '3.8'

services:
  web:
    volumes:
      - ./src:/app/src  # Hot reload for development
    ports:
      - "3000:3000"
    command: npm run dev
  
  database:
    ports:
      - "5432:5432"    # Expose database for development
```

**docker-compose.prod.yml (production):**
```yaml
version: '3.8'

services:
  web:
    environment:
      NODE_ENV: production
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
    restart: unless-stopped
  
  database:
    volumes:
      - db_data:/var/lib/postgresql/data
    deploy:
      resources:
        limits:
          memory: 1G
```

### Usage

```bash
# Development (uses override automatically)
docker-compose up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Staging
docker-compose -f docker-compose.yml -f docker-compose.staging.yml up -d
```

---

## 🛠️ Advanced Compose Features

### Profiles (Compose v1.28+)

```yaml
services:
  web:
    image: nginx
    # Always included
  
  database:
    image: postgres
    profiles: ["production", "staging"]
  
  debug:
    image: busybox
    profiles: ["debug"]
    command: sleep infinity
```

```bash
# Start only web (no profiles)
docker-compose up

# Start web + database
docker-compose --profile production up

# Start everything including debug
docker-compose --profile debug --profile production up
```

### Extensions and Anchors

```yaml
x-common-variables: &common-env
  NODE_ENV: production
  LOG_LEVEL: info

x-resource-limits: &resource-limits
  deploy:
    resources:
      limits:
        memory: 256M

services:
  web1:
    image: myapp
    environment:
      <<: *common-env
      SERVICE_NAME: web1
    <<: *resource-limits
  
  web2:
    image: myapp
    environment:
      <<: *common-env
      SERVICE_NAME: web2
    <<: *resource-limits
```

### Config and Secrets (Swarm mode)

```yaml
version: '3.8'

services:
  app:
    image: myapp
    configs:
      - source: app_config
        target: /app/config.yml
    secrets:
      - db_password
    environment:
      DB_PASSWORD_FILE: /run/secrets/db_password

configs:
  app_config:
    file: ./config.yml

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

---

## 📋 Best Practices for Docker Compose

### 1. Structure and Organization
```yaml
# Good: Clear service organization
services:
  # Frontend services
  web:
    # ...
  nginx:
    # ...
  
  # Backend services  
  api:
    # ...
  worker:
    # ...
  
  # Data services
  database:
    # ...
  redis:
    # ...
```

### 2. Environment Management
```bash
# Use multiple environment files
.env                    # Base configuration
.env.local             # Local overrides (gitignored)
.env.production        # Production settings
.env.staging           # Staging settings
```

### 3. Security Best Practices
```yaml
services:
  database:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    # Don't expose ports unless necessary
    # ports:
    #   - "5432:5432"
    networks:
      - internal  # Use internal networks

secrets:
  db_password:
    file: ./secrets/db_password.txt
    
networks:
  internal:
    internal: true
```

### 4. Resource Management
```yaml
services:
  web:
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
        reservations:
          memory: 256M
          cpus: '0.2'
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

---

## 📝 Module Summary

### What You've Learned:
✅ **Docker Compose fundamentals** and YAML syntax  
✅ **Multi-container orchestration** with service dependencies  
✅ **Container networking** and service discovery  
✅ **Volume management** and data persistence  
✅ **Environment variable handling** across services  
✅ **Service scaling** and load balancing  
✅ **Health checks** and restart policies  
✅ **Development vs production** configurations  

### Key Skills Acquired:
- Write complex Docker Compose files
- Orchestrate multi-container applications
- Configure container networking and volumes
- Manage environment variables securely
- Scale services horizontally
- Implement health checks and dependencies
- Optimize for different environments

### Compose Commands Mastery:
```bash
# Essential commands you now know
docker-compose up -d --build
docker-compose ps
docker-compose logs -f service_name
docker-compose scale web=3
docker-compose exec service bash
docker-compose down -v
```

---

## 🚀 What's Next?

You're now ready for **Module 6: Docker Hub, Registries & CI/CD** where you'll learn to:
- Push and pull images from Docker Hub
- Set up automated builds
- Implement CI/CD pipelines with GitHub Actions
- Deploy multi-container applications automatically

### Practice Exercises:
1. Create a complete e-commerce stack (frontend, API, database, cache)
2. Implement a microservices architecture with service mesh
3. Set up development and production environments
4. Build a monitoring stack with Prometheus and Grafana

Excellent work on mastering Docker Compose! You can now orchestrate complex multi-container applications like a pro! 🎉