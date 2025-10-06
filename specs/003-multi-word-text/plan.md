
# Implementation Plan: Multi-Word Text Replacements System

**Branch**: `003-multi-word-text` | **Date**: 2025-10-06 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/root/Repos/Handy/specs/003-multi-word-text/spec.md`

## Execution Flow (/plan command scope)
```
1. Load feature spec from Input path
   → If not found: ERROR "No feature spec at {path}"
2. Fill Technical Context (scan for NEEDS CLARIFICATION)
   → Detect Project Type from file system structure or context (web=frontend+backend, mobile=app+api)
   → Set Structure Decision based on project type
3. Fill the Constitution Check section based on the content of the constitution document.
4. Evaluate Constitution Check section below
   → If violations exist: Document in Complexity Tracking
   → If no justification possible: ERROR "Simplify approach first"
   → Update Progress Tracking: Initial Constitution Check
5. Execute Phase 0 → research.md
   → If NEEDS CLARIFICATION remain: ERROR "Resolve unknowns"
6. Execute Phase 1 → contracts, data-model.md, quickstart.md, agent-specific template file (e.g., `CLAUDE.md` for Claude Code, `.github/copilot-instructions.md` for GitHub Copilot, `GEMINI.md` for Gemini CLI, `QWEN.md` for Qwen Code, or `AGENTS.md` for all other agents).
7. Re-evaluate Constitution Check section
   → If new violations: Refactor design, return to Phase 1
   → Update Progress Tracking: Post-Design Constitution Check
8. Plan Phase 2 → Describe task generation approach (DO NOT create tasks.md)
9. STOP - Ready for /tasks command
```

**IMPORTANT**: The /plan command STOPS at step 7. Phases 2-4 are executed by other commands:
- Phase 2: /tasks command creates tasks.md
- Phase 3-4: Implementation execution (manual or via tools)

## Summary

This feature extends Handy's existing single-word `custom_words` system with multi-word phrase replacement capabilities. Users can define exact phrase mappings (e.g., "email signature" → full signature block, "sequel database" → "SQL database") that transform dictated speech into formatted text. The system performs exact phrase matching with longest-match-first algorithm, supports case-sensitive/insensitive options, and integrates into the existing transcription pipeline before custom_words fuzzy matching. Replacement rules are stored in JSON format with import/export capabilities for sharing macro sets.

## Technical Context
**Language/Version**: Rust (latest stable, ~1.75+) for backend, TypeScript/React for frontend
**Primary Dependencies**:
- Backend: `tauri`, `tauri-plugin-store` (settings), `serde` (JSON serialization), `regex` (phrase matching)
- Frontend: React, Zod (schemas), Tauri API bindings
**Storage**: JSON via tauri-plugin-store (extends existing settings store)
**Testing**: `cargo test` for Rust, Vitest for TypeScript/React
**Target Platform**: Cross-platform desktop (macOS, Windows, Linux) via Tauri
**Project Type**: Single Tauri desktop application (Rust backend + React frontend)
**Performance Goals**: <100ms replacement processing for typical transcriptions (<500 words), support 500+ rules without degradation
**Constraints**: Exact phrase matching only (no fuzzy/regex), non-recursive replacement, must coexist with existing custom_words system
**Scale/Scope**: 20 functional requirements, ~500 rules per user, settings UI extension, transcription pipeline integration

## Constitution Check
*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

**Principle I - Modularity First**: ✅ PASS
- Feature will be implemented as independent module in transcription pipeline
- No circular dependencies with existing managers
- Can be enabled/disabled via settings without affecting other features
- Clear interface: takes transcription text, returns modified text

**Principle II - Convention Over Configuration**: ✅ PASS
- Follows Rust conventions: snake_case functions, `anyhow::Error`, `Arc<Mutex<T>>` for shared state
- Follows TypeScript conventions: Zod schemas, functional components, proper destructuring
- Uses existing patterns from custom_words system
- Standard JSON storage via tauri-plugin-store

**Principle III - Writing Assistant Transformation**: ✅ PASS (Strong alignment)
- Enables context-aware text transformation (email signatures, technical terms)
- Foundation for future writing assistant features (voice commands, context profiles)
- Aligns with near-term goal: smart formatting and command mode
- Priority 1 "Quick Win" in roadmap Phase 1

**Principle IV - Test-Driven Development**: ✅ PASS
- Spec includes 6 acceptance scenarios with Given/When/Then format
- Plan will generate contract tests first (Tauri commands)
- Integration tests for user workflows
- Unit tests for replacement algorithm

**Principle V - Code Clarity Over Cleverness**: ✅ PASS
- Exact phrase matching (simple, predictable)
- No complex regex or fuzzy logic
- Longest-match-first algorithm is straightforward
- Clear data model (ReplacementRule, ReplacementConfig)

**Initial Assessment**: All principles satisfied. No complexity deviations required.

## Project Structure

### Documentation (this feature)
```
specs/[###-feature]/
├── plan.md              # This file (/plan command output)
├── research.md          # Phase 0 output (/plan command)
├── data-model.md        # Phase 1 output (/plan command)
├── quickstart.md        # Phase 1 output (/plan command)
├── contracts/           # Phase 1 output (/plan command)
└── tasks.md             # Phase 2 output (/tasks command - NOT created by /plan)
```

### Source Code (repository root)
```
src-tauri/                          # Rust backend
├── src/
│   ├── managers/
│   │   ├── transcription.rs       # [MODIFY] Add text replacement logic
│   │   └── text_replacements.rs   # [NEW] Replacement manager
│   ├── commands/
│   │   └── text_replacements.rs   # [NEW] Tauri commands for CRUD operations
│   ├── settings.rs                # [MODIFY] Add replacement rules schema
│   └── lib.rs                     # [MODIFY] Register new manager + commands
└── tests/
    ├── contract/
    │   └── text_replacements.rs   # [NEW] Contract tests for commands
    └── integration/
        └── text_replacements.rs   # [NEW] Integration tests for workflows

src/                                # React/TypeScript frontend
├── components/
│   └── settings/
│       └── TextReplacements.tsx   # [NEW] UI for managing replacement rules
├── hooks/
│   └── useTextReplacements.ts     # [NEW] Hook for replacement CRUD
└── lib/
    └── types.ts                   # [MODIFY] Add ReplacementRule types
```

**Structure Decision**: Single Tauri desktop application. Backend logic in `src-tauri/src/managers/`, frontend UI in `src/components/settings/`. Follows existing pattern from custom_words system (single-word replacements in transcription.rs:370-379). New text_replacements manager handles multi-word logic, integrates into transcription pipeline before custom_words.

## Phase 0: Outline & Research
1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Generate and dispatch research agents**:
   ```
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

## Phase 1: Design & Contracts
*Prerequisites: research.md complete*

1. **Extract entities from feature spec** → `data-model.md`:
   - Entity name, fields, relationships
   - Validation rules from requirements
   - State transitions if applicable

2. **Generate API contracts** from functional requirements:
   - For each user action → endpoint
   - Use standard REST/GraphQL patterns
   - Output OpenAPI/GraphQL schema to `/contracts/`

3. **Generate contract tests** from contracts:
   - One test file per endpoint
   - Assert request/response schemas
   - Tests must fail (no implementation yet)

4. **Extract test scenarios** from user stories:
   - Each story → integration test scenario
   - Quickstart test = story validation steps

5. **Update agent file incrementally** (O(1) operation):
   - Run `.specify/scripts/bash/update-agent-context.sh claude`
     **IMPORTANT**: Execute it exactly as specified above. Do not add or remove any arguments.
   - If exists: Add only NEW tech from current plan
   - Preserve manual additions between markers
   - Update recent changes (keep last 3)
   - Keep under 150 lines for token efficiency
   - Output to repository root

**Output**: data-model.md, /contracts/*, failing tests, quickstart.md, agent-specific file

## Phase 2: Task Planning Approach
*This section describes what the /tasks command will do - DO NOT execute during /plan*

**Task Generation Strategy**:

The `/tasks` command will load `.specify/templates/tasks-template.md` and generate ordered tasks from Phase 1 design artifacts:

1. **From contracts/tauri-commands.md**:
   - Task per command contract test (verify request/response schemas)
   - 8 contract test tasks (one per Tauri command)

2. **From data-model.md**:
   - Task for ReplacementRule struct + impl
   - Task for ReplacementConfig struct + impl
   - Task for TextReplacementsManager struct scaffold
   - 3 model creation tasks

3. **From quickstart.md scenarios**:
   - Task per acceptance scenario (10 integration tests)
   - Each maps to scenario 1-10 in quickstart

4. **Implementation tasks**:
   - Implement Aho-Corasick-based replacement algorithm
   - Implement Tauri commands (CRUD operations)
   - Implement import/export logic
   - Create React UI components (TextReplacements.tsx)
   - Create custom hook (useTextReplacements.ts)
   - Integrate into transcription pipeline
   - Add settings UI tab

**Ordering Strategy**:

Following TDD and dependency order:

**Phase 1 - Foundation (Parallel)**:
1. [P] Define data models (ReplacementRule, ReplacementConfig)
2. [P] Create contract tests (all 8 commands)
3. [P] Create integration test stubs (all 10 scenarios)

**Phase 2 - Backend Core (Sequential)**:
4. Implement TextReplacementsManager skeleton
5. Implement replacement algorithm with Aho-Corasick
6. Make algorithm unit tests pass
7. Implement store persistence (load/save)

**Phase 3 - Tauri Commands (Sequential)**:
8. Implement get_replacement_config command
9. Implement add_replacement_rule command
10. Implement update_replacement_rule command
11. Implement delete_replacement_rule command
12. Implement toggle_replacement_rule command
13. Implement import/export commands
14. Make all contract tests pass

**Phase 4 - Pipeline Integration (Sequential)**:
15. Integrate TextReplacementsManager into transcription pipeline
16. Make integration tests 1-3 pass (basic replacement scenarios)

**Phase 5 - Frontend (Parallel after Phase 4)**:
17. [P] Create TypeScript types (extend lib/types.ts)
18. [P] Create useTextReplacements hook
19. Create TextReplacements.tsx component
20. Add settings tab for Text Replacements
21. Implement CRUD UI operations
22. Implement import/export UI

**Phase 6 - Advanced Scenarios (Sequential)**:
23. Make integration test 4 pass (import community macros)
24. Make integration test 5 pass (coexistence with custom_words)
25. Make integration test 6 pass (case-sensitive matching)
26. Make integration tests 7-10 pass (edge cases)

**Phase 7 - Polish & Performance (Sequential)**:
27. Add usage statistics tracking
28. Create performance benchmarks (criterion.rs)
29. Optimize if needed (based on benchmark results)
30. Update CLAUDE.md with final context

**Estimated Output**: 30 tasks, numbered and ordered, with [P] markers for parallelizable tasks

**Task Categories**:
- Models: 3 tasks
- Tests: 18 tasks (8 contract + 10 integration)
- Backend: 9 tasks (manager, commands, pipeline)
- Frontend: 6 tasks (types, hook, components)
- Polish: 4 tasks (stats, benchmarks, optimization, docs)

**Critical Path**: Foundation → Backend Core → Commands → Pipeline Integration → Frontend → Advanced Scenarios

**Estimated Timeline**: 1-2 weeks (per spec success criteria)

**IMPORTANT**: This phase is executed by the /tasks command, NOT by /plan

## Phase 3+: Future Implementation
*These phases are beyond the scope of the /plan command*

**Phase 3**: Task execution (/tasks command creates tasks.md)  
**Phase 4**: Implementation (execute tasks.md following constitutional principles)  
**Phase 5**: Validation (run tests, execute quickstart.md, performance validation)

## Complexity Tracking
*Fill ONLY if Constitution Check has violations that must be justified*

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |


## Progress Tracking
*This checklist is updated during execution flow*

**Phase Status**:
- [x] Phase 0: Research complete (/plan command) - research.md created
- [x] Phase 1: Design complete (/plan command) - data-model.md, contracts/, quickstart.md, CLAUDE.md updated
- [x] Phase 2: Task planning complete (/plan command - describe approach only) - 30 tasks planned in 7 phases
- [ ] Phase 3: Tasks generated (/tasks command) - NEXT STEP: Run `/tasks` to create tasks.md
- [ ] Phase 4: Implementation complete
- [ ] Phase 5: Validation passed

**Gate Status**:
- [x] Initial Constitution Check: PASS (all 5 principles satisfied)
- [x] Post-Design Constitution Check: PASS (design confirms all principles, no violations)
- [x] All NEEDS CLARIFICATION resolved (Technical Context filled, no unknowns)
- [x] Complexity deviations documented (none required)

---
*Based on Constitution v2.1.1 - See `/memory/constitution.md`*
