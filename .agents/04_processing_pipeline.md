# Audio Processing Pipeline

## Overview

The podcast processing pipeline removes ads from podcast episodes through transcription, AI classification, and audio manipulation.

## Components

### 1. PodcastProcessor (`podcast_processor.py`)
Main coordinator class that orchestrates the entire workflow.

**Key Responsibilities:**
- Download audio from RSS feed
- Manage transcription process
- Coordinate ad classification
- Process audio to remove ads
- Handle errors and retries
- Lock management (prevents concurrent processing of same episode)

**Processing Steps:**
1. Download audio file
2. Transcribe audio to segments
3. Classify segments as ads/content
4. Refine ad boundaries (optional)
5. Process audio to remove ads
6. Update database with results

### 2. TranscriptionManager (`transcription_manager.py`)
Handles audio-to-text conversion.

**Supported Backends:**
- **Local Whisper**: Uses local whisper.cpp or faster-whisper
- **Remote Whisper**: OpenAI Whisper API
- **Groq**: Groq's whisper-large-v3-turbo

**Features:**
- Audio chunking for large files
- Progress tracking
- Retry logic with exponential backoff
- Token rate limiting

### 3. AdClassifier (`ad_classifier.py`)
Uses LLM to identify ad segments in transcripts.

**Process:**
1. Groups transcript segments into chunks
2. Sends chunks to LLM with prompt
3. Parses LLM response for ad labels
4. Stores identifications with confidence scores
5. Handles rate limiting and retries

**LLM Integration:**
- Uses LiteLLM for provider abstraction
- Supports OpenAI, Groq, Claude, etc.
- Configurable concurrency limits
- Token-per-minute rate limiting

### 4. BoundaryRefiner (`boundary_refiner.py`)
Fine-tunes ad segment boundaries for precise cutting.

**Purpose:** 
Intra-segment timestamp refinement to cut exactly at ad boundaries.

**Approach:**
- Re-analyzes audio around detected ad segments
- Uses word-level timestamps when available
- Produces refined start/end times

### 5. AudioProcessor (`audio_processor.py`)
Physical audio file manipulation using FFmpeg.

**Operations:**
- Detect silence (for boundary refinement)
- Cut segments from audio
- Add fade in/out at cut points
- Concatenate segments
- Preserve audio quality

**Configuration:**
- `fade_ms`: Milliseconds of fade at boundaries
- `min_ad_segment_length_seconds`: Minimum ad duration
- `min_ad_segement_separation_seconds`: Merge nearby ads

### 6. PodcastDownloader (`podcast_downloader.py`)
Downloads podcast episodes from RSS feeds.

**Features:**
- Follows redirects
- Handles various audio formats (MP3, M4A, etc.)
- Sanitizes filenames
- Resume capability
- Progress tracking

### 7. ProcessingStatusManager (`processing_status_manager.py`)
Tracks and reports processing progress.

**Tracks:**
- Current step (1-4)
- Step name ("Downloading", "Transcribing", "Classifying", "Processing")
- Progress percentage
- Error states

## Data Flow

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   Post      │────▶│  Downloader  │────▶│ Audio File  │
│   (GUID)    │     │              │     │  (in/jobs)  │
└─────────────┘     └──────────────┘     └─────────────┘
                                                │
                                                ▼
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   Output    │◀────│   Audio      │◀────│  Segments   │
│   (srv/)    │     │  Processor   │     │  (DB table) │
└─────────────┘     └──────────────┘     └─────────────┘
                                                ▲
┌─────────────┐     ┌──────────────┐            │
│  LLM API    │────▶│  Ad Classifier│────────────┘
│  (LiteLLM)  │     │              │
└─────────────┘     └──────────────┘
                            ▲
┌─────────────┐     ┌──────────────┐
│  Whisper    │────▶│ Transcription│
│  (local/API)│     │   Manager    │
└─────────────┘     └──────────────┘
```

## Configuration

Processing behavior controlled via:

**ProcessingSettings:**
- `num_segments_to_input_to_prompt`: Segments per LLM batch
- Prompt templates (system + user)

**OutputSettings:**
- `fade_ms`: Fade duration at cut boundaries
- `min_ad_segment_length_seconds`: Skip short ads
- `min_ad_segement_separation_seconds`: Merge close ads
- `min_confidence`: Minimum confidence threshold

**LLMSettings:**
- Model selection (gpt-4o, claude, etc.)
- API keys and endpoints
- Concurrency and rate limits
- Boundary refinement toggles

**WhisperSettings:**
- Backend selection (local/remote/Groq)
- Model sizes (base, small, medium, large)
- Timeout and retry settings

## Error Handling

- **Retry Logic**: Exponential backoff for API failures
- **Segment Recovery**: Partial results saved on failure
- **Lock Management**: Automatic lock release on error
- **Status Updates**: Real-time job status tracking

## Performance Considerations

- Audio files stored in `in/` (input) and `srv/` (output) directories
- Transcripts cached in database
- Concurrency limits on LLM calls
- SQLite WAL mode for better concurrency
