# Configuration System

## Overview

Podly uses a multi-layer configuration system combining environment variables, database settings, and runtime configuration.

## Configuration Layers

### Layer 1: Environment Variables (`.env.local`)
Sensitive credentials and deployment-specific settings.

**Required:**
```bash
# At minimum, one of these for LLM
GROQ_API_KEY=gsk_...
# OR
LLM_API_KEY=sk-...
```

**Optional:**
```bash
# Whisper backend
WHISPER_TYPE=groq  # local|remote|groq

# Auth
REQUIRE_AUTH=true
PODLY_ADMIN_USERNAME=admin
PODLY_ADMIN_PASSWORD=secure_pass
PODLY_SECRET_KEY=random_string

# Server
SERVER_THREADS=1
PORT=5001

# Discord SSO
DISCORD_CLIENT_ID=...
DISCORD_CLIENT_SECRET=...
DISCORD_REDIRECT_URI=...

# Stripe
STRIPE_SECRET_KEY=sk_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PRICE_ID=price_...
```

### Layer 2: Database Settings
Stored in SQLite, editable via web UI.

**LLM Settings:**
- llm_api_key, llm_model
- openai_base_url, openai_timeout
- openai_max_tokens
- llm_max_concurrent_calls
- llm_max_retry_attempts
- enable_boundary_refinement

**Whisper Settings:**
- whisper_type (local|remote|groq)
- local_model (base.en, small, medium, large)
- remote_api_key, remote_base_url
- groq_api_key, groq_model

**Processing Settings:**
- num_segments_to_input_to_prompt
- system_prompt_path
- user_prompt_template_path

**Output Settings:**
- fade_ms (default 50)
- min_ad_segement_separation_seconds (default 5)
- min_ad_segment_length_seconds (default 15)
- min_confidence (default 0.8)

**App Settings:**
- background_update_interval_minute
- automatically_whitelist_new_episodes
- post_cleanup_retention_days
- number_of_episodes_to_whitelist_from_archive_of_new_feed
- enable_public_landing_page
- user_limit_total
- autoprocess_on_download

### Layer 3: Runtime Config Singleton
In-memory configuration hydrated from database on startup.

Located in `app/runtime_config.py`:
```python
class RuntimeConfig:
    def __init__(self):
        self.llm_model = None
        self.whisper_type = None
        # ... etc

config = RuntimeConfig()  # Global singleton
```

## Configuration Flow

```
Environment Variables → Database → Runtime Config → Application
       (secrets)          (settings)    (in-memory)
         ↓                    ↓              ↓
    Startup load          Web UI edit   Fast access
    Write to DB           Save to DB    No DB hit
```

## Initialization

### Startup Sequence (`app/__init__.py`)

1. **Writer App:**
   - Run migrations (Alembic)
   - Bootstrap admin user
   - `ensure_defaults_and_hydrate()`
   - Seed database from env vars if empty

2. **Web App:**
   - `hydrate_runtime_config_inplace()`
   - Load config from database to memory
   - Initialize processor singleton

### Hydration (`app/config_store.py`)

```python
def ensure_defaults_and_hydrate():
    # 1. Check if settings tables have rows
    # 2. If empty, seed from environment variables
    # 3. Load all settings into RuntimeConfig singleton
    # 4. Return hydrated config
```

## Environment Variable Precedence

**On First Boot:**
- Env vars written to database if table empty
- Env vars take precedence over defaults

**After First Boot:**
- Database values used
- Env vars ignored (unless hash mismatch)
- Changes via web UI update database

**Force Reseed:**
- Delete database or settings rows
- Or set new env vars (hash change detected)

## Important Files

- `.env.local` - Local environment (gitignored)
- `.env.local.example` - Template with all options
- `app/config_store.py` - Config management logic
- `app/runtime_config.py` - Runtime singleton
- `shared/defaults.py` - Default values

## Web UI Configuration

Accessible at `/config` route:

**Sections:**
1. **LLM** - Model selection, API keys, rate limits
2. **Whisper** - Transcription backend settings
3. **Processing** - Prompt configuration
4. **Output** - Audio processing parameters
5. **App** - General application settings
6. **Discord** - SSO configuration

**Test Connections:**
- "Test LLM" button verifies API connectivity
- "Test Whisper" button tests transcription backend

## Programmatic Access

```python
# From anywhere in app
from app.runtime_config import config

# Read settings
model = config.llm_model
interval = config.background_update_interval_minute

# Settings are read-only at runtime
# Changes must go through writer service
```

## Security

- API keys stored encrypted in database (if field configured)
- Env file gitignored by default
- No secrets in logs
- Feed tokens provide limited access scope

## Migration

When adding new settings:
1. Add column to model in `app/models.py`
2. Add default in `shared/defaults.py`
3. Add to config store hydration
4. Add to web UI form
5. Create Alembic migration (ask user to generate)
