# Tauri Command Contracts: Text Replacements

**Feature**: Multi-Word Text Replacements System
**Date**: 2025-10-06
**Status**: Complete

## Overview

This document defines the Tauri command interface between the React frontend and Rust backend for the text replacements feature. All commands follow Tauri's command pattern with `#[tauri::command]` attribute.

## Command Namespace

All commands are prefixed with `text_replacements_` to avoid conflicts with existing commands.

## Commands

### 1. get_replacement_config

**Purpose**: Fetch the complete replacement configuration (all rules and metadata)

**Rust Signature**:
```rust
#[tauri::command]
pub async fn get_replacement_config(
    manager: State<'_, Arc<Mutex<TextReplacementsManager>>>
) -> Result<ReplacementConfig, String>
```

**Request** (from frontend):
```typescript
import { invoke } from '@tauri-apps/api/tauri';

const config = await invoke<ReplacementConfig>('get_replacement_config');
```

**Response**:
```json
{
  "version": "1.0.0",
  "rules": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "spoken_phrase": "email signature",
      "written_text": "Best regards,\nJohn Smith",
      "case_sensitive": false,
      "enabled": true,
      "created_at": 1696550400,
      "last_used_at": 1696636800,
      "usage_count": 42
    }
  ],
  "last_modified": 1696636800
}
```

**Error Cases**:
- Store read failure: `"Failed to load replacement config: {error}"`
- Deserialization error: `"Invalid config format: {error}"`

**Contract Tests**:
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn test_get_replacement_config_returns_valid_config() {
        // Arrange: Initialize manager with test rules
        // Act: Call get_replacement_config
        // Assert: Returns ReplacementConfig with expected structure
    }

    #[test]
    fn test_get_replacement_config_empty() {
        // Arrange: Initialize manager with empty config
        // Act: Call get_replacement_config
        // Assert: Returns config with empty rules array
    }
}
```

---

### 2. add_replacement_rule

**Purpose**: Add a new replacement rule to the configuration

**Rust Signature**:
```rust
#[derive(Debug, Deserialize)]
pub struct AddRuleRequest {
    pub spoken_phrase: String,
    pub written_text: String,
    pub case_sensitive: Option<bool>,
}

#[tauri::command]
pub async fn add_replacement_rule(
    request: AddRuleRequest,
    manager: State<'_, Arc<Mutex<TextReplacementsManager>>>
) -> Result<ReplacementRule, String>
```

**Request** (from frontend):
```typescript
interface AddRuleRequest {
  spokenPhrase: string;
  writtenText: string;
  caseSensitive?: boolean; // Optional, defaults to false
}

const newRule = await invoke<ReplacementRule>(
  'add_replacement_rule',
  {
    request: {
      spokenPhrase: 'email signature',
      writtenText: 'Best regards,\nJohn Smith',
      caseSensitive: false,
    }
  }
);
```

**Response**: Returns the created `ReplacementRule` with generated `id` and timestamps.

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "spoken_phrase": "email signature",
  "written_text": "Best regards,\nJohn Smith",
  "case_sensitive": false,
  "enabled": true,
  "created_at": 1696550400,
  "last_used_at": null,
  "usage_count": 0
}
```

**Validation**:
- `spoken_phrase`: Must not be empty after trim, max 500 chars
- `written_text`: Max 5000 chars
- Duplicate `spoken_phrase` allowed (user may want multiple variations)

**Error Cases**:
- Empty spoken phrase: `"Spoken phrase cannot be empty"`
- Exceeds length limit: `"Spoken phrase exceeds 500 character limit"`
- Store write failure: `"Failed to save rule: {error}"`

**Contract Tests**:
```rust
#[test]
fn test_add_replacement_rule_success() {
    // Arrange: Create valid AddRuleRequest
    // Act: Call add_replacement_rule
    // Assert: Returns rule with generated ID and timestamps
}

#[test]
fn test_add_replacement_rule_empty_phrase() {
    // Arrange: Create request with empty spoken_phrase
    // Act: Call add_replacement_rule
    // Assert: Returns error "Spoken phrase cannot be empty"
}

#[test]
fn test_add_replacement_rule_max_length() {
    // Arrange: Create request with 501-char spoken_phrase
    // Act: Call add_replacement_rule
    // Assert: Returns error about character limit
}
```

---

### 3. update_replacement_rule

**Purpose**: Update an existing replacement rule

**Rust Signature**:
```rust
#[derive(Debug, Deserialize)]
pub struct UpdateRuleRequest {
    pub id: String,
    pub spoken_phrase: String,
    pub written_text: String,
    pub case_sensitive: bool,
    pub enabled: bool,
}

#[tauri::command]
pub async fn update_replacement_rule(
    request: UpdateRuleRequest,
    manager: State<'_, Arc<Mutex<TextReplacementsManager>>>
) -> Result<ReplacementRule, String>
```

**Request** (from frontend):
```typescript
interface UpdateRuleRequest {
  id: string;
  spokenPhrase: string;
  writtenText: string;
  caseSensitive: boolean;
  enabled: boolean;
}

const updatedRule = await invoke<ReplacementRule>(
  'update_replacement_rule',
  {
    request: {
      id: '550e8400-e29b-41d4-a716-446655440000',
      spokenPhrase: 'email sig',
      writtenText: 'Regards,\nJohn',
      caseSensitive: false,
      enabled: true,
    }
  }
);
```

**Response**: Returns the updated `ReplacementRule` with preserved metadata (created_at, usage stats).

**Behavior**:
- Preserves `created_at`, `last_used_at`, `usage_count` from original rule
- Updates only the specified fields
- Rebuilds automaton if `spoken_phrase` or `case_sensitive` changed

**Error Cases**:
- Rule not found: `"Rule with id {id} not found"`
- Invalid ID format: `"Invalid rule ID format"`
- Validation errors: Same as add_replacement_rule

**Contract Tests**:
```rust
#[test]
fn test_update_replacement_rule_success() {
    // Arrange: Add rule, create update request
    // Act: Call update_replacement_rule
    // Assert: Returns updated rule, preserves metadata
}

#[test]
fn test_update_replacement_rule_not_found() {
    // Arrange: Create update request with non-existent ID
    // Act: Call update_replacement_rule
    // Assert: Returns "Rule with id X not found"
}
```

---

### 4. delete_replacement_rule

**Purpose**: Delete a replacement rule by ID

**Rust Signature**:
```rust
#[tauri::command]
pub async fn delete_replacement_rule(
    id: String,
    manager: State<'_, Arc<Mutex<TextReplacementsManager>>>
) -> Result<(), String>
```

**Request** (from frontend):
```typescript
await invoke('delete_replacement_rule', { id: '550e8400-e29b-41d4-a716-446655440000' });
```

**Response**: `null` (success) or error string

**Error Cases**:
- Rule not found: `"Rule with id {id} not found"`
- Store write failure: `"Failed to delete rule: {error}"`

**Contract Tests**:
```rust
#[test]
fn test_delete_replacement_rule_success() {
    // Arrange: Add rule
    // Act: Call delete_replacement_rule with rule ID
    // Assert: Rule no longer in config
}

#[test]
fn test_delete_replacement_rule_not_found() {
    // Arrange: Empty config
    // Act: Call delete_replacement_rule with random ID
    // Assert: Returns "Rule with id X not found"
}
```

---

### 5. toggle_replacement_rule

**Purpose**: Enable or disable a replacement rule without deleting it

**Rust Signature**:
```rust
#[tauri::command]
pub async fn toggle_replacement_rule(
    id: String,
    enabled: bool,
    manager: State<'_, Arc<Mutex<TextReplacementsManager>>>
) -> Result<ReplacementRule, String>
```

**Request** (from frontend):
```typescript
const updatedRule = await invoke<ReplacementRule>(
  'toggle_replacement_rule',
  { id: '550e8400-e29b-41d4-a716-446655440000', enabled: false }
);
```

**Response**: Returns the updated `ReplacementRule`

**Behavior**:
- Updates only `enabled` field
- Rebuilds automaton to include/exclude rule

**Error Cases**:
- Rule not found: `"Rule with id {id} not found"`

**Contract Tests**:
```rust
#[test]
fn test_toggle_replacement_rule_disable() {
    // Arrange: Add enabled rule
    // Act: Call toggle_replacement_rule with enabled=false
    // Assert: Rule.enabled is false
}

#[test]
fn test_toggle_replacement_rule_enable() {
    // Arrange: Add disabled rule
    // Act: Call toggle_replacement_rule with enabled=true
    // Assert: Rule.enabled is true
}
```

---

### 6. import_replacement_rules

**Purpose**: Import replacement rules from a JSON file

**Rust Signature**:
```rust
#[derive(Debug, Deserialize)]
pub struct ImportRulesRequest {
    pub file_path: String,
    pub merge_strategy: MergeStrategy,
}

#[derive(Debug, Deserialize)]
pub enum MergeStrategy {
    /// Add imported rules, skip if spoken_phrase already exists
    Skip,
    /// Add imported rules, overwrite existing rules with same spoken_phrase
    Overwrite,
    /// Replace entire config with imported rules
    Replace,
}

#[tauri::command]
pub async fn import_replacement_rules(
    request: ImportRulesRequest,
    manager: State<'_, Arc<Mutex<TextReplacementsManager>>>
) -> Result<ImportResult, String>

#[derive(Debug, Serialize)]
pub struct ImportResult {
    pub imported_count: usize,
    pub skipped_count: usize,
    pub overwritten_count: usize,
}
```

**Request** (from frontend):
```typescript
interface ImportRulesRequest {
  filePath: string;
  mergeStrategy: 'Skip' | 'Overwrite' | 'Replace';
}

interface ImportResult {
  importedCount: number;
  skippedCount: number;
  overwrittenCount: number;
}

const result = await invoke<ImportResult>(
  'import_replacement_rules',
  {
    request: {
      filePath: '/Users/john/Downloads/handy-text-replacements-2025-10-06.json',
      mergeStrategy: 'Skip',
    }
  }
);
```

**Response**:
```json
{
  "imported_count": 15,
  "skipped_count": 3,
  "overwritten_count": 0
}
```

**Validation**:
- File must exist and be readable
- JSON must match ReplacementConfig schema
- Version must be compatible (currently "1.0.0")

**Error Cases**:
- File not found: `"File not found: {path}"`
- Invalid JSON: `"Invalid JSON format: {error}"`
- Version mismatch: `"Unsupported config version: {version}"`

**Contract Tests**:
```rust
#[test]
fn test_import_rules_skip_duplicates() {
    // Arrange: Create file with some duplicate spoken_phrases
    // Act: Import with Skip strategy
    // Assert: Only new rules imported, duplicates skipped
}

#[test]
fn test_import_rules_replace_all() {
    // Arrange: Existing config, create import file
    // Act: Import with Replace strategy
    // Assert: Old rules gone, only imported rules remain
}
```

---

### 7. export_replacement_rules

**Purpose**: Export all replacement rules to a JSON file

**Rust Signature**:
```rust
#[tauri::command]
pub async fn export_replacement_rules(
    file_path: String,
    manager: State<'_, Arc<Mutex<TextReplacementsManager>>>
) -> Result<usize, String>
```

**Request** (from frontend):
```typescript
const exportedCount = await invoke<number>(
  'export_replacement_rules',
  { filePath: '/Users/john/Downloads/handy-text-replacements-2025-10-06.json' }
);
```

**Response**: Number of rules exported (integer)

**Behavior**:
- Exports entire `ReplacementConfig` as JSON
- Creates file if doesn't exist, overwrites if exists
- Pretty-prints JSON for readability

**Error Cases**:
- Cannot write file: `"Failed to write file: {error}"`
- Permission denied: `"Permission denied: {path}"`

**Contract Tests**:
```rust
#[test]
fn test_export_rules_success() {
    // Arrange: Create config with rules
    // Act: Export to temp file
    // Assert: File exists, contains valid JSON, matches config
}

#[test]
fn test_export_rules_empty_config() {
    // Arrange: Empty config
    // Act: Export to temp file
    // Assert: File contains valid empty config
}
```

---

### 8. apply_text_replacements (Internal)

**Purpose**: Apply replacement rules to transcription text (called by TranscriptionManager, not frontend)

**Rust Signature**:
```rust
impl TextReplacementsManager {
    pub fn apply_replacements(&self, text: &str) -> Result<String, anyhow::Error> {
        // Not a Tauri command - internal API
    }
}
```

**This is NOT exposed as a Tauri command** - it's called internally by the transcription pipeline.

---

## Event Emissions

### replacement_rules_updated

**Purpose**: Notify frontend when replacement rules change (add/update/delete/import)

**Rust**:
```rust
use tauri::Manager;

// After rule modification
app.emit_all("replacement_rules_updated", &config)?;
```

**Frontend**:
```typescript
import { listen } from '@tauri-apps/api/event';

listen<ReplacementConfig>('replacement_rules_updated', (event) => {
  console.log('Rules updated:', event.payload);
  // Refresh UI
});
```

---

## Error Handling

All commands return `Result<T, String>` where:
- `Ok(T)`: Success with typed response
- `Err(String)`: Error with human-readable message

**Error message format**: `"{context}: {error_detail}"`

Examples:
- `"Failed to load replacement config: permission denied"`
- `"Rule with id 550e8400-e29b-41d4-a716-446655440000 not found"`
- `"Spoken phrase exceeds 500 character limit"`

Frontend should display error messages directly to user (they're user-facing, not debug messages).

---

## Testing Strategy

### Contract Test Suite

Create `/src-tauri/tests/contract/text_replacements.rs`:

1. **Schema tests**: Verify request/response JSON matches TypeScript schemas
2. **Success paths**: Test each command's happy path
3. **Error cases**: Test each documented error condition
4. **Edge cases**: Empty strings, max lengths, special characters
5. **Concurrency**: Multiple commands in parallel (via Tauri's async runtime)

### Integration Test Suite

Create `/src-tauri/tests/integration/text_replacements.rs`:

1. **Full workflows**: Add → Update → Delete → Verify gone
2. **Import/Export round-trip**: Export → Import → Verify identical
3. **Transcription integration**: Add rule → Trigger transcription → Verify replacement applied

### Frontend Tests

Create `/src/hooks/__tests__/useTextReplacements.test.ts`:

1. **Hook behavior**: Test CRUD operations via hook
2. **Optimistic updates**: Verify UI updates before backend confirms
3. **Error rollback**: Verify UI reverts on error
4. **Event handling**: Verify hook responds to backend events

---

## Related Documents

- **Data Model**: [data-model.md](../data-model.md) - Entity definitions
- **Feature Spec**: [spec.md](../spec.md) - Functional requirements
- **Implementation Plan**: [plan.md](../plan.md) - Overall strategy
