# Feature Ideas & Research Notes

## Current Findings & Decisions

### Text Replacement System Analysis
**Current State:**
- `custom_words` (Vec<String>) uses fuzzy matching for typo correction
- Only replaces ONE word at a time - not helpful for real use cases
- Threshold-based phonetic matching (Soundex + Levenshtein)

**Pain Points:**
- Adding custom words one-by-one is tedious
- No phrase/multi-word replacement
- No macro expansion capability

**Proposed Solution:**
- Add `text_replacements: HashMap<String, String>` to settings
- Hook after word correction in `transcription.rs:370-379`
- Support multi-word phrases: "email signature" → full signature block

### LLM Post-Processing
**Idea:** Small LLM for style/grammar enhancement
**Concerns:** Performance impact
**Research Findings (2025):**
- AssemblyAI's Slam-1 uses prompt-based customization (1,000 domain terms)
- VocalNet enables multi-token prediction for faster inference
- LLMs can do streaming responses + chunked processing

**Decision:** Worth prototyping with small model (Phi-3, TinyLlama)
- Process in chunks to maintain responsiveness
- Optional setting (off by default)
- System prompt for style (formal/casual/technical)

### Context-Aware Replacements
**Idea:** Different rules per application (code in VS Code, formal in email)
**Status:** BACKLOGGED - needs good UX/UI design plan
**Potential approach:** Per-app profiles with replacement sets

---

## Feature Suggestions

### ✅ Approved/Liked by User
1. **Macros/Templates** - Voice-triggered text expansion
2. **Voice Commands** - "undo that", "new line", formatting controls
3. **Multi-word Replacements** - Fix current one-word limitation

### 🔄 Already Exists
- Transcription History Panel (already implemented)

### 📋 Backlogged
- Context-Aware Replacements (needs frontend design)

---

## Additional Feature Ideas (Round 2)

### 1. ✅ Smart Punctuation & Formatting Commands
**Voice triggers for structure:**
- "bullet point" → inserts •
- "numbered list" → 1. 2. 3.
- "code block" → ```language```
- "capitalize that" → fixes previous word/sentence
- **"this slash that" → "this/that"** (common use case)
- **"quote unquote" → wraps previous word/phrase in quotes**

**Status:** APPROVED - Implement formatting voice commands

### 2. ✅ Clipboard Stacking/History (Phased Approach)
**Phase 1 (Priority):**
- Hotkey to paste last transcription at any time
- Simple recall of most recent text

**Phase 2:**
- Multi-slot clipboard system
- Voice commands: "save to slot 1", "paste slot 2"
- Quick access to last 5 transcriptions

**Status:** APPROVED - Start with Phase 1 hotkey

### 3. 📋 Domain-Specific Dictionaries
**Professional vocabularies:**
- Medical terms, legal jargon, technical acronyms
- Downloadable/shareable dictionary packs
- Auto-load based on active app (if context-aware later)

**Status:** BACKLOGGED - Lower priority vs replacements system

### 4. ⚠️ Audio Bookmarks & Corrections
**Quality of life:**
- "bookmark this" - saves timestamp in recording
- "scratch that" - removes last N seconds before transcription
- Replay last recording to verify accuracy

**Status:** REJECTED (not aligned with app goals) - EXCEPT "scratch that" command (keep)

### 5. ✅ Collaborative Snippets/Macro Library
**Community-driven:**
- Share macro sets (programming, writing, etc.)
- **Export/import as human-readable, editable format (JSON/YAML)**
- Built-in macro marketplace/store (future consideration)

**Status:** APPROVED - Readable format priority, store later

---

## Technical Research Notes

### Post-Processing Pipeline Enhancement
**Current Flow:**
1. Audio → Whisper → Raw text
2. `apply_custom_words()` - fuzzy correction
3. Paste to app

**Enhanced Flow:**
1. Audio → Whisper → Raw text
2. `apply_custom_words()` - fuzzy correction
3. **NEW:** `apply_text_replacements()` - exact phrase matching
4. **NEW:** `apply_llm_enhancement()` - optional style/grammar (chunked)
5. **NEW:** `apply_voice_commands()` - formatting/action commands
6. Paste to app

### Performance Considerations
- LLM: Use quantized models (GGUF format), 2-4 bit
- Chunking: Process sentences independently
- Async: Don't block main thread
- Toggle: Make LLM processing opt-in

### Settings Schema Updates Needed
```rust
pub struct AppSettings {
    // Existing
    pub custom_words: Vec<String>,

    // NEW
    pub text_replacements: HashMap<String, String>,  // phrase macros
    pub voice_commands: HashMap<String, VoiceAction>, // command → action
    pub llm_enhancement_enabled: bool,
    pub llm_style_prompt: String,
    pub domain_dictionaries: Vec<String>,  // loaded dictionary IDs
}
```

### Web Search Insights (Jan 2025)
- **AssemblyAI Slam-1**: Prompt-based customization with 1000 term capacity
- **VocalNet**: Multi-token prediction reduces latency
- **LLM integration**: Streaming responses + intent recognition
- **Quick Whisper**: Auto-paste with AI editing toggle
- **WhisperWriter**: Pause detection + auto-type workflow

---

---

## Model Expansion Opportunities

### New Small Transcription Models (2024-2025)
**Emerging alternatives to Whisper:**
- Distil-Whisper variants (faster, smaller)
- Moonshine (ultra-lightweight)
- NVIDIA Canary (multilingual)
- Faster-Whisper (optimized inference)
- Parakeet (already supported!)

**Status:** Research integration paths for model diversity

### Small LLM Post-Processing Options
**Recommended models for enhancement:**
1. **IBM Granite 3.1 (quantized via Ollama)** - 2B/8B variants, Apache 2.0 license
2. **Qwen2.5/Qwen3 (quantized)** - Excellent instruction following
3. **Ollama Cloud** - Powerful models, privacy-preserving per policy
4. **Jan Nano** - Huge context window, great performance
5. **Phi-3.5-mini** - Microsoft's efficient model

**Integration approach:**
- Use Ollama for local model management
- Optional cloud fallback for power users
- Sentence-level chunking for responsiveness

### Cloud Provider Integration
**Philosophy shift:** Local-first, cloud-optional
**Potential providers:**
- OpenAI Whisper API (official)
- AssemblyAI (Slam-1 with custom terms)
- Deepgram (real-time optimized)
- Azure Speech Services
- Google Cloud Speech-to-Text

**Requirements:**
- User opt-in (privacy conscious)
- API key management in settings
- Fallback to local models
- Clear indicator when using cloud

---

## Knowledge Base & Sync Integration

### Notion Database Sync
**Automatic knowledge building:**
- Every transcription → Notion page/database entry
- Metadata: timestamp, app context, model used
- Tagging system for organization
- Search across all historical transcriptions

### Obsidian Integration
**Local-first knowledge graph:**
- Save transcriptions as daily notes or individual files
- Automatic backlinking based on content
- Markdown format with frontmatter metadata
- Plugin-style integration

### Sync Architecture
```rust
pub struct SyncProvider {
    provider_type: SyncType, // Notion | Obsidian | Custom
    config: HashMap<String, String>,
    enabled: bool,
}

pub enum SyncType {
    Notion { api_key: String, database_id: String },
    Obsidian { vault_path: String, template: String },
    Custom { webhook_url: String },
}
```

**Implementation notes:**
- Async/non-blocking sync
- Retry logic with exponential backoff
- Local queue when offline
- Privacy: user controls what gets synced

---

## Enhanced Replacements System

### Improved "Replacements" Feature
**Beyond single-word correction:**
- Define common model mistakes → correct output
  - "API" → "API" (not "a p i")
  - "sequel" → "SQL"
  - "react component" → "React Component"
- Phrase-level replacements (not just fuzzy matching)
- Case-sensitive option per rule
- Regex support for advanced users

**Settings UI:**
```json
{
  "replacements": [
    { "spoken": "this slash that", "written": "this/that" },
    { "spoken": "quote unquote", "action": "wrap_quotes", "target": "previous_word" },
    { "spoken": "sequel", "written": "SQL", "case_sensitive": true }
  ]
}
```

### Replacement Rule Types
1. **Simple:** "spoken" → "written"
2. **Action-based:** "quote unquote" → wrap_quotes(previous_word)
3. **Regex:** `/api\s+(\w+)/i` → `API $1`
4. **Contextual:** Different replacements per app (future)

---

## Additional Feature Ideas (Round 3)

### 1. Multi-Model Switching & Benchmarking
**Model competition mode:**
- A/B test different models (Whisper vs Parakeet vs cloud)
- Accuracy tracking per model
- Auto-select best model based on language/domain
- "Try this with Granite" - reprocess last audio with different LLM

**Why it fits:** Leverages your interest in model diversity

### 2. Voice Command Language (VCL)
**Programmable voice actions:**
- Define custom commands with simple syntax
- "when I say X, do Y"
- Examples:
  - "insert date" → pastes current date
  - "python function" → inserts code template
  - "email John" → opens email with pre-filled recipient
- Human-readable config file (YAML/JSON)

**Why it fits:** Extends macro system with user-defined logic

### 3. Streaming Transcription with Live Edits
**Real-time corrections:**
- See transcription appear as you speak
- Voice command "fix that" → highlights last sentence for re-transcription
- Live LLM enhancement with diff view (see before/after)
- Undo/redo stack for iterative refinement

**Why it fits:** Combines LLM enhancement with interactive control

### 4. Multi-Language Smart Switching
**Context-aware language detection:**
- Auto-detect language per sentence (code-switching support)
- "Switch to Spanish" → changes model language
- Bilingual replacements: "por favor" → keeps as-is, "API" → "API"
- Per-language replacement rules

**Why it fits:** Builds on existing language support with smarter automation

### 5. Workflow Automation Triggers
**Post-transcription actions:**
- Auto-run commands based on content:
  - If contains "TODO:" → add to task manager
  - If starts with "Note:" → append to daily journal
  - If contains code → auto-format and save to snippets
- Webhooks for custom integrations
- Zapier/IFTTT-style rule builder

**Why it fits:** Aligns with knowledge sync, extends beyond just paste

### 6. Custom Webhook Integration
**Fire-and-forget HTTP callbacks:**
- POST transcription data to any URL
- Configurable payload format (JSON/form-data)
- Headers/auth token support
- Use cases:
  - Custom logging servers
  - Discord/Slack notifications
  - Personal automation services
  - Third-party integrations

**Why it fits:** Easy to implement, maximum flexibility for power users

### 7. Context-Aware Transcription Profiles
**Per-application/context rule sets:**
- Define different pipelines for different contexts
- Auto-switch based on active application
- Use cases:
  - Code editor: Insert code snippets, no capitalization
  - Email client: Formal language, full sentences
  - Slack/Discord: Casual tone, emoji shortcuts
  - Terminal: Command shortcuts, no punctuation
- Profile inheritance (base + overrides)

**Example profiles:**
```yaml
profiles:
  vscode:
    app_match: "Visual Studio Code|Code.exe"
    replacements:
      - "arrow function": "() => {}"
      - "console log": "console.log();"
    voice_commands:
      - "new line": "\n"
      - "comment this": "// "
    llm_style: "technical, concise"

  gmail:
    app_match: "Gmail|Mail.app"
    llm_style: "professional, complete sentences"
    auto_capitalize: true
    replacements:
      - "email signature": "[Full signature block]"
```

**Why it fits:** Solves the one-size-fits-all problem, makes STT context-intelligent

### 8. Desktop Audio Capture + Diarization
**Meeting transcription with speaker separation:**
- Capture system audio (desktop/loopback)
- Record speakers via microphone simultaneously
- **Diarization**: Separate speakers ("Speaker 1", "Speaker 2", etc.)
- Use cases:
  - Zoom/Teams/Meet transcription
  - Podcast recording notes
  - Interview documentation
  - Meeting minutes automation

**Technical approach:**
- Use `cpal` for loopback audio capture (Windows: WASAPI, macOS: BlackHole/ScreenCaptureKit, Linux: PulseAudio)
- Mix desktop + mic streams or keep separate tracks
- Diarization models:
  - **pyannote.audio** (best accuracy, needs Python bridge)
  - **WhisperX** (built-in diarization + alignment)
  - **SpeechBrain** (ONNX exportable)
- Output format with timestamps + speaker labels:
  ```json
  {
    "segments": [
      {"speaker": "Speaker 1", "start": 0.0, "end": 3.2, "text": "Hello everyone"},
      {"speaker": "Speaker 2", "start": 3.5, "end": 6.1, "text": "Thanks for joining"}
    ]
  }
  ```

**UI/UX:**
- New shortcut: "Record Meeting"
- Overlay shows: Recording (desktop + mic) with waveform
- Post-recording: Speaker label editor (rename "Speaker 1" → "John")
- Export as Markdown with speaker headers

**Why it fits:** Huge productivity boost for remote work, natural extension of audio capture

### 9. MCP (Model Context Protocol) Integration
**Post-paste hook to any MCP server:**
- Send transcripts to MCP servers for advanced processing
- Use cases:
  - **Memory MCPs**: Build long-term knowledge base (like Mem0)
  - **Tool MCPs**: Trigger actions based on content
  - **Data MCPs**: Store in databases, vector stores, etc.
- Run in Phase 2 (post-paste, async)

**Architecture:**
```rust
pub struct MCPConfig {
    server_url: String,           // stdio:// or http://
    server_command: Option<String>, // For stdio servers: "uvx mcp-server-memory"
    tools: Vec<String>,            // Which MCP tools to call
    payload_template: String,      // How to format transcript data
    enabled: bool,
}

async fn send_to_mcp(text: &str, config: &MCPConfig) -> Result<()> {
    match config.server_url.as_str() {
        url if url.starts_with("stdio://") => {
            // Launch stdio MCP server
            let mut child = Command::new("sh")
                .arg("-c")
                .arg(&config.server_command.as_ref().unwrap())
                .stdin(Stdio::piped())
                .stdout(Stdio::piped())
                .spawn()?;

            // Send JSON-RPC request
            let request = json!({
                "jsonrpc": "2.0",
                "id": 1,
                "method": "tools/call",
                "params": {
                    "name": "store_memory",
                    "arguments": {"content": text}
                }
            });

            child.stdin.write_all(request.to_string().as_bytes())?;
        },
        url if url.starts_with("http://") => {
            // HTTP MCP server
            reqwest::Client::new()
                .post(url)
                .json(&json!({"text": text}))
                .send()
                .await?;
        },
        _ => return Err(anyhow!("Unsupported MCP server type")),
    }
    Ok(())
}
```

**Example integrations:**
- **mcp-server-memory**: Auto-store transcripts in memory graph
- **mcp-server-sequential-thinking**: Analyze transcript for insights
- **mcp-server-github**: Auto-create issues from "TODO:" mentions
- **Custom MCP servers**: User-built integrations

**Why it fits:** As powerful as webhooks but with standardized protocol, enables AI-native workflows

### 10. Vector Embeddings + RAG for Transcripts
**Semantic search across all transcriptions:**
- Generate embeddings for each transcript (local models)
- Store in vector database (local-first: SQLite with vec0, or LanceDB)
- Semantic search: "Find all transcripts about API design"
- RAG integration: Use past transcripts as context for LLM

**Technical stack:**
```rust
// Embedding generation (local models)
pub enum EmbeddingModel {
    MiniLM,        // all-MiniLM-L6-v2 (384 dims, 80MB)
    BGESmall,      // bge-small-en-v1.5 (384 dims, 130MB)
    Nomic,         // nomic-embed-text-v1.5 (768 dims, 275MB)
}

// Vector storage
pub struct VectorStore {
    db: Connection,  // SQLite with vec0 extension
}

impl VectorStore {
    async fn add_transcript(&self, id: i64, text: &str, embedding: Vec<f32>) {
        // Store in SQLite with vector extension
        sqlx::query!(
            "INSERT INTO transcript_embeddings (transcript_id, embedding) VALUES (?, ?)",
            id,
            embedding
        ).execute(&self.db).await?;
    }

    async fn search(&self, query: &str, limit: usize) -> Vec<TranscriptMatch> {
        // Generate query embedding
        let query_embedding = generate_embedding(query).await?;

        // Cosine similarity search
        sqlx::query_as!(
            TranscriptMatch,
            "SELECT transcript_id, cosine_similarity(embedding, ?) as score
             FROM transcript_embeddings
             ORDER BY score DESC
             LIMIT ?",
            query_embedding,
            limit
        ).fetch_all(&self.db).await?
    }
}
```

**User-facing features:**
1. **Semantic search UI:**
   - Search bar: "transcripts about rust coding"
   - Results ranked by relevance (not just keyword match)
   - Highlight matching passages

2. **Auto-tagging:**
   - Cluster similar transcripts
   - Suggest tags based on embedding proximity
   - "You have 15 transcripts about 'meetings' - create tag?"

3. **RAG-enhanced features:**
   - Ask questions about all past transcripts
   - Summarize all transcripts from last week
   - Find patterns/trends in speaking habits

**Storage format:**
```sql
CREATE TABLE transcript_embeddings (
    transcript_id INTEGER PRIMARY KEY,
    embedding BLOB,  -- F32 array serialized
    model_version TEXT,
    created_at DATETIME,
    FOREIGN KEY (transcript_id) REFERENCES history(id)
);

CREATE INDEX idx_embedding ON transcript_embeddings USING vec0(embedding);
```

**Why it fits:** Makes the history system truly powerful, enables knowledge discovery over time

---

## Additional Feature Ideas (Round 4) - Based on Codebase Deep Dive

### 1. OCR-Based Learning Auto-Corrections
**Machine learning from user edits via local screen capture:**

**What:** After pasting transcription, capture screenshots before/after user edits, OCR them locally, diff the changes, and automatically learn correction patterns.

**Why it's not creepy:**
- 100% local OCR (on-device processing)
- Only captures the active window where text was pasted
- Only triggered when user explicitly used Handy
- User can review/approve learned corrections
- Toggle off in settings (off by default)

**How it works:**

1. **Paste detection pipeline:**
```rust
async fn paste_with_learning(text: String, app_handle: &AppHandle) {
    // Paste as usual
    paste_text(&text);

    // If learning enabled in settings
    if settings.enable_edit_learning {
        // Wait 100ms for paste to render
        sleep(Duration::from_millis(100));

        // Capture "before" state (active window only)
        let before_screenshot = capture_active_window();
        let before_region = detect_text_region(&before_screenshot);

        // Store metadata for later comparison
        save_pending_learning_session(PendingSession {
            original_text: text.clone(),
            before_screenshot,
            before_region,
            timestamp: now(),
        });

        // Set timer to check later (configurable, e.g., 5-30 seconds)
        spawn_learning_check_timer(text);
    }
}

async fn check_for_edits_after_delay(original_text: String) {
    // Wait for user to edit (configurable delay)
    sleep(Duration::from_secs(10));

    // Capture "after" state
    let after_screenshot = capture_active_window();

    // OCR both regions
    let before_text = ocr_region(&before_screenshot, &before_region);
    let after_text = ocr_region(&after_screenshot, &before_region);

    // Find what changed
    if before_text != after_text {
        let corrections = extract_corrections(&before_text, &after_text);

        // Add to learning database
        for correction in corrections {
            learn_correction(correction);
        }

        // Show notification: "Learned 3 corrections from your edits"
    }

    // Cleanup screenshots (privacy)
    delete_screenshots();
}
```

2. **OCR options:**
   - **Tesseract** - Classic, CPU-friendly, good for simple text
   - **PaddleOCR** - Modern, better accuracy, supports multiple languages
   - **EasyOCR** - Good balance, supports 80+ languages
   - **macOS:** Apple Vision framework (native, fast)
   - **Windows:** Windows.Media.OCR (native)

3. **Smart diff extraction:**
```rust
fn extract_corrections(original: &str, edited: &str) -> Vec<LearnedCorrection> {
    let mut corrections = vec![];

    // Word-level diff
    let diff = diff_words(original, edited);

    for change in diff {
        match change {
            // "api" -> "API" (capitalization)
            WordChange::Case(from, to) => {
                corrections.push(LearnedCorrection {
                    original: from,
                    corrected: to,
                    correction_type: CorrectionType::Capitalization,
                    confidence: 1,
                });
            },

            // "i phone" -> "iPhone" (spacing)
            WordChange::Merge(words, result) => {
                corrections.push(LearnedCorrection {
                    original: words.join(" "),
                    corrected: result,
                    correction_type: CorrectionType::Spacing,
                    confidence: 1,
                });
            },

            // "sequel" -> "SQL" (replacement)
            WordChange::Replace(from, to) => {
                corrections.push(LearnedCorrection {
                    original: from,
                    corrected: to,
                    correction_type: CorrectionType::Replacement,
                    confidence: 1,
                });
            },
        }
    }

    corrections
}
```

4. **Learning database:**
```rust
struct LearnedCorrection {
    original: String,
    corrected: String,
    correction_type: CorrectionType,
    times_seen: u32,           // Frequency
    confidence: f32,           // Auto-apply threshold
    last_seen: DateTime,
    user_approved: bool,       // Did user review and keep it?
}

impl LearnedCorrection {
    fn should_auto_apply(&self) -> bool {
        // Only auto-apply if seen 3+ times or user-approved
        self.times_seen >= 3 || self.user_approved
    }
}
```

5. **User experience:**

   **Initial learning phase:**
   - After paste, small notification: "Learning mode active - edit as usual"
   - 10 seconds later (configurable), captures edits
   - Notification: "Learned 2 new corrections" (clickable)

   **Review UI:**
   - Settings panel shows "Learned Corrections"
   - List of patterns with frequency
   - Approve/Reject/Edit each one
   - Toggle auto-apply per correction

   **Future transcriptions:**
   - High-confidence corrections apply automatically
   - Low-confidence show subtle indicator: "API*" (asterisk = applied learned correction)

**Privacy controls:**
- Toggle: "Enable edit learning" (off by default)
- Screenshots deleted immediately after OCR
- Only captures region where text was pasted (not whole screen)
- User can review/delete all learned corrections
- Clear warning in settings: "Will capture screenshots to learn from your edits"

**Challenges to solve:**

1. **OCR accuracy:**
   - Rich text formatting might confuse OCR
   - Need to handle different fonts/sizes
   - Solution: Focus on plain text editors first, expand later

2. **Finding the pasted text:**
   - Can't OCR entire screen (slow + inaccurate)
   - Solution: Track cursor position when pasting, OCR that region only

3. **User switched windows:**
   - If user pastes then switches apps, we capture wrong window
   - Solution: Store window handle/ID, only capture if still focused

4. **Performance:**
   - OCR can be CPU-intensive
   - Solution: Run in background thread, queue multiple jobs
   - Use native OS OCR when available (faster)

5. **When to capture "after":**
   - Too soon: User hasn't edited yet
   - Too late: User closed/switched
   - Solution: Configurable delay (5-30 sec), or detect "idle" state

**Achievability:** Medium-High
- OS screenshot APIs exist (platform-specific)
- Multiple OCR libraries available
- Text diff is solved problem
- Main complexity: coordination/timing

**Why this is innovative:**
- No other dictation app does this (AFAIK)
- Solves the "learning vocabulary" problem elegantly
- Zero user friction once enabled
- Works across ALL apps (not just specific editors)

### 2. Full-Spectrum Audio Visualizer
**Leverage the unused 16-channel FFT analysis:**

**Current state:** Codebase has full FFT visualizer (`audio_toolkit/audio/visualizer.rs`) with 16 frequency buckets, but overlay only shows 9 bars.

**Enhancement:**
- Expose all 16 frequency buckets in UI
- **Voice quality indicator:** Show if input is clear/muffled/noisy
- **Frequency-based triggers:** Whistle detection to start/stop recording
- **Recording quality preview:** Red indicator if audio too quiet/loud
- **Waveform + spectrum dual view:** Time-domain + frequency-domain

**UI mockup:**
```
┌─────────────────────────────┐
│  Recording...               │
│  ████████░░░░░░░░  Quality  │ ← 16-bar spectrum
│  ▁▂▃▅▆▇█▇▆▅▃▂▁     Waveform│ ← Time-domain
│  [Cancel]                   │
└─────────────────────────────┘
```

**Why it fits:** Feature already exists, just needs UI exposure

### 3. Model Benchmarking Dashboard
**A/B test transcription models automatically:**

**What:** Compare multiple models on same audio, track accuracy/speed metrics over time.

**Features:**
- **Test mode:** Transcribe with all downloaded models simultaneously
- **Accuracy tracking:** User marks transcriptions as "accurate" or "needs fixing"
- **Speed comparison:** Chart showing inference time per model
- **Language-specific recommendations:** "Whisper Large is 15% more accurate for Spanish"
- **Auto-model selection:** Based on language/content type/time-of-day

**Implementation:**
```rust
pub struct ModelBenchmark {
    model_id: String,
    avg_inference_ms: f64,
    accuracy_rating: f64,  // User-scored, 0-1
    total_transcriptions: u32,
    language_stats: HashMap<String, LanguageStats>,
}

async fn benchmark_mode(audio: Vec<f32>) -> Vec<TranscriptionResult> {
    let models = get_all_downloaded_models();
    let mut results = vec![];

    for model in models {
        let start = Instant::now();
        let text = transcribe_with_model(&audio, &model).await?;
        let duration = start.elapsed();

        results.push(TranscriptionResult {
            model_id: model.id,
            text,
            duration_ms: duration.as_millis(),
        });
    }

    // Show comparison UI for user to pick best
    show_benchmark_results(results);
}
```

**UI:**
```
Benchmark Results:
┌──────────────┬─────────────┬──────────┬─────────┐
│ Model        │ Result      │ Time     │ Quality │
├──────────────┼─────────────┼──────────┼─────────┤
│ Whisper-L    │ Hello world │ 850ms    │ ⭐⭐⭐⭐⭐ │
│ Parakeet-V3  │ Hello world │ 320ms    │ ⭐⭐⭐⭐   │
│ Whisper-M    │ Helo world  │ 520ms    │ ⭐⭐⭐    │
└──────────────┴─────────────┴──────────┴─────────┘
```

**Why it fits:** Existing dual-engine architecture makes this natural

### 4. Output Device Audio Feedback Expansion
**Leverage unused output device selection:**

**Current state:** Output device picker exists but only for start/stop sounds.

**Enhancements:**
- **TTS readback:** Read transcription aloud for proofreading
- **Custom sound effects:** User-defined WAV files for different events
- **Audio confirmation:** Speak back "Transcribed 45 words" after completion
- **Error sounds:** Different tones for different error types
- **Multi-language TTS:** Use OS TTS APIs for confirmation in any language

**Use cases:**
- Eyes-free workflow (driving, cooking, exercising)
- Accessibility for visually impaired users
- Audio confirmation when working in noisy environments

**Implementation:**
```rust
pub enum AudioFeedbackType {
    RecordingStart,
    RecordingStop,
    TranscriptionComplete { word_count: usize },
    TranscriptionReadback { text: String },
    Error { message: String },
    Custom { file_path: String },
}

async fn play_feedback(feedback: AudioFeedbackType, device: &str) {
    match feedback {
        AudioFeedbackType::TranscriptionReadback { text } => {
            // Use OS TTS
            let audio = text_to_speech(&text).await?;
            play_audio(audio, device).await?;
        },
        AudioFeedbackType::Custom { file_path } => {
            let audio = load_wav(&file_path)?;
            play_audio(audio, device).await?;
        },
        _ => {
            // Existing WAV file playback
            play_bundled_sound(feedback, device).await?;
        }
    }
}
```

**Why it fits:** Extends existing feature with minimal new code

### 5. History System Export & Sync
**Unlock the SQLite history database:**

**Current features:** SQLite storage, star/save, auto-cleanup (5 recent)

**New capabilities:**
- **Export formats:** JSON, CSV, Markdown, DOCX
- **Date range filters:** "Export all transcripts from last month"
- **Search & filter:** Full-text search across history
- **Backup/restore:** One-click backup of entire database
- **Cloud sync:** Optional sync to S3/Dropbox/Google Drive
- **Transcript editing:** Fix errors in saved transcriptions

**Export example:**
```rust
pub enum ExportFormat {
    Json,
    Csv,
    Markdown,
    Docx,
}

async fn export_history(format: ExportFormat, filter: HistoryFilter) -> Result<PathBuf> {
    let transcripts = db.query_history(filter).await?;

    match format {
        ExportFormat::Markdown => {
            let mut md = String::new();
            for t in transcripts {
                md.push_str(&format!("## {}\n\n{}\n\n", t.title, t.text));
            }
            save_file("transcripts.md", md)
        },
        ExportFormat::Json => {
            let json = serde_json::to_string_pretty(&transcripts)?;
            save_file("transcripts.json", json)
        },
        // ... other formats
    }
}
```

**Why it fits:** History database exists, just needs export/import interfaces

### 6. Smart Always-On Mic with Keyword Activation
**Enhance existing always-on mode:**

**Current:** Microphone stays active but requires shortcut to record

**Enhancement:**
- **Wake word detection:** "Hey Handy" starts recording automatically
- **Voice commands:** "Handy, record this" → starts transcription
- **Keyword spotting:** Detect specific words without full transcription (privacy-friendly)
- **Local processing:** All detection happens on-device

**Technical approach:**
- Use lightweight keyword spotting model (e.g., **Porcupine**, **Snowboy**)
- Only ~1MB model size
- <1% CPU usage for continuous monitoring
- Integrates with existing always-on mic mode

**Privacy features:**
- Wake word detection is separate from transcription
- Audio before wake word is discarded
- Visual indicator when listening vs recording vs transcribing

**Why it fits:** Builds on existing always-on microphone infrastructure

### 7. Transcription Confidence Scores & Partial Results
**Expose internal model confidence:**

**What:** Show word-level confidence from Whisper/Parakeet models, highlight uncertain words.

**Features:**
- **Confidence coloring:** High confidence (black), Low confidence (orange/red)
- **Interactive fixing:** Click uncertain words to re-transcribe that segment
- **Partial results:** Stream words as they're transcribed (real-time)
- **Confidence threshold:** Don't auto-paste if overall confidence < 80%

**UI example:**
```
Transcription:
"Hello <world> this is a <test>"
     ^^^^^                ^^^^^
     Low confidence - click to re-transcribe
```

**Implementation:**
```rust
pub struct TranscriptionResult {
    text: String,
    segments: Vec<Segment>,
    overall_confidence: f64,
}

pub struct Segment {
    text: String,
    confidence: f64,
    start_time: f64,
    end_time: f64,
}

async fn transcribe_with_confidence(audio: Vec<f32>) -> TranscriptionResult {
    let result = whisper_engine.transcribe_with_metadata(audio)?;

    // Extract confidence from model output
    let segments: Vec<Segment> = result.segments.iter().map(|s| {
        Segment {
            text: s.text.clone(),
            confidence: s.avg_logprob.exp(), // Convert log prob to 0-1
            start_time: s.start,
            end_time: s.end,
        }
    }).collect();

    TranscriptionResult {
        text: result.text,
        segments,
        overall_confidence: calculate_avg_confidence(&segments),
    }
}
```

**Why it fits:** Models already output confidence, just needs UI exposure

### 8. Clipboard History Integration
**Remember what we replace when pasting:**

**Current:** Clipboard is saved, pasted, then restored

**Enhancement:**
- **Clipboard stack:** Keep last 10 clipboard items
- **Paste alternatives:** Paste transcription OR previous clipboard
- **Append mode:** Add transcription to existing clipboard instead of replacing
- **Clipboard shortcuts:** Paste slot 1, 2, 3 via voice commands

**Implementation:**
```rust
pub struct ClipboardManager {
    history: VecDeque<String>,
    max_size: usize,
}

impl ClipboardManager {
    pub fn push(&mut self, text: String) {
        if self.history.len() >= self.max_size {
            self.history.pop_back();
        }
        self.history.push_front(text);
    }

    pub fn paste_slot(&self, slot: usize) -> Result<()> {
        let text = self.history.get(slot).context("Slot empty")?;
        paste_text(text)?;
        Ok(())
    }
}
```

**Voice commands:**
- "Save to clipboard slot 2"
- "Paste slot 1"
- "Append to clipboard"

**Why it fits:** Extends existing clipboard preservation logic

### 9. Tray Icon State Animations
**Leverage existing dynamic icons:**

**Current:** Tray icon changes based on state (Idle/Recording/Transcribing)

**Enhancement:**
- **Animated icons:** Spinning during transcription, pulsing during recording
- **Progress indicator:** Ring around icon shows % complete for downloads
- **Notification badges:** Number badge for unread transcriptions
- **Custom themes:** User-uploadable icon sets
- **Activity log:** Click icon to see recent transcription timeline

**Example:**
```rust
pub enum TrayIconState {
    Idle,
    Recording { duration_sec: u32 },
    Transcribing { progress: f32 },  // 0.0 - 1.0
    Downloading { model: String, progress: f32 },
    Error { message: String },
}

fn update_tray_icon(state: TrayIconState) {
    let icon_path = match state {
        TrayIconState::Recording { duration_sec } => {
            generate_recording_icon(duration_sec) // Animated frames
        },
        TrayIconState::Transcribing { progress } => {
            generate_progress_icon(progress) // Ring animation
        },
        // ... other states
    };

    tray.set_icon(icon_path)?;
}
```

**Why it fits:** Tray icon system exists, just needs animation frames

### 10. Debug Mode Power Features
**Expose hidden features from debug mode:**

**Current:** Debug mode shows word correction threshold slider and 5-second timeout

**Expand debug panel:**
- **VAD settings:** Onset/offset frames, noise floor threshold
- **Audio visualization:** Full 16-channel spectrum in real-time
- **Model internals:** Temperature, beam size, sampling method for Whisper
- **Performance metrics:** Chart of transcription time vs audio length
- **Raw logs viewer:** Real-time log streaming in UI
- **Audio buffer inspector:** See raw samples, FFT output, VAD decisions
- **Network inspector:** See webhook/sync API calls with timing

**Example debug UI:**
```
┌─────────────────────────────────────────┐
│ Debug Console                           │
├─────────────────────────────────────────┤
│ VAD Settings:                           │
│   Onset Frames: [15]                    │
│   Offset Frames: [15]                   │
│   Noise Floor: [-40 dB]                 │
│                                         │
│ Model Settings:                         │
│   Temperature: [0.0]                    │
│   Beam Size: [5]                        │
│                                         │
│ Performance:                            │
│   Last Transcription: 850ms             │
│   Model Load Time: 2.3s                 │
│   Audio Buffer: 3.2s / 48kHz            │
│                                         │
│ Live Logs:                              │
│   [INFO] Recording started              │
│   [DEBUG] VAD: speech detected          │
│   [INFO] Transcribing...                │
└─────────────────────────────────────────┘
```

**Why it fits:** Debug mode exists, just needs more controls exposed

---

## Post-Transcription Pipeline Architecture

### Pipeline Philosophy
**User-configurable processing chain that balances speed with features:**

```
Audio → Transcription → [PASTE IMMEDIATELY] → [LIVE REPLACE] → Final Text
                           ↓ Raw text          ↓ Enhanced text
                         (0ms delay)        (background processing)
```

### 🚀 **Key UX Innovation: Instant Paste + Live Enhancement**

**The Problem:** Traditional approaches delay paste until all processing completes.
**The Solution:** Paste raw transcription immediately, then upgrade it in real-time.

**Flow:**
1. **Transcribe** → Get raw text from model (e.g., "hello world this slash that")
2. **Instant Paste** → Paste raw text immediately with selection/highlight
3. **Background Enhancement** → Run pipeline asynchronously:
   - Text replacements: "this slash that" → "this/that"
   - Voice commands: "quote unquote" → wrap in quotes
   - LLM enhancement: grammar/style polish
4. **Live Replace** → Animated text replacement while keeping cursor position
   - Delete selected text
   - Type enhanced version character-by-character (or instant)
   - Visual effect shows improvement happening

**Visual Effect:**
```
[0ms]  Paste: "hello world this slash that"  ← User sees this instantly
                ^^^^^^^^^^^^^^^^^^^^^^^^
                (Text stays highlighted)

[200ms] Replace: "Hello world! This/that"  ← Smooth transition
                 ^^^^^^^^^^^^^^^^^^^^^^^^
                 (Selection preserved, or cursor at end)
```

**User Experience Benefits:**
- **Zero perceived latency** - text appears immediately
- **Transparent enhancement** - see the AI improving your text
- **Interruptible** - user can start typing to override
- **Visual feedback** - clear indication of what changed
- **Trust building** - shows what the model transcribed vs what was enhanced

**Technical Implementation:**
```rust
async fn transcribe_and_paste_with_enhancement(audio: Vec<f32>) -> Result<()> {
    // 1. Transcribe (blocking, but fast)
    let raw_text = transcription_manager.transcribe(audio)?;

    // 2. PASTE IMMEDIATELY with selection
    paste_text_with_selection(&raw_text)?;

    // 3. Spawn background enhancement
    let enhanced_text = tokio::spawn(async move {
        // Phase 1: Fast transformations
        let mut text = raw_text.clone();
        text = apply_text_replacements(&text, &settings.replacements);
        text = apply_voice_commands(&text, &settings.voice_commands);

        // Phase 2: LLM enhancement (optional, slower)
        if settings.llm_enhancement_enabled {
            text = apply_llm_enhancement(&text, &settings.llm_config).await?;
        }

        Ok::<String, Error>(text)
    });

    // 4. Wait for enhancement, then replace
    match enhanced_text.await {
        Ok(Ok(enhanced)) if enhanced != raw_text => {
            // Text was enhanced - do live replace
            replace_selected_text_with_animation(&enhanced).await?;

            // 5. Phase 3: Post-paste actions (don't wait)
            tokio::spawn(async move {
                run_post_paste_hooks(&enhanced).await;
            });
        },
        _ => {
            // No enhancement or error - text is already pasted, do nothing
        }
    }

    Ok(())
}

async fn replace_selected_text_with_animation(new_text: &str) -> Result<()> {
    // Delete current selection (raw text)
    simulate_key_press(Key::Backspace)?;

    // Type new text with optional animation
    if settings.show_enhancement_animation {
        // Character-by-character typing effect
        for char in new_text.chars() {
            simulate_key_char(char)?;
            sleep(Duration::from_millis(5)).await; // Subtle delay
        }
    } else {
        // Instant replacement
        paste_text(new_text)?;
    }

    Ok(())
}
```

**Settings Configuration:**
```rust
pub struct EnhancementSettings {
    // Core toggle
    instant_paste_enabled: bool,  // Default: true

    // Animation preferences
    show_enhancement_animation: bool,  // Default: false (instant replace)
    animation_speed_ms: u32,           // Default: 5ms per char

    // Fallback behavior
    max_enhancement_wait_ms: u32,      // Default: 2000ms (timeout)
    paste_raw_if_timeout: bool,        // Default: true (don't block user)

    // Visual feedback
    show_diff_notification: bool,      // Default: true ("Enhanced 3 words")
}
```

**Edge Cases:**
1. **User starts typing before enhancement completes:**
   - Cancel enhancement task
   - Keep user's edits
   - Don't replace

2. **Enhancement takes too long (>2s):**
   - Timeout, keep raw text
   - Run enhancements in background, save to history

3. **Enhancement fails (LLM error):**
   - Graceful degradation, keep raw text
   - Log error, show notification

4. **No enhancements needed:**
   - Skip replace step entirely
   - Still run post-paste hooks

**This Is A Differentiator:**
- No other STT app does instant paste + live enhancement
- Combines speed (raw paste) with quality (enhanced output)
- Visual transparency builds user trust
- Feels magical when it works smoothly

### Pipeline Stages (Execution Order - REVISED)

#### Phase 1: Instant Paste (0ms delay)
**Goal: Get SOMETHING to user immediately**
1. Transcribe with model (Whisper/Parakeet/Cloud)
2. → **PASTE RAW TEXT WITH SELECTION**

#### Phase 2: Live Enhancement (Background, 100-2000ms)
**Goal: Improve pasted text without blocking**
3. **Text Replacements** - Exact phrase substitution
4. **Voice Commands** - Format commands ("this/that", "quote unquote")
5. **LLM Enhancement** (optional) - Grammar/style if enabled
6. → **REPLACE SELECTED TEXT** (animated or instant)

#### Phase 3: Post-Paste Actions (Async, unlimited time)
**Goal: Sync, log, integrate without blocking**
7. **Webhook Triggers** - Fire custom HTTP callbacks
8. **Notion Sync** - Save to database with metadata
9. **Obsidian/GitHub Sync** - Commit to vault/repo
10. **Automation Triggers** - Content-based actions (TODO detection, etc.)
11. **MCP Integration** - Send to MCP servers for memory/tool actions
12. **Vector Embeddings** - Generate and store for RAG/search

### Configuration Structure
```rust
pub struct TranscriptionPipeline {
    // Phase 1: Blocking (pre-paste)
    replacements: Vec<ReplacementRule>,
    voice_commands: HashMap<String, VoiceCommand>,
    llm_enhancement: Option<LLMConfig>,

    // Phase 2: Non-blocking (post-paste)
    webhooks: Vec<WebhookConfig>,
    sync_providers: Vec<SyncProvider>,
    automation_rules: Vec<AutomationRule>,
}

pub struct WebhookConfig {
    url: String,
    method: HttpMethod,
    headers: HashMap<String, String>,
    auth_token: Option<String>,
    payload_format: PayloadFormat, // JSON | FormData | Custom
    enabled: bool,
}

pub enum PayloadFormat {
    Json {
        template: String, // e.g., {"text": "{{text}}", "timestamp": "{{timestamp}}"}
    },
    FormData {
        fields: HashMap<String, String>,
    },
    Custom {
        body: String, // Raw body with template variables
    },
}
```

### Execution Flow
```rust
async fn process_transcription(audio: Vec<f32>) -> Result<String> {
    // 1. Transcribe
    let raw_text = transcription_manager.transcribe(audio)?;

    // 2. Phase 1: Blocking transformations
    let processed_text = apply_blocking_pipeline(&raw_text)?;

    // 3. PASTE IMMEDIATELY
    paste_to_application(&processed_text)?;

    // 4. Phase 2: Background processing (spawn async tasks)
    tokio::spawn(async move {
        run_post_paste_pipeline(&processed_text).await;
    });

    Ok(processed_text)
}

async fn run_post_paste_pipeline(text: &str) {
    // Parallel execution of independent tasks
    let (webhook_results, sync_results, automation_results) = tokio::join!(
        fire_webhooks(text),
        sync_to_providers(text),
        run_automation_triggers(text),
    );

    // Log any errors but don't block user
    if let Err(e) = webhook_results {
        eprintln!("Webhook error: {}", e);
    }
}
```

---

## Transcript Storage & Security

### Default Storage Format
**Human-readable, version-controllable:**
- Format: JSON Lines (`.jsonl`) - one transcript per line
- Location: `~/.handy/transcripts/YYYY-MM-DD.jsonl`
- Schema:
```json
{
  "timestamp": "2025-01-15T10:30:45Z",
  "text": "The actual transcribed text",
  "model": "whisper-large-v3",
  "language": "en",
  "duration_ms": 3420,
  "app_context": "VS Code",
  "metadata": {
    "custom_key": "custom_value"
  }
}
```

### Benefits of JSONL Format
- **Line-based:** Easy to append, no file rewriting
- **Readable:** Can `cat`, `grep`, `jq` for debugging
- **Parseable:** Every line is valid JSON
- **Git-friendly:** Clean diffs for version control
- **Shareable:** Export/import individual days or date ranges

### Encryption (Future Feature)
**Optional privacy layer:**
- Toggle: "Encrypt transcripts at rest"
- Method: Age encryption (simple, modern, CLI-friendly)
- Key management: User-provided passphrase or key file
- Format: Encrypted JSONL → `.jsonl.age`
- Decrypt on-demand for sync/search

```bash
# Encrypted storage
cat ~/.handy/transcripts/2025-01-15.jsonl.age | age -d -i ~/.handy/key.txt

# Automatic decryption for webhooks/sync
# Only happens when user enables those features
```

---

## GitHub Sync Integration

### Concept: Transcripts as Git Commits
**Automatic knowledge versioning:**
- Each transcription (or batch) → git commit
- Repo: User-specified GitHub repo (public/private)
- Format: Markdown files with frontmatter
- Organization: Daily files or per-transcription files

### File Structure Options
**Option 1: Daily aggregation**
```
transcripts/
├── 2025/
│   ├── 01/
│   │   ├── 2025-01-15.md
│   │   └── 2025-01-16.md
```

**Option 2: Individual files**
```
transcripts/
├── 2025-01-15T10-30-45-unique-id.md
├── 2025-01-15T14-22-13-unique-id.md
```

### Markdown Format
```markdown
---
timestamp: 2025-01-15T10:30:45Z
model: whisper-large-v3
language: en
duration: 3.42s
app: VS Code
tags: [coding, javascript]
---

# Transcription

The actual transcribed text goes here.

<!-- Metadata -->
- Model: Whisper Large v3
- Confidence: 0.94
```

### Git Sync Workflow
```rust
async fn sync_to_github(text: &str, metadata: TranscriptMetadata) {
    // 1. Load user config
    let config = get_github_sync_config();

    // 2. Clone/pull repo (or use existing local copy)
    let repo_path = ensure_repo_synced(&config.repo_url)?;

    // 3. Generate markdown file
    let file_path = generate_transcript_file(&repo_path, text, metadata)?;

    // 4. Git commit + push
    git_add_and_commit(&repo_path, &file_path, "feat: add transcription")?;
    git_push(&repo_path)?;
}
```

### Configuration
```rust
pub struct GitHubSyncConfig {
    repo_url: String,          // e.g., "git@github.com:user/transcripts.git"
    branch: String,            // e.g., "main"
    file_format: FileFormat,   // Daily | Individual
    auto_push: bool,           // Push immediately vs batch commits
    auth_method: GitAuth,      // SSH | Token
    enabled: bool,
}

pub enum FileFormat {
    DailyAggregation { path_template: String }, // "transcripts/{year}/{month}/{day}.md"
    Individual { path_template: String },       // "transcripts/{timestamp}-{id}.md"
}
```

---

## Pipeline Configuration UI/UX

### Settings Organization
```
Settings
├── Transcription
│   ├── Model Selection
│   └── Language
├── Pipeline (NEW)
│   ├── Text Replacements
│   │   └── [Manage Rules: JSON/YAML editor]
│   ├── Voice Commands
│   │   └── [Command mappings]
│   ├── LLM Enhancement
│   │   ├── [ ] Enable post-processing
│   │   └── Model: [Ollama/Cloud]
│   └── Post-Paste Actions
│       ├── Webhooks
│       │   └── [+ Add Webhook]
│       ├── Sync Providers
│       │   ├── [ ] Notion (configure)
│       │   ├── [ ] Obsidian (vault path)
│       │   └── [ ] GitHub (repo URL)
│       └── Automation Rules
│           └── [Content triggers]
└── Storage
    ├── Format: JSONL
    ├── Location: ~/.handy/transcripts/
    └── [ ] Encrypt at rest (future)
```

### Pipeline Execution Order (Visual)
```
┌─────────────┐
│   Audio     │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│  Transcription  │ ← Whisper/Parakeet/Cloud
└────────┬────────┘
         │
         ▼
    ┌────────────────────┐
    │ PHASE 1: BLOCKING  │
    │ ─────────────────  │
    │ 1. Replacements    │
    │ 2. Voice Commands  │
    │ 3. LLM (optional)  │
    └────────┬───────────┘
             │
             ▼
      ┌──────────────┐
      │  PASTE NOW!  │ ← User sees result
      └──────────────┘
             │
             ▼
    ┌──────────────────────┐
    │ PHASE 2: BACKGROUND  │
    │ ──────────────────── │
    │ 4. Webhooks (async)  │
    │ 5. Notion (async)    │
    │ 6. GitHub (async)    │
    │ 7. Obsidian (async)  │
    │ 8. Automation (async)│
    └──────────────────────┘
```

---

## Next Steps (Updated)
1. ✅ Implement multi-word text replacements (HashMap approach)
2. ✅ Add voice formatting commands ("this/that", "quote unquote")
3. ✅ Phase 1: Hotkey to re-paste last transcription
4. 🔬 Design post-transcription pipeline architecture (blocking vs async phases)
5. 🔬 Implement webhook system (easy win, maximum flexibility)
6. 🔬 Prototype small LLM integration (Granite 3.1, Qwen3, Jan Nano via Ollama)
7. 🔬 Spike: Notion/Obsidian/GitHub sync architecture
8. 🔬 Research: Additional STT model integration (Distil-Whisper, Moonshine)
9. 📝 Design: Human-readable transcript storage (JSONL format)
10. 📝 Design: Pipeline configuration UI/UX
