# Feature Specification: Multi-Word Text Replacements System

**Feature Branch**: `003-multi-word-text`
**Created**: 2025-10-06
**Status**: Draft
**Input**: User description: "Multi word text replacements"

## Execution Flow (main)
```
1. Parse user description from Input
   → User needs multi-word/phrase replacement capability
2. Extract key concepts from description
   → Actors: Users creating transcriptions
   → Actions: Define replacement rules, apply during transcription
   → Data: Replacement rules (phrase pairs), transcription text
   → Constraints: Must not interfere with existing custom_words system
3. For each unclear aspect:
   → None identified - feature scope is clear
4. Fill User Scenarios & Testing section
   → Primary: User defines phrase replacements, system applies them
5. Generate Functional Requirements
   → All requirements testable and unambiguous
6. Identify Key Entities
   → Replacement Rule, Replacement Config
7. Run Review Checklist
   → No implementation details, focused on user value
8. Return: SUCCESS (spec ready for planning)
```

---

## ⚡ Quick Guidelines

This feature extends Handy's existing `custom_words` system (single-word fuzzy matching) with **multi-word phrase replacements**. Users can define exact phrase mappings that transform dictated speech into formatted text.

**Current Limitation:** The existing `custom_words` feature only replaces one word at a time using fuzzy phonetic matching. Users cannot replace phrases or create text macros.

**This Feature Solves:** Enable users to define multi-word phrase replacements (e.g., "email signature" → full signature block, "this slash that" → "this/that") for context-aware text transformation.

---

## User Scenarios & Testing *(mandatory)*

### Primary User Story

As a **Handy user**, I want to define custom phrase replacements so that common multi-word expressions, technical terms, and personal macros are automatically transformed into my preferred text format during transcription, saving me time and reducing post-transcription editing.

**Example Use Cases:**
- Email writer: "email signature" → "Best regards,\nJohn Smith\njohn@example.com"
- Developer: "sequel database" → "SQL database"
- Technical writer: "a p i" → "API" (handles model spacing issues)
- General user: "this slash that" → "this/that"

### Acceptance Scenarios

1. **Given** I have defined a replacement rule ("email signature" → full signature block), **When** I dictate "please add my email signature", **Then** the transcription contains the full signature block text

2. **Given** I have defined "sequel" → "SQL", **When** I dictate "we use sequel database", **Then** the transcription reads "we use SQL database"

3. **Given** I have defined multiple replacement rules, **When** I dictate text containing several phrases that match rules, **Then** all matching phrases are replaced correctly in the order they appear

4. **Given** I have imported a community macro set, **When** I dictate text using those macros, **Then** the replacements work exactly as if I had defined them manually

5. **Given** I have both single-word custom_words and multi-word replacement rules, **When** I dictate text matching both types, **Then** both systems apply their replacements without conflict

6. **Given** I define a case-sensitive replacement ("api" → "API"), **When** I dictate "the api endpoint", **Then** the transcription reads "the API endpoint" with correct capitalization

### Edge Cases

- What happens when replacement phrases overlap (e.g., "hello world" and "world peace" both defined)?
  - **Expected**: Longest match takes precedence (greedy matching)

- What happens when a replacement phrase appears at the start or end of transcription?
  - **Expected**: Replacements work regardless of position

- What happens when a replacement phrase is part of a longer word?
  - **Expected**: Only replace complete phrase matches (word boundaries respected)

- What happens when the replacement text itself contains a phrase that matches another rule?
  - **Expected**: Replacements apply only to original transcription, not to replacement results (no recursive replacement)

- What happens when I have 100+ replacement rules?
  - **Expected**: Performance remains acceptable (<100ms additional processing time)

- What happens if I define a replacement with an empty "replace with" value?
  - **Expected**: System deletes the phrase (useful for removing filler words)

---

## Requirements *(mandatory)*

### Functional Requirements

**Replacement Rule Management:**
- **FR-001**: Users MUST be able to define replacement rules as phrase pairs (spoken phrase → written text)
- **FR-002**: Users MUST be able to add, edit, and delete replacement rules
- **FR-003**: System MUST support multi-word phrases in both "spoken" and "written" parts (no word count limit)
- **FR-004**: System MUST allow empty "written" text (for phrase deletion)
- **FR-005**: System MUST persist replacement rules across application restarts

**Replacement Application:**
- **FR-006**: System MUST apply replacement rules after transcription completes but before custom_words fuzzy matching
- **FR-007**: System MUST perform exact phrase matching (not fuzzy/phonetic matching)
- **FR-008**: System MUST respect word boundaries (only replace complete phrase matches)
- **FR-009**: System MUST apply longest matching phrase when overlaps exist (greedy matching)
- **FR-010**: System MUST NOT recursively apply replacements to replacement results
- **FR-011**: System MUST support case-sensitive and case-insensitive replacement options per rule

**Configuration Storage:**
- **FR-012**: Replacement rules MUST be stored in a human-readable format (JSON or YAML)
- **FR-013**: Users MUST be able to export their replacement rules to a file
- **FR-014**: Users MUST be able to import replacement rules from a file
- **FR-015**: Imported rules MUST merge with existing rules (with conflict resolution options)

**Performance & Quality:**
- **FR-016**: Replacement processing MUST complete in <100ms for typical transcriptions (<500 words)
- **FR-017**: System MUST support at least 500 replacement rules without performance degradation
- **FR-018**: System MUST handle special characters in replacement text (newlines, tabs, quotes, unicode)

**Integration:**
- **FR-019**: System MUST coexist with existing custom_words fuzzy matching (no conflicts)
- **FR-020**: Replacement rules MUST be accessible to future features (voice commands, context profiles)

### Key Entities

**ReplacementRule**
- Spoken phrase (what user dictates) - string, multi-word
- Written text (what appears in transcription) - string, may contain special chars
- Case sensitivity flag (true = exact case match, false = ignore case) - boolean, default false
- Enabled flag (allows temporary disabling without deletion) - boolean, default true
- Creation date (for sorting/management) - timestamp
- Last used date (for analytics) - timestamp
- Usage count (how many times applied) - integer

**ReplacementConfig**
- List of replacement rules - ordered collection
- Config version (for migration) - semver string
- Last modified date - timestamp
- Import/export format version - semver string

---

## Review & Acceptance Checklist

### Content Quality
- [x] No implementation details (languages, frameworks, APIs)
- [x] Focused on user value and business needs
- [x] Written for non-technical stakeholders
- [x] All mandatory sections completed

### Requirement Completeness
- [x] No [NEEDS CLARIFICATION] markers remain
- [x] Requirements are testable and unambiguous
- [x] Success criteria are measurable (performance targets, feature counts)
- [x] Scope is clearly bounded (phrase replacement only, no complex logic)
- [x] Dependencies and assumptions identified (builds on existing custom_words)

---

## Execution Status

- [x] User description parsed
- [x] Key concepts extracted (phrase pairs, macro expansion, exact matching)
- [x] Ambiguities marked (none)
- [x] User scenarios defined (email signatures, technical terms, common phrases)
- [x] Requirements generated (20 functional requirements)
- [x] Entities identified (ReplacementRule, ReplacementConfig)
- [x] Review checklist passed

---

## Success Criteria

**Phase 1 (Foundation) - This Feature:**
- Users can define at least 50 replacement rules via settings UI
- Replacement processing adds <50ms to transcription pipeline
- Export/import works with JSON format
- At least 3 community macro sets shared within first month

**Alignment with Roadmap:**
- **Writing Assistant Alignment Score**: 8/10 (enables context-aware text transformation)
- **User Value Score**: 9/10 (universal need for macros and corrections)
- **Implementation Effort**: M (1-2 weeks to extend existing custom_words system)
- **Roadmap Phase**: Phase 1 - Foundation
- **Roadmap Status**: Priority 1 (Quick Win)

**Constitution Compliance:**
- **Principle I (Modularity)**: ✅ Independent module, can be toggled on/off
- **Principle II (Conventions)**: ✅ JSON/YAML storage, standard Rust patterns
- **Principle III (Writing Assistant)**: ✅ Enables context-aware text transformation (near-term goal)
- **Principle IV (TDD)**: ✅ Spec includes testable acceptance scenarios
- **Principle V (Clarity)**: ✅ Simple exact matching, no complex AI or fuzzy logic

---

## Notes for Implementation Planning

**Existing System Context** (from CLAUDE.md analysis):
- Current `custom_words` system: `Vec<String>` with fuzzy matching in `transcription.rs:370-379`
- Replacement should hook into same pipeline location
- Settings stored using `tauri-plugin-store` (JSON persistence)

**Suggested Technical Approach** (for /plan phase):
- Extend settings schema with `text_replacements: HashMap<String, String>` or structured array
- Add `apply_text_replacements()` function before `apply_custom_words()` in transcription pipeline
- Use case-insensitive string matching for default behavior, flag for case-sensitive
- Implement longest-match-first algorithm to handle overlapping phrases

**Integration Points:**
- Settings UI: Add "Text Replacements" section with add/edit/delete/import/export
- Transcription pipeline: Insert after Whisper output, before custom_words
- Storage: Extend existing Tauri store with replacement rules

**Future Extensions** (out of scope for this spec):
- Voice commands that trigger replacements (Phase 1, different feature)
- Context-aware replacement profiles (Phase 3)
- Regex support for advanced users (Phase 5)
- OCR learning that auto-generates rules (Phase 5)

---

**Status: READY FOR PLANNING**

This feature is the foundation for all context-aware writing assistance in Handy. Once implemented, it enables voice commands, macros, and eventually context-aware profiles.

**Next Command:** `/plan` to create implementation plan with technical design.
