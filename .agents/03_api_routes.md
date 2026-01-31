# API Routes & Endpoints

## Route Organization

Routes are organized as Flask Blueprints in `src/app/routes/`:

### main_bp - Main Routes
- `GET /` - Landing page / dashboard
- `GET /index` - Index redirect
- `GET /health` - Health check endpoint

### feed_bp - Feed Management
- `GET /api/feeds` - List all feeds
- `POST /api/feeds` - Add new feed
- `GET /api/feeds/<id>` - Get feed details
- `PUT /api/feeds/<id>` - Update feed
- `DELETE /api/feeds/<id>` - Delete feed
- `POST /api/feeds/<id>/refresh` - Refresh feed from RSS
- `GET /api/feeds/<id>/rss` - Get ad-free RSS feed
- `POST /api/feeds/<id>/whitelist` - Whitelist all episodes
- `GET /feed/<token>/rss` - Public RSS with access token

### post_bp - Episode Management
- `GET /api/posts` - List posts (with filters)
- `GET /api/posts/<id>` - Get post details
- `POST /api/posts/<id>/download` - Download episode
- `POST /api/posts/<id>/process` - Process episode (remove ads)
- `POST /api/posts/<id>/whitelist` - Toggle whitelist status
- `GET /api/posts/<id>/audio` - Stream audio file
- `GET /api/posts/<id>/transcript` - Get transcript
- `POST /api/posts/<id>/transcribe` - Trigger transcription
- `GET /api/posts/<id>/status` - Get processing status
- `POST /api/posts/bulk-whitelist` - Bulk whitelist operation

### config_bp - Configuration
- `GET /api/config` - Get all settings
- `PUT /api/config` - Update settings
- `GET /api/config/llm` - Get LLM settings
- `PUT /api/config/llm` - Update LLM settings
- `GET /api/config/whisper` - Get Whisper settings
- `PUT /api/config/whisper` - Update Whisper settings
- `GET /api/config/processing` - Get processing settings
- `PUT /api/config/processing` - Update processing settings
- `GET /api/config/output` - Get output settings
- `PUT /api/config/output` - Update output settings
- `GET /api/config/app` - Get app settings
- `PUT /api/config/app` - Update app settings
- `POST /api/config/test-llm` - Test LLM connection
- `POST /api/config/test-whisper` - Test Whisper connection

### jobs_bp - Job Management
- `GET /api/jobs` - List all jobs
- `GET /api/jobs/<id>` - Get job details
- `POST /api/jobs/<id>/cancel` - Cancel job
- `GET /api/jobs/run/<run_id>` - Get run status
- `POST /api/jobs/run` - Trigger new run
- `DELETE /api/jobs/run/<run_id>` - Delete run

### auth_bp - Authentication
- `POST /api/auth/login` - Login
- `POST /api/auth/logout` - Logout
- `GET /api/auth/me` - Get current user
- `POST /api/auth/register` - Register new user (if enabled)
- `GET /api/auth/feed-tokens` - List feed access tokens
- `POST /api/auth/feed-tokens` - Create feed access token
- `DELETE /api/auth/feed-tokens/<id>` - Revoke token

### discord_bp - Discord SSO
- `GET /api/discord/login` - Initiate Discord OAuth
- `GET /api/discord/callback` - OAuth callback
- `POST /api/discord/link` - Link Discord to account

### billing_bp - Billing/Stripe
- `GET /api/billing/subscription` - Get subscription status
- `POST /api/billing/checkout` - Create checkout session
- `POST /api/billing/portal` - Create customer portal session
- `POST /api/billing/webhook` - Stripe webhook handler

## Authentication

Most endpoints require authentication when `REQUIRE_AUTH=true`:
- Session-based auth with cookies
- Feed tokens for RSS access (`?token=<token_id>:<secret>`)
- Rate limiting applied to auth endpoints

## API Response Format

Success:
```json
{
  "status": "success",
  "data": { ... }
}
```

Error:
```json
{
  "status": "error",
  "error": "Error message",
  "code": 400
}
```

## Feed RSS Format

Ad-free RSS feeds maintain original podcast metadata:
- Original title, description, author
- Episode GUIDs preserved for player compatibility
- Enclosure URLs point to processed audio
- Episode artwork retained

Access via:
- `/api/feeds/<id>/rss` (with session auth)
- `/feed/<token>/rss` (with token auth)
