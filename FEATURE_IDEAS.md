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

---

## Next Steps (Updated)
1. ✅ Implement multi-word text replacements (HashMap approach)
2. ✅ Add voice formatting commands ("this/that", "quote unquote")
3. ✅ Phase 1: Hotkey to re-paste last transcription
4. 🔬 Prototype small LLM integration (Granite 3.1, Qwen3, Jan Nano via Ollama)
5. 🔬 Spike: Notion/Obsidian sync architecture
6. 🔬 Research: Additional STT model integration (Distil-Whisper, Moonshine)
7. 📝 Design: Human-readable replacement format (JSON/YAML export)
