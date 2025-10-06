# Data Model: Multi-Word Text Replacements System

**Feature**: Multi-Word Text Replacements System
**Date**: 2025-10-06
**Status**: Complete

## Overview

This document defines the data structures for the multi-word text replacements system. The model supports phrase pairs with metadata for management, analytics, and configuration.

## Core Entities

### ReplacementRule

Represents a single phrase replacement rule.

**Rust Definition**:
```rust
use serde::{Deserialize, Serialize};
use uuid::Uuid;

#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
pub struct ReplacementRule {
    /// Unique identifier for the rule
    pub id: String,

    /// The phrase as spoken by the user (source)
    /// Multi-word, e.g., "email signature", "sequel database"
    pub spoken_phrase: String,

    /// The text to insert in transcription (target)
    /// Can be multi-line, contain special chars, or be empty (for deletion)
    pub written_text: String,

    /// Whether to match case exactly (true) or ignore case (false)
    /// Default: false (case-insensitive matching)
    pub case_sensitive: bool,

    /// Whether this rule is active
    /// Allows temporary disabling without deletion
    /// Default: true
    pub enabled: bool,

    /// Unix timestamp (seconds) when rule was created
    pub created_at: u64,

    /// Unix timestamp (seconds) when rule was last used
    /// None if never used
    pub last_used_at: Option<u64>,

    /// Count of how many times this rule has been applied
    /// Used for analytics and sorting by popularity
    pub usage_count: u64,
}

impl ReplacementRule {
    /// Create a new replacement rule with generated ID and timestamps
    pub fn new(spoken_phrase: String, written_text: String) -> Self {
        Self {
            id: Uuid::new_v4().to_string(),
            spoken_phrase,
            written_text,
            case_sensitive: false,
            enabled: true,
            created_at: Self::current_timestamp(),
            last_used_at: None,
            usage_count: 0,
        }
    }

    /// Update last_used_at timestamp and increment usage_count
    pub fn mark_used(&mut self) {
        self.last_used_at = Some(Self::current_timestamp());
        self.usage_count += 1;
    }

    fn current_timestamp() -> u64 {
        std::time::SystemTime::now()
            .duration_since(std::time::UNIX_EPOCH)
            .unwrap()
            .as_secs()
    }
}
```

**TypeScript Definition**:
```typescript
import { z } from 'zod';

export const ReplacementRuleSchema = z.object({
  id: z.string().uuid(),
  spokenPhrase: z.string().min(1),
  writtenText: z.string(), // Empty string allowed (for deletion)
  caseSensitive: z.boolean().default(false),
  enabled: z.boolean().default(true),
  createdAt: z.number().int().positive(),
  lastUsedAt: z.number().int().positive().optional(),
  usageCount: z.number().int().nonnegative().default(0),
});

export type ReplacementRule = z.infer<typeof ReplacementRuleSchema>;
```

**Field Constraints**:
- `id`: UUID v4 format, unique across all rules
- `spoken_phrase`: Non-empty string, trimmed, max 500 characters
- `written_text`: String (may be empty), max 5000 characters
- `case_sensitive`: Boolean, default false
- `enabled`: Boolean, default true
- `created_at`: Unix timestamp in seconds, must be ≤ current time
- `last_used_at`: Unix timestamp in seconds or null, must be ≥ created_at if set
- `usage_count`: Non-negative integer

### ReplacementConfig

Container for all replacement rules with versioning for migrations.

**Rust Definition**:
```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ReplacementConfig {
    /// Semantic version for migration compatibility
    /// Current: "1.0.0"
    pub version: String,

    /// List of all replacement rules
    /// Order matters for display but not for matching (Aho-Corasick handles that)
    pub rules: Vec<ReplacementRule>,

    /// Unix timestamp when config was last modified
    pub last_modified: u64,
}

impl ReplacementConfig {
    /// Create empty config with current version
    pub fn new() -> Self {
        Self {
            version: "1.0.0".to_string(),
            rules: Vec::new(),
            last_modified: Self::current_timestamp(),
        }
    }

    /// Add a new rule to the config
    pub fn add_rule(&mut self, rule: ReplacementRule) {
        self.rules.push(rule);
        self.last_modified = Self::current_timestamp();
    }

    /// Update an existing rule by ID
    pub fn update_rule(&mut self, id: &str, updated_rule: ReplacementRule) -> Result<(), String> {
        match self.rules.iter_mut().find(|r| r.id == id) {
            Some(rule) => {
                *rule = updated_rule;
                self.last_modified = Self::current_timestamp();
                Ok(())
            }
            None => Err(format!("Rule with id {} not found", id)),
        }
    }

    /// Remove a rule by ID
    pub fn remove_rule(&mut self, id: &str) -> Result<(), String> {
        let initial_len = self.rules.len();
        self.rules.retain(|r| r.id != id);

        if self.rules.len() < initial_len {
            self.last_modified = Self::current_timestamp();
            Ok(())
        } else {
            Err(format!("Rule with id {} not found", id))
        }
    }

    /// Get all enabled rules (for processing)
    pub fn enabled_rules(&self) -> Vec<&ReplacementRule> {
        self.rules.iter().filter(|r| r.enabled).collect()
    }

    fn current_timestamp() -> u64 {
        std::time::SystemTime::now()
            .duration_since(std::time::UNIX_EPOCH)
            .unwrap()
            .as_secs()
    }
}

impl Default for ReplacementConfig {
    fn default() -> Self {
        Self::new()
    }
}
```

**TypeScript Definition**:
```typescript
export const ReplacementConfigSchema = z.object({
  version: z.string().regex(/^\d+\.\d+\.\d+$/), // Semver format
  rules: z.array(ReplacementRuleSchema),
  lastModified: z.number().int().positive(),
});

export type ReplacementConfig = z.infer<typeof ReplacementConfigSchema>;
```

## Storage Format

### Tauri Store Key

Stored in tauri-plugin-store under key: `"text_replacements"`

**Example JSON**:
```json
{
  "version": "1.0.0",
  "rules": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "spoken_phrase": "email signature",
      "written_text": "Best regards,\nJohn Smith\njohn@example.com",
      "case_sensitive": false,
      "enabled": true,
      "created_at": 1696550400,
      "last_used_at": 1696636800,
      "usage_count": 42
    },
    {
      "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
      "spoken_phrase": "sequel",
      "written_text": "SQL",
      "case_sensitive": false,
      "enabled": true,
      "created_at": 1696550400,
      "last_used_at": null,
      "usage_count": 0
    }
  ],
  "last_modified": 1696636800
}
```

### Import/Export Format

Same JSON structure as storage format. Files use `.json` extension.

**File naming convention**: `handy-text-replacements-YYYY-MM-DD.json`

**Validation on import**:
- Check version compatibility (currently accepts "1.0.0")
- Validate all rules against schema
- Generate new UUIDs if IDs conflict with existing rules
- Preserve usage statistics from imported file

## State Transitions

### ReplacementRule Lifecycle

```
[Created] → [Active] → [Used] → [Disabled] → [Deleted]
    ↓          ↓          ↓           ↓
 enabled=T  enabled=T  enabled=T  enabled=F
```

**States**:
1. **Created**: New rule added, `usage_count=0`, `last_used_at=None`
2. **Active**: Rule is `enabled=true`, can be matched
3. **Used**: Rule has been applied, `usage_count>0`, `last_used_at` set
4. **Disabled**: Rule set to `enabled=false`, not applied but retained in config
5. **Deleted**: Rule removed from config entirely

**Transitions**:
- Created → Active: Automatically on creation (default `enabled=true`)
- Active → Used: When rule matches during transcription
- Active ↔ Disabled: User toggles `enabled` flag in UI
- Any state → Deleted: User deletes rule (irreversible without import)

### ReplacementConfig Lifecycle

```
[Empty] → [Populated] → [Modified] → [Exported/Imported]
```

**Operations**:
- Add rule: Append to `rules` array, update `last_modified`
- Update rule: Modify rule in place, update `last_modified`
- Delete rule: Remove from array, update `last_modified`
- Export: Serialize entire config to JSON file
- Import: Merge imported rules with existing (with conflict resolution)

## Validation Rules

### ReplacementRule Validation

1. **spoken_phrase**:
   - Must not be empty or whitespace-only
   - Trimmed before storage
   - Maximum 500 characters
   - Should not contain leading/trailing whitespace after trim

2. **written_text**:
   - May be empty (for phrase deletion)
   - Maximum 5000 characters
   - Preserves newlines, tabs, special characters

3. **id**:
   - Must be valid UUID v4 format
   - Must be unique within config

4. **Timestamps**:
   - `created_at` must be reasonable (not in future, not before 2020)
   - `last_used_at` must be ≥ `created_at` if set

5. **usage_count**:
   - Must be non-negative
   - Should align with `last_used_at` (if count>0, timestamp should be set)

### ReplacementConfig Validation

1. **version**:
   - Must follow semantic versioning (MAJOR.MINOR.PATCH)
   - Currently only "1.0.0" supported
   - Future versions may require migration logic

2. **rules**:
   - Array may be empty (valid initial state)
   - No duplicate IDs allowed
   - All rules must pass individual validation

3. **Size limits**:
   - Maximum 1000 rules per config (soft limit, warn user)
   - Maximum 500KB total config size (hard limit)

## Indexing & Performance

### In-Memory Structure

The `TextReplacementsManager` maintains:
1. `rules: Vec<ReplacementRule>` - All rules from config
2. `automaton: Option<AhoCorasick>` - Pre-built pattern matcher
3. `dirty: bool` - Flag to track when automaton needs rebuild

**Rebuild triggers**:
- Add rule
- Update rule (if `spoken_phrase` or `case_sensitive` changed)
- Delete rule
- Toggle rule enabled status

**Optimization**:
- Automaton built lazily on first use after change
- Only enabled rules included in automaton
- Case-insensitive automaton built by default, case-sensitive rules handled separately

### Matching Algorithm

See research.md for detailed algorithm. Summary:

1. Run Aho-Corasick on input text → get all matches
2. Sort matches by position, then length (descending)
3. Apply non-overlapping matches (greedy longest-first)
4. Update rule statistics (mark_used) for applied rules
5. Return modified text

## Migration Strategy

### Version 1.0.0 → Future Versions

When introducing breaking changes to schema:

1. Detect old version in loaded config
2. Run migration function: `migrate_1_0_to_2_0(config) -> Result<Config>`
3. Update version field in config
4. Save migrated config back to store

**Example migration scenarios**:
- Adding new required fields: Provide defaults
- Removing fields: Drop during deserialization
- Changing field types: Convert values with validation

### Backward Compatibility

Version 1.0.0 aims for stability. Future additions should use optional fields when possible to avoid forced migrations.

## Related Documents

- **Feature Spec**: [spec.md](./spec.md) - Functional requirements
- **Research**: [research.md](./research.md) - Algorithm and pattern decisions
- **Implementation Plan**: [plan.md](./plan.md) - Overall implementation strategy
