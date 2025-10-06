<!--
Sync Impact Report:
Version: 0.0.0 → 1.0.0
Modified Principles: N/A (initial creation)
Added Sections:
  - Core Principles (5 principles focused on clean code, modularity, conventions, and writing assistant transformation)
  - Development Standards (code style, testing, documentation)
  - Writing Assistant Vision (strategic direction)
  - Governance (amendment process and compliance)
Removed Sections: N/A
Templates Requiring Updates:
  ✅ plan-template.md - Constitution Check section will reference these principles
  ✅ spec-template.md - Aligned with functional requirements approach
  ✅ tasks-template.md - Task categories align with principles
Follow-up TODOs: None
-->

# Handy Constitution

## Core Principles

### I. Modularity First

**Every feature must be architected as modular, independently testable components.** This applies to both Rust backend modules and React frontend components.

Backend modules in `src-tauri/src/` MUST be self-contained with clear boundaries. Each manager (audio, model, transcription, history) operates independently through well-defined interfaces. No circular dependencies between managers. State sharing only through Tauri's managed state or explicit event passing.

Frontend components in `src/` MUST follow single responsibility principle. Each component handles one concern. No God components that mix UI, business logic, and state management. Use composition over inheritance. Extract reusable logic into custom hooks.

**Rationale:** Handy is explicitly built to be forkable. Modular architecture makes it easy for others to understand, extend, and customize specific features without touching unrelated code. This is core to the project's open source mission.

### II. Convention Over Configuration

**Follow established Rust, TypeScript, and Tauri conventions strictly.** Consistency makes the codebase accessible to new contributors and reduces cognitive load.

Rust conventions are non-negotiable:
- Snake_case for functions and variables, PascalCase for types
- Use `anyhow::Error` for error handling with descriptive context via `.context()`
- Builder pattern for complex initialization chains
- `Arc<Mutex<T>>` for shared state between threads
- Result types with `?` operator for error propagation
- Structured logging with appropriate levels (debug, info, warn, error)

TypeScript conventions are mandatory:
- Use Zod schemas for runtime type validation and inference
- Functional components with explicit TypeScript interfaces
- PascalCase for components, camelCase for variables/functions
- `useCallback` for stable function references, `useMemo` for expensive computations
- Import type syntax for type-only imports: `import type { Foo } from './types'`
- Destructure props with defaults: `{ disabled = false }`

**Rationale:** Following conventions reduces friction for contributors. The codebase should feel familiar to anyone who knows Rust or TypeScript. No surprises, no custom patterns to learn.

### III. Writing Assistant Transformation

**Handy is evolving from a speech-to-text tool into a true writing assistant.** All new features must align with this vision.

Writing assistant capabilities MUST include:
- Context-aware transcription that understands writing modes (email, code, prose, messaging)
- Intelligent formatting and punctuation based on content type
- Real-time grammar and style suggestions during transcription
- Integration with writing workflows (markdown, documentation, code comments)
- Memory of user's writing patterns and preferences
- Multi-modal input support (voice + text editing)

Features that only serve "dictation" use cases should be questioned. Features that help users *write better* should be prioritized. The goal is assistance, not just transcription.

**Rationale:** The project differentiator is not just local STT, but becoming a privacy-focused writing partner. This principle ensures every decision moves toward that vision.

### IV. Test-Driven Development (TDD)

**Tests must be written before implementation. No exceptions.**

For new features following the spec workflow:
1. Contract tests written first (API surface, command interfaces)
2. Integration tests written second (feature workflows)
3. Tests MUST fail initially (proves they test something)
4. Implementation written to make tests pass
5. Refactor with tests as safety net

For bug fixes:
1. Write failing test that reproduces the bug
2. Fix the bug
3. Verify test passes

Test categories required:
- Contract tests: Verify Tauri command interfaces (input/output shapes)
- Integration tests: Verify feature workflows end-to-end
- Unit tests: Verify individual function behavior (especially complex logic)

Use appropriate test frameworks:
- Rust: `cargo test` with `#[cfg(test)]` modules
- TypeScript: Vitest for frontend unit/integration tests

**Rationale:** TDD prevents regressions, documents expected behavior, and ensures code is testable by design. Given Handy's cross-platform nature, automated tests are critical.

### V. Code Clarity Over Cleverness

**Prioritize readable, maintainable code over performance optimizations unless proven necessary.**

Code must be self-documenting:
- Use descriptive variable names (no abbreviations except standard ones like `idx`, `ctx`)
- Function names describe action and intent (`process_audio_with_vad` not `process`)
- Comment the "why" not the "what" (only when why is non-obvious)
- Separate logical sections with clear comment blocks
- Extract complex logic into named functions

Performance optimizations require:
- Proof of performance problem (benchmarks, profiling)
- Clear comments explaining the optimization
- Tests verifying correctness is maintained

Avoid:
- Nested ternaries
- Deep callback chains (use async/await)
- Magic numbers (use named constants)
- Abbreviations that sacrifice clarity

**Rationale:** Handy is built for forkability. Contributors should be able to understand code quickly. Premature optimization creates complexity that hinders the core mission.

## Development Standards

### Code Style

**Rust formatting:**
- Use `rustfmt` with project defaults (run `cargo fmt`)
- Logical sections separated by `/* ─────────── Section Name ─────────── */`
- Error context required on all `?` operations: `.context("descriptive error message")?`
- Imports grouped: std library, external crates, internal modules

**TypeScript formatting:**
- Use Prettier with project configuration
- One component per file (except tightly coupled components)
- Props interfaces defined above component
- Export component at bottom of file
- Group imports: React/libraries, components, hooks, types, utilities

### Testing Requirements

**Test coverage expectations:**
- All Tauri commands MUST have contract tests
- All user-facing features MUST have integration tests
- Complex algorithms MUST have unit tests
- Bug fixes MUST include regression tests

**Test quality standards:**
- Tests must have descriptive names explaining what they verify
- Use arrange-act-assert pattern
- One logical assertion per test (multiple checks of same outcome are okay)
- No test interdependencies (each test runs independently)
- Mock external dependencies (audio devices, file system, models)

### Documentation Standards

**Code documentation:**
- Public Rust APIs require doc comments (`///`)
- Public TypeScript APIs require JSDoc comments
- Non-obvious algorithms require explanation comments
- Complex state management requires documentation of state transitions

**Project documentation:**
- README.md covers installation and quick start
- CLAUDE.md provides AI assistant development guidance
- BUILD.md contains platform-specific build instructions
- Architecture documented inline in this constitution

**Feature documentation:**
- Specifications follow `.specify/templates/spec-template.md`
- Implementation plans follow `.specify/templates/plan-template.md`
- Tasks follow `.specify/templates/tasks-template.md`

## Writing Assistant Vision

This section defines the strategic direction for transforming Handy into a writing assistant.

**Near-term goals:**
- Context detection: Identify when user is writing code vs prose vs messages
- Smart formatting: Apply context-appropriate punctuation and capitalization
- Command mode: Voice commands for editing operations ("delete last sentence")
- Correction mode: "Actually I meant..." allows voice-driven corrections

**Medium-term goals:**
- Style memory: Learn user's writing patterns (formal vs casual, vocabulary)
- Multi-modal editing: Combine voice with keyboard for efficient editing
- Writing templates: Quick access to common structures (emails, bug reports)
- Integration plugins: Support for VS Code, Obsidian, Notion

**Long-term vision:**
- AI-assisted writing: Suggestions for clarity, conciseness, tone
- Research integration: Pull in context from documents, web, notes
- Collaboration features: Voice annotations, shared transcripts
- Accessibility focus: Writing assistance for users with different needs

**Non-goals:**
- Cloud-based processing (privacy is core to identity)
- Social/sharing features (stay focused on writing)
- Becoming a general-purpose AI assistant (writing is the scope)

## Governance

### Amendment Process

This constitution can be amended through the following process:

1. **Proposal:** Open GitHub issue tagged "constitution" with rationale
2. **Discussion:** Community feedback period (minimum 1 week)
3. **Approval:** Maintainer approval required for changes
4. **Implementation:** Update constitution.md with version increment
5. **Propagation:** Update all dependent templates and documentation

### Versioning Policy

Constitution version follows semantic versioning (MAJOR.MINOR.PATCH):

**MAJOR bump:** Removing or fundamentally changing core principles (backward incompatible governance)
**MINOR bump:** Adding new principles, expanding guidance (new requirements)
**PATCH bump:** Clarifications, typos, wording improvements (no semantic change)

### Compliance Review

**All pull requests must:**
- Verify compliance with core principles
- Pass automated tests (TDD requirement)
- Follow code style conventions
- Include appropriate documentation

**Maintainers must:**
- Reject PRs that violate principles without justification
- Request simplification when complexity is not warranted (Principle V)
- Verify new features align with writing assistant vision (Principle III)

**Quarterly reviews:**
- Assess constitution effectiveness
- Update principles based on project evolution
- Ensure templates remain aligned

### Development Guidance

For runtime development assistance, AI coding assistants should reference:
- `CLAUDE.md` - Claude Code specific guidance
- `BUILD.md` - Platform-specific build requirements
- This constitution - Architectural principles and standards

**Version**: 1.0.0 | **Ratified**: 2025-10-06 | **Last Amended**: 2025-10-06
