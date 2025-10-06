# Research: Multi-Word Text Replacements System

**Feature**: Multi-Word Text Replacements System
**Date**: 2025-10-06
**Status**: Complete

## Research Questions

Based on the Technical Context, the following areas require research for best practices and implementation patterns:

1. **Phrase matching algorithms** - How to efficiently match multi-word phrases
2. **Tauri state management** - Best practices for shared state in managers
3. **Settings schema extension** - How to extend tauri-plugin-store without breaking existing settings
4. **Performance optimization** - String matching at scale (500+ rules)

## Decisions & Rationale

### 1. Phrase Matching Algorithm

**Decision**: Use Aho-Corasick algorithm for multi-pattern string matching

**Rationale**:
- Aho-Corasick builds a finite state machine from all patterns, enabling O(n + m + z) time complexity where:
  - n = length of input text
  - m = total length of all patterns
  - z = number of matches found
- Significantly faster than naive string matching for multiple patterns
- Well-suited for 500+ replacement rules
- Available via `aho-corasick` crate (mature, widely used)

**Alternatives Considered**:
- Naive approach (iterate each rule, find/replace): O(p * n) where p = number of patterns. Too slow for 500+ rules.
- Regex-based: More flexible but slower and more complex. Spec requires exact matching only.
- Trie-based manual implementation: Reinventing the wheel when Aho-Corasick exists.

**Implementation Notes**:
- Build Aho-Corasick automaton once when rules load
- Rebuild when rules change
- Sort matches by position, then by length (longest first) for greedy matching
- Use case-insensitive matching as default, respect case_sensitive flag per rule

### 2. Tauri State Management Pattern

**Decision**: Create `TextReplacementsManager` following existing manager pattern

**Rationale**:
- Existing codebase uses manager pattern (AudioManager, ModelManager, TranscriptionManager)
- Each manager is `Arc<Mutex<T>>` for thread-safe shared access
- Managers registered in `lib.rs` via `.manage(manager)`
- Tauri commands access managers via `State<'_, Arc<Mutex<Manager>>>`

**Structure**:
```rust
pub struct TextReplacementsManager {
    rules: Vec<ReplacementRule>,
    automaton: Option<AhoCorasick>,
    store: Arc<StoreWrapper>, // Reference to tauri-plugin-store
}

impl TextReplacementsManager {
    pub fn new(store: Arc<StoreWrapper>) -> Self { ... }
    pub fn load_rules(&mut self) -> Result<()> { ... }
    pub fn apply_replacements(&self, text: &str) -> Result<String> { ... }
    pub fn add_rule(&mut self, rule: ReplacementRule) -> Result<()> { ... }
    // ... other CRUD methods
}
```

**Integration Points**:
- Initialize in `lib.rs::setup()` after store is created
- Pass to `TranscriptionManager` or call directly in transcription pipeline
- Commands in `commands/text_replacements.rs` use `State<Arc<Mutex<TextReplacementsManager>>>`

### 3. Settings Schema Extension

**Decision**: Add new key `text_replacements` to existing store, use Serde for serialization

**Rationale**:
- tauri-plugin-store supports arbitrary JSON keys
- Existing pattern: `custom_words: Vec<String>`, `settings: SettingsSchema`
- Add new key without breaking existing settings
- Use Serde's `#[serde(skip_serializing_if = "Vec::is_empty")]` for optional fields

**Schema**:
```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ReplacementRule {
    pub id: String,                  // UUID for identification
    pub spoken_phrase: String,       // What user dictates
    pub written_text: String,        // What appears (empty = deletion)
    pub case_sensitive: bool,        // Default false
    pub enabled: bool,               // Default true
    pub created_at: u64,             // Unix timestamp
    pub last_used_at: Option<u64>,   // Unix timestamp
    pub usage_count: u64,            // Analytics
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ReplacementConfig {
    pub version: String,             // Semver for migrations
    pub rules: Vec<ReplacementRule>,
}
```

**Migration Strategy**:
- Check if `text_replacements` key exists in store
- If missing, create empty config with version "1.0.0"
- If exists but old version, run migration logic
- Store as JSON array in tauri-plugin-store

### 4. Performance Optimization Strategy

**Decision**: Lazy rebuild of Aho-Corasick automaton, benchmark-driven optimization

**Rationale**:
- Building Aho-Corasick automaton has upfront cost (~O(m) where m = total pattern length)
- Rebuild only when rules change, not on every transcription
- Cache automaton in manager alongside rules
- Target: <100ms processing for 500-word transcription with 500 rules

**Optimization Techniques**:
1. **Lazy automaton rebuild**: Only rebuild when `rules` vector changes
2. **Case folding**: Use Aho-Corasick's case-insensitive mode for rules with `case_sensitive: false`
3. **Early exit**: If no matches found, return original text immediately
4. **Batch processing**: Process all matches in single pass (Aho-Corasick already does this)
5. **Memory efficiency**: Use `SmallVec` for small rule sets to avoid heap allocations

**Benchmarking Plan**:
- Create benchmark suite with criterion.rs
- Test scenarios: 10, 100, 500 rules
- Text sizes: 50, 500, 5000 words
- Ensure <100ms for typical case (500 words, 100 rules)

### 5. Non-Recursive Replacement Strategy

**Decision**: Apply replacements to original text only, mark replaced regions

**Rationale**:
- Spec requires non-recursive replacement (FR-010)
- If "A" → "B" and "B" → "C", result should be "B" not "C"
- Solution: Track byte ranges of applied replacements, skip overlapping matches

**Algorithm**:
```
1. Run Aho-Corasick on original text → get all match positions
2. Sort matches by position, then by length (descending) for greedy matching
3. Initialize set of "replaced ranges" (empty)
4. For each match:
   - If match range overlaps any replaced range, skip
   - Apply replacement
   - Add match range to "replaced ranges"
5. Return modified text
```

**Edge Case Handling**:
- Overlapping phrases: Longest match wins (greedy)
- Replacement text contains match phrase: Not re-replaced (marked as replaced)
- Word boundaries: Use Aho-Corasick with word boundary checking (or post-filter)

## Best Practices Research

### Rust String Manipulation
- Use `String::replace` for simple cases, but Aho-Corasick for multiple patterns
- Avoid repeated allocations: pre-allocate `String::with_capacity(text.len() + estimate)`
- Use `str::chars()` for unicode-aware operations (important for multi-language support)

### Tauri Command Patterns
- Commands should be lightweight wrappers around manager methods
- Use `#[tauri::command]` macro with proper error types
- Return `Result<T, String>` or custom error type for frontend consumption
- Log errors in command layer before returning to frontend

### React State Management
- Use custom hooks (`useTextReplacements`) to encapsulate Tauri invoke calls
- Optimistic updates: Update UI immediately, rollback on error
- Use `useCallback` for stable function references in event handlers
- Zod schemas for runtime validation of data from backend

### Testing Strategy
- Contract tests: Verify Tauri command input/output schemas
- Unit tests: Test replacement algorithm in isolation (various edge cases)
- Integration tests: Test full workflow (add rule → dictate → verify replacement)
- Property-based testing: Use `proptest` for fuzzing replacement logic

## Dependencies Evaluation

### Required Crates (Rust)
- `aho-corasick` (^1.1): Multi-pattern string matching
- `uuid` (^1.6): Generate IDs for replacement rules
- `serde` (already in project): JSON serialization
- `tauri` (already in project): Framework
- `tauri-plugin-store` (already in project): Settings storage

### Required Packages (TypeScript)
- `zod` (already in project): Schema validation
- `react` (already in project): UI framework
- `@tauri-apps/api` (already in project): Tauri bindings

### Development Dependencies
- `criterion` (Rust): Benchmarking
- `proptest` (Rust): Property-based testing
- `vitest` (TypeScript): Frontend testing

## Integration Patterns

### Transcription Pipeline Integration

**Current Pipeline** (from CLAUDE.md and spec notes):
```
Audio Recording → VAD Filtering → Whisper Transcription →
custom_words (fuzzy matching) → Paste to app
```

**Updated Pipeline**:
```
Audio Recording → VAD Filtering → Whisper Transcription →
[NEW] text_replacements (exact phrase matching) →
custom_words (fuzzy single-word matching) → Paste to app
```

**Integration Point**: In `managers/transcription.rs`, add call to `TextReplacementsManager::apply_replacements()` after Whisper output, before `apply_custom_words()`.

**Pseudocode**:
```rust
// In transcription.rs
let transcript = whisper_model.transcribe(audio)?;

// Apply text replacements (new)
let transcript = text_replacements_manager
    .lock()
    .unwrap()
    .apply_replacements(&transcript)?;

// Apply custom words (existing)
let transcript = apply_custom_words(&transcript, &custom_words);

// Paste to app (existing)
paste_text(&transcript)?;
```

### Settings UI Integration

**Existing Pattern**: Settings window with tabs (General, Audio, Models, Custom Words)

**Add New Tab**: "Text Replacements"

**UI Components**:
- `TextReplacements.tsx`: Main component
  - List of rules (table with spoken phrase, written text, enabled toggle)
  - Add rule button → modal/form
  - Edit rule (inline or modal)
  - Delete rule (with confirmation)
  - Import/Export buttons (file picker)
  - Search/filter for large rule sets

**State Management**:
- `useTextReplacements.ts` hook:
  - `rules: ReplacementRule[]` (fetched from backend on mount)
  - `addRule(rule)`, `updateRule(id, rule)`, `deleteRule(id)`
  - `importRules(file)`, `exportRules()` (JSON file operations)
  - Optimistic updates with rollback on error

## Risk Assessment

### Performance Risks
- **Risk**: 500 rules with 500-word transcription takes >100ms
- **Mitigation**: Benchmark early, optimize automaton build, consider caching
- **Fallback**: Warn user if rules exceed tested limits, suggest disabling rarely-used rules

### Compatibility Risks
- **Risk**: New settings schema breaks existing users
- **Mitigation**: Version settings, provide migration path, test upgrade scenarios
- **Fallback**: Ship empty config by default, users opt-in to feature

### UX Risks
- **Risk**: Users confused by interaction between text_replacements and custom_words
- **Mitigation**: Clear documentation, tooltips in UI, example rules
- **Fallback**: Document execution order explicitly in settings UI

## Open Questions

All questions resolved during research phase.

## References

- Aho-Corasick algorithm: https://en.wikipedia.org/wiki/Aho%E2%80%93Corasick_algorithm
- aho-corasick crate: https://docs.rs/aho-corasick/latest/aho_corasick/
- Tauri state management: https://tauri.app/v1/guides/features/command/#accessing-managed-state
- tauri-plugin-store: https://github.com/tauri-apps/tauri-plugin-store
