# Development Guide

## Prerequisites

- Docker and Docker Compose (recommended)
- Python 3.10+ (for native development)
- Node.js 18+ (for frontend development)
- FFmpeg (for audio processing)

## Quick Start (Docker)

```bash
# Clone repository
git clone https://github.com/mebezac/Podcast-AdBlock.git
cd Podcast-AdBlock

# Make script executable
chmod +x run_podly_docker.sh

# Build and run
./run_podly_docker.sh --build
./run_podly_docker.sh -d  # Detached mode

# Access app
open http://localhost:5001
```

## Docker Options

```bash
./run_podly_docker.sh --cpu     # Force CPU processing
./run_podly_docker.sh --gpu     # Force GPU processing (NVIDIA/ROCm)
./run_podly_docker.sh --build   # Just build, don't run
./run_podly_docker.sh --test-build  # Clean build from scratch
```

## Native Development Setup

### Backend Setup

```bash
# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# .venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt

# Set up environment
cp .env.local.example .env.local
# Edit .env.local with your keys

# Run migrations
cd src
alembic upgrade head

# Start backend
python main.py
```

### Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Start dev server
npm run dev

# Build for production
npm run build
```

## Project Structure Deep Dive

```
src/
├── app/                        # Flask application
│   ├── __init__.py            # App factory & setup
│   ├── models.py              # SQLAlchemy models
│   ├── extensions.py          # Flask extensions (db, migrate, scheduler)
│   ├── runtime_config.py      # Runtime configuration singleton
│   ├── config_store.py        # Config management
│   ├── feeds.py               # Feed business logic
│   ├── posts.py               # Post business logic
│   ├── processor.py           # Processor singleton
│   ├── background.py          # Background job setup
│   ├── logger.py              # Logging configuration
│   ├── db_guard.py            # Database guard utilities
│   ├── db_commit.py           # DB commit helpers
│   ├── ipc.py                 # Inter-process communication
│   ├── timeout_decorator.py   # Timeout utilities
│   ├── job_manager.py         # Job queue management
│   ├── jobs_manager.py        # Jobs manager v2
│   ├── jobs_manager_run_service.py  # Job runner service
│   ├── post_cleanup.py        # Cleanup utilities
│   ├── auth/                  # Authentication package
│   │   ├── __init__.py
│   │   ├── settings.py        # Auth configuration
│   │   ├── passwords.py       # Password hashing
│   │   ├── guards.py          # Auth decorators
│   │   ├── middleware.py      # Auth middleware
│   │   ├── service.py         # Auth business logic
│   │   ├── rate_limiter.py    # Rate limiting
│   │   ├── discord_service.py # Discord OAuth
│   │   ├── discord_settings.py
│   │   ├── bootstrap.py       # Admin bootstrap
│   │   ├── state.py           # Auth state
│   │   └── feed_tokens.py     # Feed token management
│   ├── routes/                # API blueprints
│   │   ├── __init__.py
│   │   ├── main_routes.py
│   │   ├── feed_routes.py
│   │   ├── post_routes.py
│   │   ├── config_routes.py
│   │   ├── jobs_routes.py
│   │   ├── auth_routes.py
│   │   ├── billing_routes.py
│   │   ├── discord_routes.py
│   │   └── post_stats_utils.py
│   ├── writer/                # Writer service
│   │   ├── __init__.py
│   │   ├── service.py         # Main writer loop
│   │   ├── client.py          # Writer client API
│   │   ├── executor.py        # Command executor
│   │   ├── protocol.py        # IPC protocol
│   │   ├── model_ops.py       # Model operations
│   │   └── actions/           # Action handlers
│   │       ├── __init__.py
│   │       ├── jobs.py
│   │       ├── feeds.py
│   │       ├── users.py
│   │       ├── system.py
│   │       ├── cleanup.py
│   │       └── processor.py
│   ├── static/                # Static assets
│   └── templates/             # Jinja2 templates
├── podcast_processor/         # Audio processing
│   ├── podcast_processor.py   # Main coordinator
│   ├── transcription_manager.py
│   ├── ad_classifier.py
│   ├── ad_merger.py
│   ├── audio_processor.py
│   ├── boundary_refiner.py
│   ├── word_boundary_refiner.py
│   ├── podcast_downloader.py
│   ├── processing_status_manager.py
│   ├── transcribe.py
│   ├── audio.py
│   ├── cue_detector.py
│   ├── prompt.py
│   ├── model_output.py
│   ├── llm_concurrency_limiter.py
│   ├── llm_error_classifier.py
│   ├── llm_model_call_utils.py
│   └── token_rate_limiter.py
├── migrations/                # Alembic migrations
│   ├── versions/              # Migration files
│   └── env.py
├── shared/                    # Shared utilities
│   ├── processing_paths.py
│   ├── config.py
│   ├── defaults.py
│   └── test_utils.py
└── main.py                    # Entry point
```

## Testing

```bash
# Run all tests
./scripts/ci.sh

# Run specific test
pytest tests/test_feeds.py -v

# Run with coverage
pytest --cov=src --cov-report=html
```

## Code Quality

```bash
# Format code
black src/

# Sort imports
isort src/

# Type check
mypy src/

# Lint
pylint src/

# Run all checks (CI)
./scripts/ci.sh
```

## Debugging

### Enable Debug Mode
```python
# In .env.local
FLASK_DEBUG=true
```

### Logs
- Application logs: `src/instance/logs/app.log`
- Writer logs: Same file (tagged with [WRITER])
- Docker logs: `docker compose logs -f`

### Database Inspection
```bash
# Access SQLite CLI
docker exec -it podly sqlite3 /app/src/instance/sqlite3.db

# Or use Python shell
docker exec -it podly python
>>> from app import create_app
>>> from app.models import *
>>> from app.extensions import db
```

## Common Development Tasks

### Add a New Route
1. Create handler in `app/routes/<name>_routes.py`
2. Create Blueprint
3. Register in `app/routes/__init__.py`
4. Add tests

### Add a Database Model
1. Define in `app/models.py`
2. Generate migration: `alembic revision --autogenerate -m "message"`
3. Apply: `alembic upgrade head`
4. Add to writer model_ops if needed

### Modify Processing Pipeline
1. Edit relevant file in `podcast_processor/`
2. Update `PodcastProcessor` coordinator if needed
3. Test with sample podcast
4. Check error handling

### Add Configuration Option
1. Add to model in `app/models.py`
2. Add default to `shared/defaults.py`
3. Add to config_store hydration
4. Add to web UI
5. Generate migration

## Environment Variables Reference

See `.env.local.example` for complete list.

## Docker Compose Files

- `compose.yml` - Production configuration
- `compose.dev.cpu.yml` - CPU-only development
- `compose.dev.nvidia.yml` - NVIDIA GPU support
- `compose.dev.rocm.yml` - AMD ROCm GPU support

## Troubleshooting

**Port already in use:**
```bash
lsof -i :5001 | grep LISTEN | awk '{print $2}' | xargs kill -9
```

**Database locked:**
- Check writer service is running
- Verify no stray Python processes
- Clear WAL files: `rm sqlite3.db-wal sqlite3.db-shm`

**Frontend not loading:**
- Check Vite dev server running on :5173
- Verify CORS origins in Flask config
- Check browser console for errors

## Contributing

See `docs/contributors.md` for contribution guidelines.
