# Quickstart: Multi-Word Text Replacements System

**Feature**: Multi-Word Text Replacements System
**Date**: 2025-10-06
**Status**: Complete

## Overview

This document provides step-by-step instructions for testing the multi-word text replacements feature. It maps directly to the acceptance scenarios in the feature spec and serves as both user documentation and integration test scenarios.

## Prerequisites

- Handy application installed and running
- Microphone access granted
- At least one Whisper model downloaded
- Settings window accessible

## Test Scenarios

### Scenario 1: Define and Use a Basic Replacement

**Given**: I have defined a replacement rule ("email signature" → full signature block)
**When**: I dictate "please add my email signature"
**Then**: The transcription contains the full signature block text

**Steps**:

1. **Open Settings**
   - Launch Handy
   - Click system tray icon → "Settings" (or use keyboard shortcut)

2. **Navigate to Text Replacements**
   - Click "Text Replacements" tab in settings window
   - Verify empty state shows "No replacement rules yet"

3. **Add Replacement Rule**
   - Click "Add Rule" button
   - Fill in form:
     - Spoken Phrase: `email signature`
     - Written Text:
       ```
       Best regards,
       John Smith
       john@example.com
       ```
     - Case Sensitive: `false` (unchecked)
   - Click "Save"
   - Verify rule appears in table with:
     - Spoken phrase: "email signature"
     - Preview: "Best regards,..." (truncated)
     - Enabled toggle: ON
     - Usage count: 0

4. **Test Transcription**
   - Focus any text editor (TextEdit, Notes, VS Code, etc.)
   - Hold global shortcut key (default: Cmd+Shift+A)
   - Speak: "please add my email signature"
   - Release shortcut
   - Wait for transcription (~1-2 seconds)

5. **Verify Result**
   - Expected text in editor:
     ```
     please add my Best regards,
     John Smith
     john@example.com
     ```
   - Open Settings → Text Replacements
   - Verify rule usage count increased to 1
   - Verify last used timestamp is recent

**Success Criteria**:
- ✅ Rule successfully created
- ✅ Phrase "email signature" replaced with full signature block
- ✅ Usage statistics updated

---

### Scenario 2: Technical Term Replacement

**Given**: I have defined "sequel" → "SQL"
**When**: I dictate "we use sequel database"
**Then**: The transcription reads "we use SQL database"

**Steps**:

1. **Add Technical Term Rule**
   - Settings → Text Replacements → Add Rule
   - Spoken Phrase: `sequel`
   - Written Text: `SQL`
   - Case Sensitive: `false`
   - Save

2. **Test Transcription**
   - Focus text editor
   - Hold shortcut, speak: "we use sequel database"
   - Release shortcut

3. **Verify Result**
   - Expected: `we use SQL database`
   - Verify "sequel" was replaced with "SQL"

**Success Criteria**:
- ✅ Single-word replacement works
- ✅ Replacement preserves surrounding context

---

### Scenario 3: Multiple Replacements in One Transcription

**Given**: I have defined multiple replacement rules
**When**: I dictate text containing several phrases that match rules
**Then**: All matching phrases are replaced correctly in the order they appear

**Steps**:

1. **Add Multiple Rules**
   - Add rule: "a p i" → "API"
   - Add rule: "this slash that" → "this/that"
   - Add rule: "dot com" → ".com"

2. **Test Transcription**
   - Focus text editor
   - Hold shortcut, speak: "the a p i endpoint is this slash that dot com"
   - Release shortcut

3. **Verify Result**
   - Expected: `the API endpoint is this/that.com`
   - Verify all three phrases replaced correctly

**Success Criteria**:
- ✅ Multiple rules apply in single transcription
- ✅ Replacements don't interfere with each other
- ✅ All usage counts incremented

---

### Scenario 4: Import Community Macro Set

**Given**: I have imported a community macro set
**When**: I dictate text using those macros
**Then**: The replacements work exactly as if I had defined them manually

**Steps**:

1. **Create Sample Import File**
   - Create file: `handy-coding-macros.json`
   - Content:
     ```json
     {
       "version": "1.0.0",
       "rules": [
         {
           "id": "00000000-0000-0000-0000-000000000001",
           "spoken_phrase": "console log",
           "written_text": "console.log()",
           "case_sensitive": false,
           "enabled": true,
           "created_at": 1696550400,
           "last_used_at": null,
           "usage_count": 0
         },
         {
           "id": "00000000-0000-0000-0000-000000000002",
           "spoken_phrase": "function definition",
           "written_text": "function example() {\n  // TODO\n}",
           "case_sensitive": false,
           "enabled": true,
           "created_at": 1696550400,
           "last_used_at": null,
           "usage_count": 0
         }
       ],
       "last_modified": 1696550400
     }
     ```

2. **Import Rules**
   - Settings → Text Replacements → Import button
   - Select file `handy-coding-macros.json`
   - Choose merge strategy: "Skip duplicates"
   - Click "Import"
   - Verify success message: "Imported 2 rules (0 skipped)"

3. **Test Imported Rules**
   - Focus VS Code or text editor
   - Hold shortcut, speak: "add console log here"
   - Release shortcut
   - Expected: `add console.log() here`

4. **Test Multi-Line Macro**
   - Hold shortcut, speak: "create a function definition"
   - Release shortcut
   - Expected:
     ```
     create a function example() {
       // TODO
     }
     ```

**Success Criteria**:
- ✅ Import succeeds without errors
- ✅ Imported rules appear in list
- ✅ Imported rules work in transcription
- ✅ Multi-line replacements preserve formatting

---

### Scenario 5: Coexistence with Custom Words (Single-Word Fuzzy Matching)

**Given**: I have both single-word custom_words and multi-word replacement rules
**When**: I dictate text matching both types
**Then**: Both systems apply their replacements without conflict

**Steps**:

1. **Set Up Both Systems**
   - Settings → Custom Words → Add: `handy` (existing fuzzy matching feature)
   - Settings → Text Replacements → Add rule: "thank you" → "thanks"

2. **Test Transcription with Both**
   - Focus text editor
   - Hold shortcut, speak: "thank you for handy"
   - Release shortcut

3. **Verify Result**
   - Expected: `thanks for handy`
   - Text replacements applied first: "thank you" → "thanks"
   - Custom words applied second: "handy" → "Handy" (capitalized via custom_words logic)

**Success Criteria**:
- ✅ Text replacements execute before custom_words
- ✅ Both systems work without conflict
- ✅ Pipeline order is deterministic

---

### Scenario 6: Case-Sensitive Replacement

**Given**: I define a case-sensitive replacement ("api" → "API")
**When**: I dictate "the api endpoint"
**Then**: The transcription reads "the API endpoint" with correct capitalization

**Steps**:

1. **Add Case-Sensitive Rule**
   - Settings → Text Replacements → Add Rule
   - Spoken Phrase: `api`
   - Written Text: `API`
   - Case Sensitive: `true` (checked)
   - Save

2. **Test Lowercase Match**
   - Focus text editor
   - Hold shortcut, speak: "the api endpoint"
   - Release shortcut
   - Expected: `the API endpoint`

3. **Test Uppercase Non-Match**
   - Hold shortcut, speak: "the API endpoint" (if Whisper transcribes "API")
   - Release shortcut
   - Expected: `the API endpoint` (no replacement, already uppercase)

**Success Criteria**:
- ✅ Case-sensitive flag respected
- ✅ Only exact case matches replaced

---

### Scenario 7: Empty Replacement (Phrase Deletion)

**Given**: I define a replacement with empty "written text"
**When**: I dictate text containing that phrase
**Then**: The phrase is deleted from transcription

**Steps**:

1. **Add Deletion Rule**
   - Settings → Text Replacements → Add Rule
   - Spoken Phrase: `um`
   - Written Text: `` (empty)
   - Save

2. **Test Deletion**
   - Focus text editor
   - Hold shortcut, speak: "I think um we should"
   - Release shortcut
   - Expected: `I think  we should` (note: double space where "um" was)

**Success Criteria**:
- ✅ Phrase successfully deleted
- ✅ Surrounding text preserved

**Note**: Double-space handling is a future enhancement. Current version preserves spacing exactly.

---

### Scenario 8: Disable Rule Temporarily

**Given**: I have a rule enabled
**When**: I disable it via toggle
**Then**: The rule no longer applies but remains in my configuration

**Steps**:

1. **Add Rule**
   - Settings → Text Replacements → Add Rule
   - Spoken Phrase: `dot org`
   - Written Text: `.org`
   - Save

2. **Test Enabled**
   - Hold shortcut, speak: "visit example dot org"
   - Expected: `visit example.org`

3. **Disable Rule**
   - Settings → Text Replacements
   - Find "dot org" rule
   - Click toggle to OFF
   - Verify toggle shows "Disabled"

4. **Test Disabled**
   - Hold shortcut, speak: "visit example dot org"
   - Expected: `visit example dot org` (no replacement)

5. **Re-enable and Verify**
   - Toggle back to ON
   - Hold shortcut, speak: "visit example dot org"
   - Expected: `visit example.org` (replacement works again)

**Success Criteria**:
- ✅ Toggle disables rule without deletion
- ✅ Disabled rule not applied
- ✅ Re-enabling restores functionality

---

### Scenario 9: Export and Import Round-Trip

**Given**: I have multiple rules defined
**When**: I export them, clear config, then import
**Then**: All rules restored with metadata intact

**Steps**:

1. **Create Test Rules**
   - Add 3 rules with different phrases
   - Use some rules to generate usage statistics

2. **Export Rules**
   - Settings → Text Replacements → Export button
   - Choose location: `~/Downloads/handy-backup.json`
   - Verify file created

3. **Clear Config**
   - Delete all rules manually (or use dev tools to clear store)
   - Verify empty state

4. **Import Backup**
   - Settings → Text Replacements → Import button
   - Select `~/Downloads/handy-backup.json`
   - Merge strategy: "Replace all"
   - Import

5. **Verify Restoration**
   - Check all 3 rules present
   - Verify usage statistics preserved
   - Test one rule to ensure it works

**Success Criteria**:
- ✅ Export creates valid JSON file
- ✅ Import restores all rules
- ✅ Metadata (usage counts, timestamps) preserved
- ✅ Imported rules function correctly

---

### Scenario 10: Performance with Many Rules

**Given**: I have 100+ replacement rules
**When**: I transcribe a 500-word document
**Then**: Processing completes in <100ms (acceptable performance)

**Steps**:

1. **Generate Large Rule Set**
   - Use script or manually create 100 rules (see `scripts/generate-test-rules.sh`)
   - Import via JSON file

2. **Prepare Long Transcription**
   - Find or create 500-word speech sample
   - Focus text editor

3. **Measure Performance**
   - Hold shortcut, speak 500-word text
   - Release shortcut
   - Note transcription completion time

4. **Check Logs**
   - Open developer console (Cmd+Option+I)
   - Look for log: `"Text replacement processing time: Xms"`
   - Verify X < 100ms

**Success Criteria**:
- ✅ 100 rules loaded without errors
- ✅ Transcription completes without noticeable delay
- ✅ Processing time <100ms (logged in console)

**Note**: Performance test requires dev build with timing logs enabled.

---

## Edge Cases & Error Scenarios

### Edge Case 1: Overlapping Phrases (Longest Match Wins)

**Setup**:
- Add rule: "hello world" → "HELLO_WORLD"
- Add rule: "world peace" → "WORLD_PEACE"

**Test**:
- Speak: "hello world peace"
- Expected: `HELLO_WORLD peace` (longer match "hello world" takes precedence)

**Success**: ✅ Greedy longest-match algorithm working

---

### Edge Case 2: Phrase at Start/End of Transcription

**Setup**:
- Add rule: "hello there" → "greetings"

**Test**:
- Speak: "hello there" (phrase at start)
- Expected: `greetings`
- Speak: "say hello there" (phrase at end)
- Expected: `say greetings`

**Success**: ✅ Position-independent matching

---

### Edge Case 3: Replacement Contains Trigger Phrase (Non-Recursive)

**Setup**:
- Add rule: "short" → "short version"

**Test**:
- Speak: "use the short form"
- Expected: `use the short version form` (not "short version version version...")

**Success**: ✅ Non-recursive replacement working

---

### Error Case 1: Invalid Import File

**Test**:
- Create file with invalid JSON
- Attempt import
- Expected: Error message "Invalid JSON format: {details}"

**Success**: ✅ Graceful error handling

---

### Error Case 2: Exceeding Character Limits

**Test**:
- Try to create rule with 501-character spoken phrase
- Expected: Error message "Spoken phrase exceeds 500 character limit"

**Success**: ✅ Validation prevents invalid rules

---

## User Workflows

### Workflow 1: Email Power User

**Persona**: Writes 50+ emails per day, uses signatures, common phrases

**Setup**:
1. Add rule: "email signature" → full signature
2. Add rule: "meeting link" → Zoom URL
3. Add rule: "team alias" → team@company.com

**Daily Usage**:
- Dictate emails with voice
- Use macros naturally in speech
- Save 10+ minutes per day on repetitive typing

---

### Workflow 2: Developer Using Voice Coding

**Persona**: Developer with RSI, uses voice for code comments and documentation

**Setup**:
1. Import community "Code Macros" pack
2. Add personal rules: "my name" → "John Smith", "project name" → "Handy"

**Daily Usage**:
- Dictate code comments
- Use macros for boilerplate (console.log, function templates)
- Reduce typing strain significantly

---

### Workflow 3: Writer with Technical Content

**Persona**: Technical writer documenting APIs and systems

**Setup**:
1. Add domain-specific acronyms: "a p i" → "API", "s d k" → "SDK"
2. Add product names: "our product" → "SuperApp Pro"

**Daily Usage**:
- Dictate documentation naturally
- Technical terms auto-formatted correctly
- Maintains consistent terminology across documents

---

## Integration Test Automation

The scenarios above should be automated as integration tests:

**Test file**: `/src-tauri/tests/integration/text_replacements_quickstart.rs`

**Structure**:
```rust
#[test]
fn scenario_1_basic_replacement() {
    // Arrange: Add rule
    // Act: Simulate transcription
    // Assert: Output contains replacement
}

// ... one test per scenario
```

**Frontend E2E tests**: Use Playwright or similar for UI workflows

---

## Troubleshooting

**Issue**: Replacement not applying

**Checks**:
1. Rule enabled? (check toggle in settings)
2. Spoken phrase exact match? (check Whisper transcription output)
3. Rules loaded? (check console for errors)

**Issue**: Performance slow with many rules

**Checks**:
1. How many rules? (>500 may degrade performance)
2. Check logs for processing time
3. Consider disabling rarely-used rules

**Issue**: Import fails

**Checks**:
1. Valid JSON format?
2. Version compatible? (must be "1.0.0")
3. File readable? (check permissions)

---

## Related Documents

- **Feature Spec**: [spec.md](./spec.md) - Requirements and acceptance criteria
- **Contracts**: [contracts/tauri-commands.md](./contracts/tauri-commands.md) - API definitions
- **Data Model**: [data-model.md](./data-model.md) - Entity structures
