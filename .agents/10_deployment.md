# Deployment & Operations

## Deployment Options

### 1. Railway (Cloud - Easiest)

One-click cloud deployment with automatic HTTPS.

**Steps:**
1. Click [![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/podly)
2. Add environment variables in Railway dashboard
3. Deploy

**Pros:**
- Automatic HTTPS
- Managed database (SQLite still works)
- Easy scaling
- No server management

**Cons:**
- Costs ~$10/month for hobby tier
- Less control than self-hosted

### 2. Docker Self-Hosting (Recommended)

Run on your own server with Docker Compose.

**Requirements:**
- Linux server (Ubuntu/Debian preferred)
- Docker & Docker Compose installed
- 4GB+ RAM (8GB recommended)
- Domain name (optional but recommended)

**Setup:**
```bash
# Clone repository
git clone https://github.com/mebezac/Podcast-AdBlock.git
cd Podcast-AdBlock

# Configure
```

**Reverse Proxy (Nginx):**
```nginx
server {
    listen 80;
    server_name podly.example.com;
    
    location / {
        proxy_pass http://localhost:5001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**HTTPS with Let's Encrypt:**
```bash
# Using certbot
sudo certbot --nginx -d podly.example.com
```

### 3. Native Installation (Advanced)

Run directly on server without Docker.

**Requirements:**
- Python 3.10+
- FFmpeg installed
- Node.js 18+ (for frontend)
- systemd (for service management)

**Setup:**
```bash
# Install dependencies
sudo apt update
sudo apt install python3-pip ffmpeg nodejs npm

# Clone and setup
git clone https://github.com/mebezac/Podcast-AdBlock.git
cd Podcast-AdBlock

# Python setup
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Frontend build
cd frontend
npm install
npm run build
cd ..

# Copy frontend build
cp -r frontend/dist/* src/app/static/

# Run
export $(cat .env.local | xargs)
python src/main.py
```

## Docker Compose Reference

**Production (`compose.yml`):**
```yaml
services:
  web:
    build: .
    ports:
      - "5001:5001"
    volumes:
      - ./data:/app/src/instance/data
    env_file: .env.local
    environment:
      - PODLY_DISABLE_SCHEDULER=false
      
  writer:
    build: .
    command: python -m app.writer
    volumes:
      - ./data:/app/src/instance/data
    env_file: .env.local
    environment:
      - PODLY_DISABLE_SCHEDULER=true
```

**Development (`compose.dev.cpu.yml`):**
- CPU-only processing
- Volume mounts for live code editing
- Debug mode enabled

## Environment Variables for Production

**Required:**
```bash
# AI Services
GROQ_API_KEY=gsk_...  # OR
LLM_API_KEY=sk-...

# Security (enable for multi-user)
REQUIRE_AUTH=true
PODLY_ADMIN_USERNAME=admin
PODLY_ADMIN_PASSWORD=secure_random_password
PODLY_SECRET_KEY=openssl rand -base64 64
```

**Recommended:**
```bash
# Server
SERVER_THREADS=4
PORT=5001

# Whisper (Groq is fastest)
WHISPER_TYPE=groq
GROQ_API_KEY=gsk_...

# Database persistence
PODLY_INSTANCE_DIR=/app/data
```

## Data Persistence

**Docker Volumes:**
```yaml
volumes:
  - ./data:/app/src/instance/data
```

**Important Paths:**
- `sqlite3.db` - Main database
- `in/` - Downloaded audio files
- `srv/` - Processed audio files
- `logs/` - Application logs

**Backup:**
```bash
# Backup script
tar czf backup-$(date +%Y%m%d).tar.gz data/

# Restore
tar xzf backup-20240115.tar.gz
```

## Monitoring

**Health Check:**
```bash
curl http://localhost:5001/health
```

**Logs:**
```bash
# Docker
docker compose logs -f --tail 100

# Native
tail -f src/instance/logs/app.log
```

**Metrics:**
- Job success/failure rates in database
- Download counts per episode
- Processing duration tracking

## Scaling Considerations

**SQLite Limitations:**
- Single writer process (already handled by writer service)
- Not suitable for high-concurrency write scenarios
- For 100+ concurrent users, consider PostgreSQL migration

**Performance Tuning:**
- Increase `SERVER_THREADS` for more concurrent requests
- Use GPU for Whisper (much faster)
- Enable boundary refinement for better quality (slower)
- Adjust `background_update_interval_minute`

## Security Checklist

- [ ] Change default admin password
- [ ] Set strong `PODLY_SECRET_KEY`
- [ ] Enable `REQUIRE_AUTH` for multi-user
- [ ] Use HTTPS in production
- [ ] Restrict Discord guilds if using SSO
- [ ] Set rate limiting
- [ ] Regular backups
- [ ] Keep dependencies updated

## Troubleshooting Production

**High Memory Usage:**
- Whisper large models need 4GB+ RAM
- Use smaller models: `WHISPER_LOCAL_MODEL=base.en`
- Or use Groq API instead of local

**Slow Processing:**
- Enable GPU support if available
- Increase `llm_max_concurrent_calls` (watch rate limits)
- Use faster LLM (GPT-4 is faster than Claude)

**Database Locked Errors:**
- Verify writer service is running: `docker compose ps`
- Check writer logs: `docker compose logs writer`
- Restart both services

**Feed Not Updating:**
- Check background scheduler running
- Verify `background_update_interval_minute` is set
- Check feed URL is valid
- Review logs for errors

## Updating

**Docker:**
```bash
./run_podly_docker.sh -d down
git pull
./run_podly_docker.sh --build
./run_podly_docker.sh -d
```

**Database Migrations:**
```bash
docker exec -it podly alembic upgrade head
```

## Cost Estimates

**Self-Hosted (monthly):**
- VPS: $5-20
- API costs: $0-5 (depending on usage)
- **Total: $5-25**

**Railway:**
- Hosting: ~$10
- API costs: $0-5
- **Total: $10-15**

**API Usage (per 1 hour of audio):**
- Whisper (Groq): ~$0.006
- LLM (GPT-4o): ~$0.01-0.03
- **Total: ~$0.02-0.04 per episode**

## Support

- Discord: https://discord.gg/FRB98GtF6N
- Issues: GitHub Issues
- Docs: See `docs/` folder
