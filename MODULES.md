# MODULES.md — zlipstream Module Breakdown

This document describes the internal modules that compose **zlipstream**.

---

# 1. zlipstreamd — Injection Daemon

## 1.1 Process Monitor
- Detects new processes via ptrace/fanotify.
- Captures execve events.

## 1.2 Injection Engine
- Injects LD_PRELOAD into process environment.
- Ensures libzlipstream.so loads before program main.

## 1.3 IPC Server
- Unix socket for communication with interposer.
- Handles conversion requests and returns results.

## 1.4 Cache Manager
- Optional memory/disk caching.
- Hash-based deduplication.

---

# 2. libzlipstream.so — Interposition Layer

## 2.1 Hook Layer
- Overrides libc filesystem calls.
- Delegates to resolver when needed.

## 2.2 Resolver
- Identifies virtual extensions.
- Selects matching rules.
- Contacts daemon when pipeline execution is required.

## 2.3 Syscall Adapter
- Wraps open/stat/rename calls.
- Produces FDs backed by ephemeral files.

## 2.4 Write/Rename Handler
- Detects writes to virtual files.
- Materializes real files on disk.

---

# 3. GhostFX — Conversion Engine

## 3.1 Pipeline Interpreter
- Executes transformation steps sequentially.
- Strict type checking.

## 3.2 Transformation Primitives
- Markdown → HTML  
- HTML → PDF  
- PNG/JPEG/WebP decode/encode  
- SVG raster  
- JSON/YAML/TOML parse/render  
- Archive pack/unpack  

## 3.3 Rule Loader
- Loads TOML/YAML rules.
- Validates pipelines and primitives.

## 3.4 Engine Runtime
- Manages buffers and temp dirs.
- Ensures atomic writes.

---

# End of MODULES.md
