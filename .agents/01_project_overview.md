# Podly - Project Overview

## What is Podly?

Podly is an **ad-blocking service for podcasts** that creates ad-free RSS feeds. It uses AI to automatically detect and remove advertisement segments from podcast episodes.

## How It Works

1. **Request Episode** - User requests a podcast episode through the web UI or RSS feed
2. **Download** - Podly downloads the requested episode from the original RSS feed
3. **Transcribe** - Whisper (local or remote) transcribes the episode audio to text
4. **Identify Ads** - LLM (GPT/claude/etc.) analyzes transcription to label ad segments
5. **Remove Ads** - Audio processor removes identified ad segments
6. **Deliver** - Podly serves the ad-free version via a custom RSS feed URL

## Core Technologies

- **Backend**: Flask (Python) with SQLAlchemy ORM
- **Frontend**: React + TypeScript + Vite + Tailwind CSS
- **Database**: SQLite with WAL mode
- **Task Queue**: APScheduler for background jobs
- **AI/ML**: 
  - Whisper (OpenAI/local/Groq) for transcription
  - LiteLLM for LLM ad classification (OpenAI, Groq, etc.)
- **Audio Processing**: FFmpeg for audio manipulation

## Project Structure

```
Podcast-AdBlock/
├── src/                          # Main Python backend
│   ├── app/                      # Flask application
│   │   ├── auth/                 # Authentication system
│   │   ├── routes/               # API endpoints (blueprints)
│   │   ├── writer/               # Database writer service
│   │   ├── static/               # Static assets
│   │   └── templates/            # Jinja2 templates
│   ├── podcast_processor/        # Audio processing pipeline
│   ├── migrations/               # Alembic database migrations
│   └── main.py                   # Application entry point
├── frontend/                     # React frontend
├── docs/                         # Documentation
├── tests/                        # Test suite
├── compose.yml                   # Docker Compose configuration
└── run_podly_docker.sh          # Docker helper script
```

## Architecture Patterns

### Dual-App Architecture (Reader/Writer Pattern)
Podly uses a unique architecture to handle SQLite concurrency:
- **Web App (Reader)**: Handles HTTP requests, serves content, schedules jobs
- **Writer Service**: Dedicated process for all database writes via IPC queue
- **IPC Communication**: Uses Python multiprocessing Manager for command queue

This prevents SQLite write locks and deadlocks in multi-threaded environments.

### Processing Pipeline
- **PodcastProcessor**: Main coordinator
- **TranscriptionManager**: Handles Whisper transcription
- **AdClassifier**: LLM-based ad detection
- **AudioProcessor**: FFmpeg audio cutting
- **BoundaryRefiner**: Precision ad boundary detection

## Key Features

- Automatic podcast feed fetching and parsing
- AI-powered ad detection and removal
- User authentication (optional)
- Discord SSO integration
- Stripe billing integration
- Feed access tokens for secure sharing
- Background job processing
- Public landing page option

## Deployment Options

1. **Railway** (cloud hosting) - One-click deploy
2. **Local Docker** - Self-hosted with Docker Compose
3. **Bare metal** - Native Python installation

## Configuration

Configuration is managed via:
- Environment variables (`.env.local` file)
- Database settings (via web UI)
- Runtime config singleton

See `.env.local.example` for all available options.
