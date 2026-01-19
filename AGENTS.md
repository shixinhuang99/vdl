# Vdl – Project Architecture & Design Notes

## Project Overview

**Vdl** is a personal macOS desktop application built for learning and experimentation.
Its primary purpose is to provide a native GUI frontend for `yt-dlp`, using **GPUI** (the Rust GUI framework behind Zed) as the UI layer.

The project is intentionally scoped as:

- A non-commercial tool
- A learning vehicle for **GPUI**, Rust async/task management, and process integration
- A thin, well-structured wrapper around `yt-dlp`, not a reimplementation

---

## High-Level Architecture

Vdl follows a clear separation of concerns:

```
┌────────────────────────┐
│        GPUI UI         │
│  - URL input           │
│  - Format selection    │
│  - Progress display    │
│  - Status / errors     │
└───────────┬────────────┘
            │ (events / messages)
┌───────────┴────────────┐
│     Rust Application   │
│  - Task orchestration  │
│  - yt-dlp invocation   │
│  - JSON parsing        │
│  - Progress parsing    │
└───────────┬────────────┘
            │ (system process)
┌───────────┴────────────┐
│    External Tools      │
│  - yt-dlp (CLI)        │
│  - ffmpeg (optional)   │
└────────────────────────┘
```

---

## Core Design Principles

1. **yt-dlp is treated as an external dependency**
   - Vdl never reimplements site extractors or download logic
   - All media handling is delegated to `yt-dlp` via its CLI interface

2. **Rust owns orchestration, not media logic**
   - Rust code is responsible for:
     - Spawning processes
     - Parsing JSON output
     - Tracking progress and state
     - Updating the UI

   - Rust does _not_ parse media streams or handle codecs

3. **GPUI is used as a native UI layer**
   - No web views or Electron-style embedding
   - UI is event-driven and updated from background tasks
   - Long-running downloads must never block the UI thread

---

## yt-dlp Integration Strategy

Vdl interacts with `yt-dlp` exclusively via the command line.

### Metadata / Format Discovery

To inspect a URL and list available formats:

```bash
yt-dlp -j <URL>
```

- JSON output is parsed in Rust
- `formats` are mapped to UI-friendly representations
- Default behavior prefers:
  - Best video + best audio
  - Automatic merging when `ffmpeg` is available

### Download Execution

Downloads are started using:

```bash
yt-dlp -f <format> <URL>
```

- `stdout` is read line-by-line
- Progress information is parsed and forwarded to the UI
- Download tasks run in background threads or async tasks

---

## Audio / Video Handling Model

Modern platforms often provide **separate audio and video streams** (DASH).

Key assumptions:

- yt-dlp handles stream selection
- yt-dlp invokes `ffmpeg` to merge audio and video
- If `ffmpeg` is missing, the user may receive video-only files

Vdl does not manually merge streams.

---

## Initial MVP Feature Scope

The initial version of Vdl should focus on:

- Single URL input
- Format inspection via `yt-dlp -j`
- One-click download (default best quality)
- Real-time progress display
- Clear success / failure states

Explicitly out of scope for MVP:

- Account management
- Built-in cookies editor
- Advanced post-processing
- Multi-user or server features

---

## Non-Goals

- Replacing `yt-dlp`
- Supporting DRM-protected content
- Acting as a general-purpose download manager
- Providing legal or policy enforcement

---

## Naming & Identity

- App name: **Vdl**
- Naming intentionally avoids platform-specific references (YouTube, TikTok, etc.)
- The app presents itself as a neutral “media fetcher”, not a platform downloader

---

## Guidance for AI Agents

When assisting with this project, AI agents should:

- Prefer **simple, explicit Rust code** over clever abstractions
- Respect the separation between UI, orchestration, and external tools
- Avoid reimplementing functionality already provided by `yt-dlp`
- Keep changes aligned with the MVP scope unless explicitly requested
- Assume macOS as the primary target platform

---

This document defines the architectural intent of the project and should be treated as the source of truth for early development decisions.
