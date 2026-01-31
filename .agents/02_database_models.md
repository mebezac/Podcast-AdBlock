# Database Models & Schema

## Core Models

### Feed
Podcast RSS feed definition
```python
- id: Integer (primary key)
- alt_id: Text (legacy compatibility)
- title: Text (required)
- description: Text
- author: Text
- rss_url: Text (unique, required)
- image_url: Text
- auto_whitelist_new_episodes_override: Boolean
- posts: Relationship to Post
- user_feeds: Relationship to UserFeed
- access_tokens: Relationship to FeedAccessToken
```

### Post
Individual podcast episode
```python
- id: Integer (primary key)
- feed_id: ForeignKey to Feed
- guid: Text (unique, required)
- download_url: Text (unique, required)
- title: Text (required)
- unprocessed_audio_path: Text
- processed_audio_path: Text
- description: Text
- release_date: DateTime
- duration: Integer
- whitelisted: Boolean (default False)
- image_url: Text
- download_count: Integer
- refined_ad_boundaries: JSON
- refined_ad_boundaries_updated_at: DateTime
- segments: Relationship to TranscriptSegment
- model_calls: Relationship to ModelCall
- processing_jobs: Relationship to ProcessingJob
```

### TranscriptSegment
Audio transcription segment
```python
- id: Integer (primary key)
- post_id: ForeignKey to Post
- sequence_num: Integer (required)
- start_time: Float (required)
- end_time: Float (required)
- text: Text (required)
- identifications: Relationship to Identification
```

### User
Application user account
```python
- id: Integer (primary key)
- username: String(255) (unique, required)
- password_hash: String(255) (required)
- role: String(50) (default 'user')
- feed_allowance: Integer (default 0)
- feed_subscription_status: String(32) (default 'inactive')
- stripe_customer_id: String(64)
- stripe_subscription_id: String(64)
- discord_id: String(32) (unique)
- discord_username: String(100)
- last_active: DateTime
- manual_feed_allowance: Integer
- user_feeds: Relationship to UserFeed
- feed_access_tokens: Relationship to FeedAccessToken
```

### ProcessingJob
Async job tracking
```python
- id: String(36) (primary key, UUID)
- jobs_manager_run_id: ForeignKey to JobsManagerRun
- post_guid: String(255) (required)
- status: String(50) (pending|running|completed|failed|cancelled|skipped)
- current_step: Integer (0-4)
- step_name: String(100)
- total_steps: Integer (default 4)
- progress_percentage: Float
- started_at: DateTime
- completed_at: DateTime
- error_message: Text
- scheduler_job_id: String(255)
- requested_by_user_id: ForeignKey to User
- billing_user_id: ForeignKey to User
```

### JobsManagerRun
Batch job run tracking
```python
- id: String(36) (primary key)
- status: String(50)
- trigger: String(100)
- started_at: DateTime
- completed_at: DateTime
- total_jobs: Integer
- queued_jobs: Integer
- running_jobs: Integer
- completed_jobs: Integer
- failed_jobs: Integer
- skipped_jobs: Integer
- context_json: JSON
- counters_reset_at: DateTime
```

## Settings Models (Singleton Tables)

All settings tables have `id = 1` as the only row:

### LLMSettings
- llm_api_key, llm_model, openai_base_url
- openai_timeout, openai_max_tokens
- llm_max_concurrent_calls, llm_max_retry_attempts
- enable_boundary_refinement, enable_word_level_boundary_refinement

### WhisperSettings
- whisper_type (local|remote|groq|test)
- local_model, remote_model, remote_api_key
- groq_api_key, groq_model

### ProcessingSettings
- num_segments_to_input_to_prompt
- system_prompt_path, user_prompt_template_path

### OutputSettings
- fade_ms, min_ad_segement_separation_seconds
- min_ad_segment_length_seconds, min_confidence

### AppSettings
- background_update_interval_minute
- automatically_whitelist_new_episodes
- post_cleanup_retention_days
- number_of_episodes_to_whitelist_from_archive_of_new_feed
- enable_public_landing_page, user_limit_total
- autoprocess_on_download

### DiscordSettings
- client_id, client_secret, redirect_uri
- guild_ids, allow_registration

## Junction Tables

### UserFeed
Many-to-many relationship between users and feeds they support
```python
- feed_id + user_id (unique constraint)
```

### FeedAccessToken
Secure token for RSS feed access without password
```python
- token_id: String(32) (unique, indexed)
- token_hash: String(64)
- token_secret: String(128)
- feed_id, user_id
- created_at, last_used_at, revoked
```

### Identification
Ad classification result linking segment to model call
```python
- transcript_segment_id + model_call_id + label (unique)
- confidence: Float
```

### ModelCall
Record of LLM API calls for ad classification
```python
- post_id, first_segment_sequence_num, last_segment_sequence_num
- model_name, prompt, response
- status, error_message, retry_attempts
```

## Database Configuration

- **Engine**: SQLite with WAL mode enabled
- **Pool Size**: 5 connections
- **Max Overflow**: 5 connections
- **Busy Timeout**: 90 seconds
- **Pragmas**: journal_mode=WAL, synchronous=NORMAL, wal_autocheckpoint=1000

## Important Rules

⚠️ **All database writes must go through the `writer` service.**
- Web app is configured with read-only sessions
- Direct `db.session.commit()` is prohibited in application code
- Use `writer_client.action()` or model operations instead
