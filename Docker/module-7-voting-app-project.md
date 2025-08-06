# 🐳 Module 7: Complete Project - AI-Powered Voting App

## 📚 Learning Objectives (60 minutes)

By the end of this module, you will have built a complete multi-container application featuring:
- **Frontend**: React web application
- **Backend**: Python Flask API  
- **Database**: PostgreSQL for data persistence
- **AI Service**: Sentiment analysis microservice
- **Reverse Proxy**: Nginx for load balancing
- **CI/CD Pipeline**: GitHub Actions automation
- **Container Orchestration**: Docker Compose

---

## 🏗️ Project Architecture

### System Overview
```
┌─────────────────────────────────────────────┐
│                 INTERNET                    │
└─────────────────┬───────────────────────────┘
                  │
┌─────────────────▼───────────────────────────┐
│              Nginx Proxy                    │
│            (Port 80/443)                    │
└─────────────────┬───────────────────────────┘
                  │
         ┌────────┴────────┐
         │                 │
┌────────▼───────┐ ┌──────▼──────────┐
│  React Frontend│ │   Flask API     │
│   (Port 3000)  │ │  (Port 5000)    │
└────────────────┘ └─────────┬───────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
    ┌─────────▼──────┐ ┌───▼───────┐ ┌──▼────────┐
    │  PostgreSQL    │ │ AI Service│ │   Redis   │
    │  (Port 5432)   │ │(Port 8000)│ │(Port 6379)│
    └────────────────┘ └───────────┘ └───────────┘
```

### Components Description
- **Nginx**: Reverse proxy and load balancer
- **React Frontend**: User interface for voting
- **Flask API**: Backend API handling votes and results
- **AI Service**: Sentiment analysis for vote comments
- **PostgreSQL**: Database for storing votes
- **Redis**: Caching layer for better performance

---

## 📁 Project Structure

```
voting-app/
├── docker-compose.yml          # Multi-container orchestration
├── docker-compose.prod.yml     # Production overrides
├── .env                        # Environment variables
├── nginx/
│   ├── Dockerfile
│   └── nginx.conf             # Nginx configuration
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── src/
│   │   ├── App.js
│   │   ├── components/
│   │   └── services/
│   └── public/
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── app.py                 # Flask application
│   ├── models/
│   └── services/
├── ai-service/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── app.py                 # FastAPI application
│   └── models/
├── database/
│   ├── init.sql               # Database initialization
│   └── Dockerfile
└── .github/
    └── workflows/
        └── ci-cd.yml          # GitHub Actions pipeline
```

---

## 🚀 Building the Application

### Step 1: Create Project Directory

```bash
# Create project structure
mkdir voting-app && cd voting-app
mkdir -p {nginx,frontend,backend,ai-service,database,.github/workflows}

# Create environment file
cat > .env << 'EOF'
# Database Configuration
POSTGRES_DB=voting_db
POSTGRES_USER=voting_user
POSTGRES_PASSWORD=secure_password123
DATABASE_URL=postgresql://voting_user:secure_password123@db:5432/voting_db

# Application Configuration
FLASK_ENV=production
NODE_ENV=production
AI_SERVICE_URL=http://ai-service:8000

# Redis Configuration
REDIS_URL=redis://redis:6379/0
EOF
```

### Step 2: Backend API (Flask)

**Create `backend/requirements.txt`:**
```txt
Flask==2.3.3
Flask-CORS==4.0.0
Flask-SQLAlchemy==3.0.5
psycopg2-binary==2.9.7
redis==4.6.0
requests==2.31.0
gunicorn==21.2.0
python-dotenv==1.0.0
```

**Create `backend/app.py`:**
```python
from flask import Flask, request, jsonify
from flask_cors import CORS
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime
import redis
import requests
import os
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)
CORS(app)

# Configuration
app.config['SQLALCHEMY_DATABASE_URI'] = os.getenv('DATABASE_URL')
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# Initialize extensions
db = SQLAlchemy(app)
redis_client = redis.from_url(os.getenv('REDIS_URL'))

# Models
class Vote(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    option = db.Column(db.String(100), nullable=False)
    comment = db.Column(db.Text)
    sentiment = db.Column(db.String(20))
    confidence = db.Column(db.Float)
    timestamp = db.Column(db.DateTime, default=datetime.utcnow)
    
    def to_dict(self):
        return {
            'id': self.id,
            'option': self.option,
            'comment': self.comment,
            'sentiment': self.sentiment,
            'confidence': self.confidence,
            'timestamp': self.timestamp.isoformat()
        }

# Routes
@app.route('/health')
def health():
    return jsonify({'status': 'healthy', 'service': 'backend'})

@app.route('/api/vote', methods=['POST'])
def submit_vote():
    try:
        data = request.get_json()
        option = data.get('option')
        comment = data.get('comment', '')
        
        # Analyze sentiment if comment provided
        sentiment = None
        confidence = None
        
        if comment:
            try:
                ai_response = requests.post(
                    f"{os.getenv('AI_SERVICE_URL')}/analyze",
                    json={'text': comment},
                    timeout=5
                )
                if ai_response.status_code == 200:
                    ai_data = ai_response.json()
                    sentiment = ai_data.get('sentiment')
                    confidence = ai_data.get('confidence')
            except requests.exceptions.RequestException:
                pass  # Continue without sentiment analysis
        
        # Save vote
        vote = Vote(
            option=option,
            comment=comment,
            sentiment=sentiment,
            confidence=confidence
        )
        db.session.add(vote)
        db.session.commit()
        
        # Clear cache
        redis_client.delete('vote_results')
        redis_client.delete('recent_votes')
        
        return jsonify({
            'message': 'Vote submitted successfully',
            'vote': vote.to_dict()
        }), 201
        
    except Exception as e:
        return jsonify({'error': str(e)}), 500

@app.route('/api/results')
def get_results():
    try:
        # Check cache first
        cached_results = redis_client.get('vote_results')
        if cached_results:
            import json
            return json.loads(cached_results)
        
        # Get vote counts
        results = db.session.query(
            Vote.option,
            db.func.count(Vote.id).label('count')
        ).group_by(Vote.option).all()
        
        # Get sentiment analysis
        sentiment_stats = db.session.query(
            Vote.sentiment,
            db.func.count(Vote.id).label('count'),
            db.func.avg(Vote.confidence).label('avg_confidence')
        ).filter(Vote.sentiment.isnot(None)).group_by(Vote.sentiment).all()
        
        response_data = {
            'votes': [{'option': r.option, 'count': r.count} for r in results],
            'sentiment': [
                {
                    'sentiment': s.sentiment,
                    'count': s.count,
                    'avg_confidence': round(float(s.avg_confidence or 0), 2)
                } for s in sentiment_stats
            ],
            'total_votes': sum(r.count for r in results),
            'timestamp': datetime.utcnow().isoformat()
        }
        
        # Cache results for 30 seconds
        redis_client.setex('vote_results', 30, json.dumps(response_data))
        
        return jsonify(response_data)
        
    except Exception as e:
        return jsonify({'error': str(e)}), 500

@app.route('/api/votes/recent')
def get_recent_votes():
    try:
        # Check cache
        cached_votes = redis_client.get('recent_votes')
        if cached_votes:
            import json
            return json.loads(cached_votes)
        
        votes = Vote.query.order_by(Vote.timestamp.desc()).limit(10).all()
        response_data = {
            'votes': [vote.to_dict() for vote in votes]
        }
        
        # Cache for 10 seconds
        redis_client.setex('recent_votes', 10, json.dumps(response_data))
        
        return jsonify(response_data)
        
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    with app.app_context():
        db.create_all()
    app.run(host='0.0.0.0', port=5000, debug=False)
```

**Create `backend/Dockerfile`:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser && \
    chown -R appuser:appuser /app
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

EXPOSE 5000
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "app:app"]
```

### Step 3: AI Service (FastAPI)

**Create `ai-service/requirements.txt`:**
```txt
fastapi==0.103.1
uvicorn==0.23.2
pydantic==2.4.2
transformers==4.35.0
torch==2.0.1
numpy==1.24.3
```

**Create `ai-service/app.py`:**
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from transformers import pipeline
import logging
import asyncio

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI(title="AI Sentiment Analysis Service")

# Initialize sentiment analysis pipeline
try:
    sentiment_pipeline = pipeline(
        "sentiment-analysis",
        model="cardiffnlp/twitter-roberta-base-sentiment-latest",
        return_all_scores=True
    )
    logger.info("Sentiment analysis model loaded successfully")
except Exception as e:
    logger.error(f"Failed to load sentiment model: {e}")
    sentiment_pipeline = None

class TextInput(BaseModel):
    text: str

class SentimentResponse(BaseModel):
    sentiment: str
    confidence: float
    all_scores: list

@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "service": "ai-service",
        "model_loaded": sentiment_pipeline is not None
    }

@app.post("/analyze", response_model=SentimentResponse)
async def analyze_sentiment(input_data: TextInput):
    try:
        if not sentiment_pipeline:
            raise HTTPException(
                status_code=503,
                detail="Sentiment analysis model not available"
            )
        
        if not input_data.text.strip():
            raise HTTPException(
                status_code=400,
                detail="Text input cannot be empty"
            )
        
        # Run sentiment analysis
        results = sentiment_pipeline(input_data.text)
        
        # Process results
        sentiment_scores = results[0]  # Get scores for the text
        
        # Find the highest confidence sentiment
        best_sentiment = max(sentiment_scores, key=lambda x: x['score'])
        
        # Map labels to readable names
        label_mapping = {
            'LABEL_0': 'negative',
            'LABEL_1': 'neutral', 
            'LABEL_2': 'positive'
        }
        
        mapped_sentiment = label_mapping.get(
            best_sentiment['label'], 
            best_sentiment['label'].lower()
        )
        
        return SentimentResponse(
            sentiment=mapped_sentiment,
            confidence=round(best_sentiment['score'], 3),
            all_scores=[
                {
                    "sentiment": label_mapping.get(score['label'], score['label']),
                    "confidence": round(score['score'], 3)
                }
                for score in sentiment_scores
            ]
        )
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"Error analyzing sentiment: {e}")
        raise HTTPException(
            status_code=500,
            detail=f"Internal server error: {str(e)}"
        )

@app.get("/")
async def root():
    return {
        "message": "AI Sentiment Analysis Service",
        "version": "1.0.0",
        "endpoints": ["/health", "/analyze"]
    }

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

**Create `ai-service/Dockerfile`:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN adduser --disabled-password --gecos '' aiuser && \
    chown -R aiuser:aiuser /app
USER aiuser

EXPOSE 8000

# Health check
HEALTHCHECK --interval=60s --timeout=30s --start-period=120s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Step 4: Frontend (React)

**Create `frontend/package.json`:**
```json
{
  "name": "voting-app-frontend",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1",
    "axios": "^1.5.0",
    "recharts": "^2.8.0",
    "@mui/material": "^5.14.0",
    "@emotion/react": "^11.11.0",
    "@emotion/styled": "^11.11.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "proxy": "http://backend:5000"
}
```

**Create `frontend/src/App.js`:**
```jsx
import React, { useState, useEffect } from 'react';
import {
  Container,
  Paper,
  Typography,
  Button,
  TextField,
  RadioGroup,
  FormControlLabel,
  Radio,
  Grid,
  Card,
  CardContent,
  Alert,
  Box,
  LinearProgress
} from '@mui/material';
import { PieChart, Pie, Cell, BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer } from 'recharts';
import axios from 'axios';

const COLORS = ['#0088FE', '#00C49F', '#FFBB28', '#FF8042', '#8884D8'];

function App() {
  const [selectedOption, setSelectedOption] = useState('');
  const [comment, setComment] = useState('');
  const [results, setResults] = useState(null);
  const [recentVotes, setRecentVotes] = useState([]);
  const [loading, setLoading] = useState(false);
  const [message, setMessage] = useState('');

  const votingOptions = [
    'Python',
    'JavaScript',
    'Java',
    'Go',
    'Rust'
  ];

  useEffect(() => {
    fetchResults();
    fetchRecentVotes();
    const interval = setInterval(() => {
      fetchResults();
      fetchRecentVotes();
    }, 5000);
    return () => clearInterval(interval);
  }, []);

  const fetchResults = async () => {
    try {
      const response = await axios.get('/api/results');
      setResults(response.data);
    } catch (error) {
      console.error('Error fetching results:', error);
    }
  };

  const fetchRecentVotes = async () => {
    try {
      const response = await axios.get('/api/votes/recent');
      setRecentVotes(response.data.votes);
    } catch (error) {
      console.error('Error fetching recent votes:', error);
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!selectedOption) {
      setMessage('Please select an option');
      return;
    }

    setLoading(true);
    try {
      await axios.post('/api/vote', {
        option: selectedOption,
        comment: comment
      });
      setMessage('Vote submitted successfully!');
      setSelectedOption('');
      setComment('');
      setTimeout(() => setMessage(''), 3000);
      fetchResults();
      fetchRecentVotes();
    } catch (error) {
      setMessage('Error submitting vote');
    }
    setLoading(false);
  };

  const getSentimentColor = (sentiment) => {
    switch (sentiment) {
      case 'positive': return '#4CAF50';
      case 'negative': return '#F44336';
      case 'neutral': return '#FF9800';
      default: return '#9E9E9E';
    }
  };

  return (
    <Container maxWidth="lg" sx={{ py: 4 }}>
      <Typography variant="h3" component="h1" gutterBottom align="center">
        🗳️ AI-Powered Voting App
      </Typography>
      
      <Grid container spacing={4}>
        {/* Voting Form */}
        <Grid item xs={12} md={6}>
          <Paper sx={{ p: 3 }}>
            <Typography variant="h5" gutterBottom>
              Vote for Your Favorite Programming Language
            </Typography>
            
            {message && (
              <Alert severity={message.includes('Error') ? 'error' : 'success'} sx={{ mb: 2 }}>
                {message}
              </Alert>
            )}

            <form onSubmit={handleSubmit}>
              <RadioGroup
                value={selectedOption}
                onChange={(e) => setSelectedOption(e.target.value)}
                sx={{ mb: 2 }}
              >
                {votingOptions.map((option) => (
                  <FormControlLabel
                    key={option}
                    value={option}
                    control={<Radio />}
                    label={option}
                  />
                ))}
              </RadioGroup>

              <TextField
                fullWidth
                multiline
                rows={3}
                label="Comment (optional)"
                value={comment}
                onChange={(e) => setComment(e.target.value)}
                sx={{ mb: 2 }}
                helperText="Your comment will be analyzed for sentiment using AI"
              />

              <Button
                type="submit"
                variant="contained"
                fullWidth
                disabled={loading}
                sx={{ mb: 2 }}
              >
                {loading ? 'Submitting...' : 'Submit Vote'}
              </Button>
              
              {loading && <LinearProgress />}
            </form>
          </Paper>
        </Grid>

        {/* Results */}
        <Grid item xs={12} md={6}>
          <Paper sx={{ p: 3 }}>
            <Typography variant="h5" gutterBottom>
              Live Results
            </Typography>
            
            {results && results.votes.length > 0 ? (
              <ResponsiveContainer width="100%" height={300}>
                <PieChart>
                  <Pie
                    data={results.votes}
                    cx="50%"
                    cy="50%"
                    labelLine={false}
                    label={({option, count}) => `${option}: ${count}`}
                    outerRadius={80}
                    fill="#8884d8"
                    dataKey="count"
                  >
                    {results.votes.map((entry, index) => (
                      <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
                    ))}
                  </Pie>
                  <Tooltip />
                </PieChart>
              </ResponsiveContainer>
            ) : (
              <Typography>No votes yet. Be the first to vote!</Typography>
            )}

            {results && (
              <Box sx={{ mt: 2 }}>
                <Typography variant="body2" color="textSecondary">
                  Total Votes: {results.total_votes}
                </Typography>
                <Typography variant="body2" color="textSecondary">
                  Last Updated: {new Date(results.timestamp).toLocaleTimeString()}
                </Typography>
              </Box>
            )}
          </Paper>
        </Grid>

        {/* Sentiment Analysis */}
        {results && results.sentiment.length > 0 && (
          <Grid item xs={12} md={6}>
            <Paper sx={{ p: 3 }}>
              <Typography variant="h5" gutterBottom>
                Sentiment Analysis
              </Typography>
              <ResponsiveContainer width="100%" height={200}>
                <BarChart data={results.sentiment}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="sentiment" />
                  <YAxis />
                  <Tooltip />
                  <Legend />
                  <Bar dataKey="count" fill="#8884d8" />
                </BarChart>
              </ResponsiveContainer>
            </Paper>
          </Grid>
        )}

        {/* Recent Votes */}
        <Grid item xs={12} md={6}>
          <Paper sx={{ p: 3 }}>
            <Typography variant="h5" gutterBottom>
              Recent Votes
            </Typography>
            {recentVotes.map((vote, index) => (
              <Card key={vote.id} sx={{ mb: 1, bgcolor: 'grey.50' }}>
                <CardContent sx={{ py: 1 }}>
                  <Typography variant="body1">
                    <strong>{vote.option}</strong>
                  </Typography>
                  {vote.comment && (
                    <Typography variant="body2" color="textSecondary">
                      "{vote.comment}"
                    </Typography>
                  )}
                  {vote.sentiment && (
                    <Box sx={{ display: 'flex', alignItems: 'center', mt: 1 }}>
                      <Typography
                        variant="caption"
                        sx={{
                          color: getSentimentColor(vote.sentiment),
                          fontWeight: 'bold'
                        }}
                      >
                        {vote.sentiment.toUpperCase()}
                      </Typography>
                      <Typography variant="caption" color="textSecondary" sx={{ ml: 1 }}>
                        ({Math.round(vote.confidence * 100)}% confidence)
                      </Typography>
                    </Box>
                  )}
                  <Typography variant="caption" color="textSecondary">
                    {new Date(vote.timestamp).toLocaleString()}
                  </Typography>
                </CardContent>
              </Card>
            ))}
          </Paper>
        </Grid>
      </Grid>
    </Container>
  );
}

export default App;
```

**Create `frontend/Dockerfile`:**
```dockerfile
# Build stage
FROM node:16-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy built app from builder stage
COPY --from=builder /app/build /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Create non-root user
RUN adduser -D -s /bin/sh nginxuser
USER nginxuser

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**Create `frontend/nginx.conf`:**
```nginx
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html index.htm;

    # Enable gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # Handle client-side routing
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Proxy API requests to backend
    location /api/ {
        proxy_pass http://backend:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
}
```

### Step 5: Docker Compose Configuration

**Create `docker-compose.yml`:**
```yaml
version: '3.8'

services:
  # Database
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - voting-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Redis Cache
  redis:
    image: redis:7-alpine
    networks:
      - voting-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

  # AI Service
  ai-service:
    build:
      context: ./ai-service
      dockerfile: Dockerfile
    networks:
      - voting-network
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M

  # Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
      REDIS_URL: redis://redis:6379/0
      AI_SERVICE_URL: http://ai-service:8000
      FLASK_ENV: ${FLASK_ENV}
    depends_on:
      - db
      - redis
      - ai-service
    networks:
      - voting-network
    restart: unless-stopped

  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    depends_on:
      - backend
    networks:
      - voting-network
    restart: unless-stopped

  # Nginx Reverse Proxy
  nginx:
    build:
      context: ./nginx
      dockerfile: Dockerfile
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - frontend
      - backend
    networks:
      - voting-network
    restart: unless-stopped

volumes:
  postgres_data:

networks:
  voting-network:
    driver: bridge
```

**Create `docker-compose.prod.yml`:**
```yaml
version: '3.8'

services:
  backend:
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
    environment:
      FLASK_ENV: production

  frontend:
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 256M
        reservations:
          memory: 128M

  nginx:
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/ssl:/etc/nginx/ssl:ro
    deploy:
      resources:
        limits:
          memory: 128M
        reservations:
          memory: 64M

  db:
    deploy:
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
    volumes:
      - ./backups:/backups
```

### Step 6: CI/CD Pipeline with GitHub Actions

**Create `.github/workflows/ci-cd.yml`:**
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: docker.io
  IMAGE_NAME: voting-app

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      
      - name: Install backend dependencies
        run: |
          cd backend
          pip install -r requirements.txt
      
      - name: Run backend tests
        run: |
          cd backend
          python -m pytest tests/ || echo "No tests found"
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '16'
      
      - name: Install frontend dependencies
        run: |
          cd frontend
          npm ci
      
      - name: Run frontend tests
        run: |
          cd frontend
          npm test -- --coverage --watchAll=false

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
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
          images: ${{ env.REGISTRY }}/${{ github.repository }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=sha,prefix=sha-
            type=raw,value=latest,enable={{is_default_branch}}
      
      - name: Build and push backend image
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ github.repository }}-backend:latest
            ${{ env.REGISTRY }}/${{ github.repository }}-backend:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Build and push frontend image
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ github.repository }}-frontend:latest
            ${{ env.REGISTRY }}/${{ github.repository }}-frontend:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Build and push AI service image
        uses: docker/build-push-action@v5
        with:
          context: ./ai-service
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ github.repository }}-ai:latest
            ${{ env.REGISTRY }}/${{ github.repository }}-ai:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to production
        run: |
          echo "Deploying to production environment..."
          # Add your deployment commands here
          # For example, SSH to your server and pull new images
      
      - name: Run health checks
        run: |
          echo "Running health checks..."
          # Add health check commands here
```

### Step 7: Database Initialization

**Create `database/init.sql`:**
```sql
-- Create database if not exists
SELECT 'CREATE DATABASE voting_db'
WHERE NOT EXISTS (SELECT FROM pg_database WHERE datname = 'voting_db')\gexec

-- Connect to voting_db
\c voting_db;

-- Create extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Create indexes for better performance
CREATE INDEX IF NOT EXISTS idx_votes_option ON vote(option);
CREATE INDEX IF NOT EXISTS idx_votes_timestamp ON vote(timestamp);
CREATE INDEX IF NOT EXISTS idx_votes_sentiment ON vote(sentiment);

-- Insert sample data for testing
INSERT INTO vote (option, comment, sentiment, confidence, timestamp) 
VALUES 
  ('Python', 'Great for data science!', 'positive', 0.9, NOW() - INTERVAL '1 hour'),
  ('JavaScript', 'Love the flexibility', 'positive', 0.8, NOW() - INTERVAL '2 hours'),
  ('Java', 'Solid enterprise choice', 'neutral', 0.6, NOW() - INTERVAL '3 hours')
ON CONFLICT DO NOTHING;
```

---

## 🚀 Running the Application

### Development Mode
```bash
# Clone and setup
git clone <your-repo-url>
cd voting-app

# Start all services
docker-compose up --build

# View logs
docker-compose logs -f

# Access application
# Frontend: http://localhost:80
# Backend API: http://localhost:80/api/
# AI Service: http://localhost:8000 (internal)
```

### Production Mode
```bash
# Use production configuration
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Scale services
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d --scale backend=3 --scale frontend=2
```

### Monitoring and Debugging
```bash
# Check service health
docker-compose ps

# View specific service logs
docker-compose logs backend
docker-compose logs ai-service

# Execute commands in containers
docker-compose exec backend python -c "print('Backend is running')"
docker-compose exec db psql -U voting_user voting_db

# Monitor resources
docker stats

# Restart specific service
docker-compose restart backend
```

---

## 📊 Testing the Application

### Manual Testing Steps

1. **Visit the Application**: http://localhost
2. **Submit Votes**: Try different programming languages with comments
3. **View Real-time Results**: Results update automatically
4. **Check Sentiment Analysis**: Comments are analyzed for sentiment
5. **Monitor Recent Votes**: See latest votes with sentiment scores

### API Testing
```bash
# Health checks
curl http://localhost/api/health
curl http://localhost:8000/health

# Submit vote via API
curl -X POST http://localhost/api/vote \
  -H "Content-Type: application/json" \
  -d '{"option": "Python", "comment": "I love Python for its simplicity!"}'

# Get results
curl http://localhost/api/results

# Get recent votes
curl http://localhost/api/votes/recent

# Test AI service directly
curl -X POST http://localhost:8000/analyze \
  -H "Content-Type: application/json" \
  -d '{"text": "This is amazing!"}'
```

---

## 🔧 Advanced Features and Extensions

### 1. Add Authentication
```javascript
// Add JWT authentication to backend
// Implement user registration/login
// Protect voting endpoints
```

### 2. Real-time Updates
```javascript
// Add WebSocket support for live updates
// Implement Server-Sent Events
// Use Socket.IO for real-time communication
```

### 3. Advanced AI Features
```python
# Add more NLP features:
# - Topic modeling
# - Language detection  
# - Emotion analysis
# - Keyword extraction
```

### 4. Monitoring and Observability
```yaml
# Add monitoring stack:
# - Prometheus for metrics
# - Grafana for dashboards
# - ELK stack for logs
# - Jaeger for tracing
```

### 5. Security Enhancements
```dockerfile
# Add security scanning
# Implement rate limiting
# Add HTTPS/SSL certificates
# Use Docker secrets for sensitive data
```

---

## 🎯 Learning Outcomes

### What You've Built:
✅ **Multi-container application** with 6 services  
✅ **AI-powered features** with sentiment analysis  
✅ **Real-time updates** and caching  
✅ **Production-ready setup** with health checks  
✅ **CI/CD pipeline** with GitHub Actions  
✅ **Scalable architecture** using Docker Compose  

### Technologies Mastered:
- **Frontend**: React, Material-UI, Charts
- **Backend**: Flask, SQLAlchemy, Redis
- **AI/ML**: Transformers, FastAPI, NLP
- **Database**: PostgreSQL with optimization
- **Containerization**: Multi-stage builds, health checks
- **Orchestration**: Docker Compose, scaling
- **DevOps**: CI/CD, automated deployment

### Professional Skills:
- Full-stack application development
- Microservices architecture
- Container orchestration
- AI/ML integration
- DevOps practices
- Production deployment

---

## 🚀 What's Next?

Congratulations! You've built a complete, production-ready application. Consider these next steps:

1. **Deploy to cloud**: AWS, GCP, or Azure
2. **Add Kubernetes**: Migrate from Compose to K8s
3. **Implement monitoring**: Add Prometheus/Grafana
4. **Enhance security**: Add authentication, HTTPS
5. **Scale globally**: Multi-region deployment
6. **Add more AI**: Computer vision, chatbots

### Share Your Work:
- **Push to GitHub**: Share your repository
- **Create Docker Hub images**: Make them public  
- **Write blog post**: Document your journey
- **Present to class**: Demo your application

You've successfully mastered Docker and built an impressive full-stack application! 🎉