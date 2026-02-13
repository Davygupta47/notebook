# Paper-to-Notebook Deployment Guide

This guide covers deploying the Paper-to-Notebook application using Docker, Render, and Vercel.

---

## Table of Contents
- [Prerequisites](#prerequisites)
- [Local Development](#local-development)
- [Docker Deployment](#docker-deployment)
- [Render Deployment](#render-deployment)
- [Vercel Deployment](#vercel-deployment)
- [Environment Variables](#environment-variables)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required
- **Google Gemini API Key**: Get it from [Google AI Studio](https://aistudio.google.com/apikey) (free tier available)
- **Python 3.11+** (for local development)
- **Docker** (for containerized deployment)
- **Git** (for version control)

### For Deployment
- **Render Account**: [Sign up at render.com](https://render.com) (free tier available)
- **Vercel Account** (optional): [Sign up at vercel.com](https://vercel.com) (free tier available)

---

## Local Development

### 1. Clone the Repository
```bash
git clone https://github.com/Davygupta47/notebook.git
cd notebook
```

### 2. Create Virtual Environment
```bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS/Linux
python3 -m venv venv
source venv/bin/activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

### 4. Set Environment Variables
```bash
# Create .env file from template
cp .env.example .env

# Edit .env and add your API key
# GOOGLE_API_KEY=your_actual_key_here
```

### 5. Run the Application
```bash
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

Navigate to: http://localhost:8000

---

## Docker Deployment

### Using Docker Directly

#### 1. Build the Image
```bash
docker build -t paper-to-notebook .
```

#### 2. Run the Container
```bash
docker run -d \
  --name paper-to-notebook \
  -p 8000:8000 \
  -e GOOGLE_API_KEY=your_api_key_here \
  -e MAX_UPLOAD_MB=30 \
  paper-to-notebook
```

#### 3. Access the Application
Open: http://localhost:8000

#### 4. Stop the Container
```bash
docker stop paper-to-notebook
docker rm paper-to-notebook
```

---

### Using Docker Compose

#### 1. Create Environment File
```bash
# Create .env file
echo "GOOGLE_API_KEY=your_api_key_here" > .env
```

#### 2. Start the Application
```bash
docker-compose up -d
```

#### 3. View Logs
```bash
docker-compose logs -f
```

#### 4. Stop the Application
```bash
docker-compose down
```

---

## Render Deployment

### Method 1: Deploy via Dashboard (Recommended)

#### 1. Fork/Push Repository
- Fork this repository to your GitHub account
- Or push your local changes to GitHub

#### 2. Create New Web Service
- Go to [Render Dashboard](https://dashboard.render.com/)
- Click **"New +"** → **"Web Service"**
- Connect your GitHub repository

#### 3. Configure Service
- **Name**: `paper-to-notebook` (or your choice)
- **Environment**: `Docker`
- **Region**: Choose closest to you
- **Branch**: `main`
- **Dockerfile Path**: `./Dockerfile` (auto-detected)
- **Instance Type**: `Free` (or choose paid for better performance)

#### 4. Add Environment Variables
Click **"Advanced"** and add:
```
GOOGLE_API_KEY = your_api_key_here
MAX_UPLOAD_MB = 30
PORT = 8000
```

#### 5. Deploy
- Click **"Create Web Service"**
- Wait 5-10 minutes for initial deployment
- Your app will be live at: `https://your-app-name.onrender.com`

---

### Method 2: Deploy via render.yaml (Blueprint)

#### 1. Update render.yaml
Edit `render.yaml` and ensure settings are correct:
```yaml
services:
  - type: web
    name: paper-to-notebook
    env: docker
    plan: free  # or 'starter' for paid
    region: oregon  # or your preferred region
    dockerfilePath: ./Dockerfile
```

#### 2. Deploy Blueprint
- In Render Dashboard, click **"New +"** → **"Blueprint"**
- Connect repository
- Select `render.yaml`
- Add `GOOGLE_API_KEY` in environment variables
- Deploy

---

### Method 3: Deploy via Render CLI

#### 1. Install Render CLI
```bash
# macOS
brew install render

# Windows/Linux
curl -sSL https://render.com/install.sh | bash
```

#### 2. Login
```bash
render login
```

#### 3. Deploy
```bash
render deploy
```

---

## Vercel Deployment

**Note**: Vercel is optimized for frontend/serverless. For full functionality, deploy the backend on Render and frontend on Vercel.

### Frontend-Only Deployment (Static UI)

#### 1. Prepare Vercel Config
The `vercel.json` is already configured. Update `API_ENDPOINT`:
```json
{
  "env": {
    "API_ENDPOINT": "https://your-render-backend.onrender.com"
  }
}
```

#### 2. Modify Frontend
Edit `static/index.html` to use `API_ENDPOINT`:
```javascript
// In fetch calls, use:
const API_BASE = process.env.API_ENDPOINT || '';
fetch(`${API_BASE}/api/generate`, { ... })
```

#### 3. Deploy to Vercel
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
vercel

# Or deploy via dashboard:
# 1. Go to https://vercel.com/new
# 2. Import your GitHub repository
# 3. Deploy
```

---

### Full-Stack Deployment (Hybrid)

**Architecture**: Frontend on Vercel + Backend on Render

#### 1. Deploy Backend to Render
Follow [Render Deployment](#render-deployment) steps above.

#### 2. Update CORS in app.py
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://your-vercel-app.vercel.app"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

#### 3. Deploy Frontend to Vercel
Follow [Frontend-Only Deployment](#frontend-only-deployment) above.

---

## Environment Variables

### Required Variables

| Variable | Description | Example | Required |
|----------|-------------|---------|----------|
| `GOOGLE_API_KEY` | Your Gemini API key | `AIza...` | ✅ Yes |

### Optional Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `MAX_UPLOAD_MB` | Max PDF size in MB | `30` | ❌ No |
| `PORT` | Server port | `8000` | ❌ No |

---

## Post-Deployment

### 1. Test the Deployment
```bash
# Health check
curl https://your-app-url.com/health

# Expected response:
# {"status":"ok"}
```

### 2. Upload Test PDF
- Navigate to your deployed URL
- Upload a sample research paper PDF
- Enter your API key
- Click "Generate Notebook"

### 3. Monitor Logs

**Render**:
```bash
# Via Dashboard: Logs tab in your service
# Via CLI:
render logs -s your-service-name -f
```

**Docker**:
```bash
docker logs -f paper-to-notebook
```

---

## Troubleshooting

### Common Issues

#### 1. **404 Model Not Found**
**Problem**: Gemini model not available in your region

**Solution**: Update `config.py`:
```python
DEFAULT_MODEL = "gemini-2.0-flash-exp"  # or another available model
```

#### 2. **Rate Limit Exceeded (429)**
**Problem**: Too many API requests

**Solution**: 
- Wait and retry (automatic retry is built-in)
- Upgrade your Gemini API plan
- Reduce concurrent users

#### 3. **PDF Too Large**
**Problem**: Upload fails with 413 error

**Solution**: 
- Compress PDF
- Increase `MAX_UPLOAD_MB` environment variable
- Split large papers

#### 4. **Connection Timeout**
**Problem**: Generation takes too long

**Solution**:
- Use a paid Render instance (more CPU/memory)
- Increase timeout in `config.py`:
```python
EXECUTE_TIMEOUT = 600  # seconds
```

#### 5. **Docker Build Fails**
**Problem**: Missing dependencies

**Solution**:
```bash
# Clean build
docker build --no-cache -t paper-to-notebook .

# Check logs
docker logs paper-to-notebook
```

#### 6. **CORS Errors (Vercel)**
**Problem**: Frontend can't access backend API

**Solution**: Add CORS middleware (see [Full-Stack Deployment](#full-stack-deployment))

---

## Performance Optimization

### 1. Use Paid Tier
Free tier on Render:
- Cold starts (~1 minute to wake up)
- Limited CPU/RAM
- May timeout on long generations

Paid tier ($7/month):
- Always warm
- Better performance
- More reliable

### 2. Optimize Docker Image
```dockerfile
# Use multi-stage build to reduce size
FROM python:3.11-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

FROM python:3.11-slim
COPY --from=builder /root/.local /root/.local
COPY . .
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 3. Add Caching
Consider adding Redis for:
- Analysis results caching
- Rate limiting
- Session management

---

## Security Best Practices

### 1. Never Commit API Keys
```bash
# Ensure .env is in .gitignore
echo ".env" >> .gitignore
```

### 2. Use Environment Variables
Always set sensitive data via environment variables, not hardcoded.

### 3. Enable HTTPS
Both Render and Vercel provide automatic HTTPS. Ensure:
- Use HTTPS URLs
- Set secure cookies (if using sessions)

### 4. Rate Limiting
Consider adding rate limiting:
```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@app.post("/api/generate")
@limiter.limit("5/minute")
async def generate(...):
    ...
```

---

## Monitoring

### 1. Set Up Alerts (Render)
- Go to service settings
- Enable email notifications for:
  - Deploy failures
  - Service downtime
  - High error rates

### 2. Check Logs Regularly
```bash
# Render
render logs -s paper-to-notebook --tail 100

# Docker
docker logs --tail 100 -f paper-to-notebook
```

### 3. Monitor API Usage
- Check [Google AI Studio](https://aistudio.google.com) for API usage
- Set up quota alerts

---

## Scaling

### Horizontal Scaling (Multiple Instances)

**Render**:
- Upgrade to Pro plan
- Enable autoscaling in service settings
- Set min/max instances

**Docker**:
```yaml
# docker-compose.yml
services:
  app:
    deploy:
      replicas: 3
    # ... rest of config
```

### Vertical Scaling (More Resources)

**Render**:
- Upgrade instance type (Starter/Standard)
- More CPU/RAM

---

## Updates & Maintenance

### Update Application

#### Docker
```bash
# Pull latest changes
git pull origin main

# Rebuild image
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

#### Render
- Auto-deploys on push to `main` branch (if enabled)
- Or manually trigger in dashboard: **"Manual Deploy" → "Deploy latest commit"**

### Update Dependencies
```bash
# Update requirements.txt
pip install --upgrade google-genai fastapi uvicorn

# Freeze new versions
pip freeze > requirements.txt

# Commit and push
git add requirements.txt
git commit -m "Update dependencies"
git push
```

---

## Support

### Resources
- **Documentation**: [implement.md](implement.md)
- **GitHub Issues**: [Report bugs](https://github.com/Davygupta47/notebook/issues)
- **Render Docs**: [render.com/docs](https://render.com/docs)
- **Vercel Docs**: [vercel.com/docs](https://vercel.com/docs)

### Getting Help
1. Check [Troubleshooting](#troubleshooting) section
2. Search existing GitHub issues
3. Create new issue with:
   - Deployment method (Docker/Render/Vercel)
   - Error logs
   - Steps to reproduce

---

## License

This project is open source under the MIT License. See [LICENSE](LICENSE) for details.

---

**Happy Deploying! 🚀**
