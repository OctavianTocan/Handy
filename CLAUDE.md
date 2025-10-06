# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Prerequisites:**
- [Rust](https://rustup.rs/) (latest stable)
- [Bun](https://bun.sh/) package manager
- Platform-specific dependencies (see BUILD.md for details)

**Core Development:**
```bash
# Install dependencies
bun install

# Run in development mode
bun run tauri dev
# If cmake error on macOS:
CMAKE_POLICY_VERSION_MINIMUM=3.5 bun run tauri dev

# Build for production
bun run tauri build

# Frontend only development
bun run dev        # Start Vite dev server
bun run build      # Build frontend (TypeScript + Vite)
bun run preview    # Preview built frontend

# Type checking
bunx tsc --noEmit  # Run TypeScript type checker
```

**Model Setup (Required for Development):**
```bash
# Create models directory and download VAD model
mkdir -p src-tauri/resources/models
curl -o src-tauri/resources/models/silero_vad_v4.onnx https://blob.handy.computer/silero_vad_v4.onnx
```

## Architecture Overview

Handy is a cross-platform desktop speech-to-text application built with Tauri (Rust backend + React/TypeScript frontend).

### Core Components

**Backend (Rust - src-tauri/src/):**
- `lib.rs` - Main application entry point with Tauri setup, tray menu, and managers
- `managers/` - Core business logic managers:
  - `audio.rs` - Audio recording and device management
  - `model.rs` - Whisper model downloading and management  
  - `transcription.rs` - Speech-to-text processing pipeline
- `audio_toolkit/` - Low-level audio processing:
  - `audio/` - Device enumeration, recording, resampling 
  - `vad/` - Voice Activity Detection using Silero VAD
- `commands/` - Tauri command handlers for frontend communication
- `shortcut.rs` - Global keyboard shortcut handling
- `settings.rs` - Application settings management

**Frontend (React/TypeScript - src/):**
- `App.tsx` - Main application component with onboarding flow
- `components/settings/` - Settings UI components
- `components/model-selector/` - Model management interface
- `hooks/` - React hooks for settings and model management
- `lib/types.ts` - Shared TypeScript type definitions

### Key Architecture Patterns

**Manager Pattern:** Core functionality is organized into managers (Audio, Model, Transcription) that are initialized at startup and managed by Tauri's state system.

**Command-Event Architecture:** Frontend communicates with backend via Tauri commands, backend sends updates via events.

**Pipeline Processing:** Audio → VAD → Whisper → Text output with configurable components at each stage.

### Technology Stack

**Core Libraries:**
- `whisper-rs` - Local Whisper inference with GPU acceleration
- `transcribe-rs` - CPU-optimized Parakeet V3 model support
- `cpal` - Cross-platform audio I/O
- `vad-rs` - Voice Activity Detection (Silero VAD)
- `rdev` - Global keyboard shortcuts and system events
- `rubato` - Audio resampling
- `rodio` - Audio playback for feedback sounds
- `tauri-plugin-store` - Settings persistence

**Platform-Specific Features:**
- macOS: Metal acceleration for Whisper, accessibility permissions
- Windows: Vulkan acceleration, code signing
- Linux: OpenBLAS + Vulkan acceleration

**Available Models:**
- Whisper Small/Medium/Turbo/Large (GPU-accelerated)
- Parakeet V3 (CPU-optimized with auto language detection)

### Application Flow

1. **Initialization:** App starts minimized to tray, loads settings, initializes managers
2. **Model Setup:** First-run downloads preferred Whisper model (Small/Medium/Turbo/Large)
3. **Recording:** Global shortcut triggers audio recording with VAD filtering
4. **Processing:** Audio sent to Whisper model for transcription
5. **Output:** Text pasted to active application via system clipboard

### Settings System

Settings are stored using Tauri's store plugin with reactive updates:
- Keyboard shortcuts (configurable, supports push-to-talk)
- Audio devices (microphone/output selection)
- Model preferences (Small/Medium/Turbo/Large Whisper variants)
- Audio feedback and translation options

### Single Instance Architecture

The app enforces single instance behavior - launching when already running brings the settings window to front rather than creating a new process.

## Code Style Guidelines

**Rust (Backend):**
- Use `anyhow::Error` for error handling with descriptive context messages
- Prefer `Arc<Mutex<T>>` for shared state in managers
- Log with appropriate levels: `debug!()`, `info!()`, `eprintln!()` for errors
- Builder pattern for initialization chains
- Snake_case for functions and variables, PascalCase for types
- Separate logical sections with comment blocks: `/* ─────────── */`
- Use `?` operator with anyhow context for error propagation

**TypeScript/React (Frontend):**
- Functional components with TypeScript interfaces
- Zod schemas for type validation and inference (see `src/lib/types.ts`)
- `useCallback` hooks for stable function references in components
- Destructure props with defaults: `{ disabled = false }`
- Prefer interface aliases over type aliases for objects
- PascalCase for components, camelCase for variables/functions
- Use `import type` for TypeScript type imports

**Error Handling:**
- Frontend: Try/catch with user feedback via toast notifications, rollback optimistic updates
- Backend: `?` operator with `.context()` for descriptive error messages
- Log errors at appropriate debug levels

**Component Patterns:**
- Container component pattern for layout (see `App.tsx`)
- Composition over inheritance
- Use Zustand for global state management (see `src/stores/`)
- Minimize prop drilling with context where appropriate

## Important Development Notes

**Manager Initialization:**
- All managers (Audio, Model, Transcription, History) are initialized in `lib.rs` and managed via Tauri state
- Managers use `Arc<Mutex<T>>` for thread-safe shared access
- Frontend accesses managers via Tauri commands in `src-tauri/src/commands/`

**Audio Processing Pipeline:**
- Recording happens in `audio_toolkit/audio/recorder.rs`
- VAD filtering in `audio_toolkit/vad/` (Silero model required)
- Resampling in `audio_toolkit/audio/resampler.rs` (48kHz → 16kHz for Whisper)
- Transcription orchestrated by `managers/transcription.rs`

**Settings Management:**
- Settings stored using `tauri-plugin-store` with JSON persistence
- Reactive updates via events from backend to frontend
- Frontend hooks in `src/hooks/` provide reactive access to settings

**Platform Permissions:**
- macOS requires accessibility and microphone permissions
- Handled via `tauri-plugin-macos-permissions` and native permissions API
- Permission checks in `AccessibilityPermissions.tsx`
