# ARCHITECTURE.md — zlipstream Architecture

This document describes the high‑level architecture of **zlipstream**, a user‑space file‑extension virtualization layer providing transparent, on‑the‑fly file format transformations for all user processes.

---

# 1. System Overview

zlipstream is composed of three cooperating components:

1. **Injection Daemon (`zlipstreamd`)**  
   Ensures that every new process started by the user loads the zlipstream interposition library.

2. **Interposition Layer (`libzlipstream.so`)**  
   A shared library injected into all user processes.  
   It overrides selected filesystem‑related libc functions to virtualize file access.

3. **Conversion Engine (GhostFX)**  
   Internal pipeline‑based transformation engine that generates virtual files when needed.

All components run entirely in **user‑space**, without FUSE, kernel modules, or modified executables.

---

# 2. Component Interaction Diagram

```
 +-----------------------+
 |   User Applications   |
 | (shell, GUI, IDE...)  |
 +-----------+-----------+
             |
             |  1) Process creation
             v
 +-----------------------+      2) Injects LD_PRELOAD
 |   zlipstreamd         +--------------------------------+
 | (Injection Daemon)    |                                |
 +-----------+-----------+                                |
             |                                            |
             | 3) zlipstream.so active                   |
             v                                            |
 +---------------------------+                            |
 |  libzlipstream.so         |<----------------------------+
 | (Interposition Layer)     |
 +---------------+-----------+
                 |
                 | 4) Needs virtual file contents
                 v
 +---------------------------+
 |       GhostFX Engine      |
 | (in-memory pipelines)     |
 +---------------+-----------+
                 |
                 | 5) Returns bytes to caller
                 v
 +---------------------------+
 | Real Filesystem          |
 +---------------------------+
```

---

# 3. Injection Daemon (`zlipstreamd`)

### Responsibilities

- Monitors new processes created by the user using one of:
  - `ptrace`‑based exec trapping  
  - `fanotify`  
  - A dedicated user namespace environment  
- Injects `LD_PRELOAD` pointing to `libzlipstream.so` before the process executes its main binary.
- Maintains lightweight state about which processes are under virtualization.
- Communicates with `libzlipstream.so` through a Unix domain socket when conversions are required.

### Properties

- Requires **no root** after installation.
- Does **not** modify executables, PATH, shells, or launchers.
- Is active for the entire user session.

---

# 4. Interposition Layer (`libzlipstream.so`)

This shared library performs syscall‑level virtualization by overriding specific libc functions.

### Interposed Functions

- `open`, `openat`
- `stat`, `lstat`, `fstatat`
- `rename`, `renameat`, `renameat2`
- `unlink`
- `readdir` (only when virtual extension visibility is enabled)

### Behavior

1. For every file operation, check whether:
   - the file exists physically  
   - a virtual extension rule applies  

2. When a virtual extension is accessed:
   - Request data from GhostFX through the daemon
   - Serve generated content to the process
   - Maintain POSIX‑compatible semantics (errno, flags, permissions)

3. For writes or renames:
   - Trigger GhostFX persistent conversion
   - Write the resulting file to disk following standard semantics

### Isolation

The interposition layer *never* mutates or replaces libc.  
It intercepts at link‑time through LD_PRELOAD semantics only.

---

# 5. GhostFX Conversion Engine

### Overview

GhostFX is a pluggable, pipeline-based transformation system used to generate virtual files or persist converted files.

### Pipeline Model

A pipeline transforms:

```
input bytes → step1 → step2 → ... → output bytes
```

Example:

```
SVG → rasterize → PNG
MD → parse → HTML → PDF
JSON → parse → YAML
```

### Built-in Transformation Primitives

- **Text / Markup**
  - Markdown parser
  - HTML renderer
  - Minimal PDF generator

- **Images**
  - PNG/JPEG/WebP decode/encode
  - Basic SVG rasterizer

- **Structured Data**
  - JSON/YAML/TOML parsing and rendering

- **Archives**
  - zip, tar, gzip mini-implementations

External tools (pandoc, ffmpeg, imagemagick) can be used optionally, but are not required.

### Execution Context

GhostFX is executed:
- In the daemon when conversions are needed  
- In-memory, streaming-friendly  
- Returning temporary file handles or direct byte buffers

---

# 6. File Virtualization Logic

### Read Access

1. A process attempts to open `target.extB`.
2. `libzlipstream.so` checks:
   - Does `target.extB` exist? → open normally  
   - Is there a rule `extA → extB`?  
   - Does `target.extA` exist?  
3. If valid, request `extB` generation from GhostFX.
4. Provide FD backed by ephemeral file or memory.

### Write Access

When a process writes to a virtual extension:

- Create a **real** file on disk
- Populate it using the corresponding virtual pipeline
- Continue write operations as normal

### Rename Access

`rename(old.extA, new.extB)`:

- If `extB` is a virtual extension
  - Trigger conversion pipeline
  - Write real `new.extB`
  - Remove original file if required

---

# 7. Configuration Model

A user config file (TOML or YAML):

- Lists virtual extensions per type
- Defines allowed pipelines
- Controls visibility mode
- Controls optional usage of external converters

Example:

```toml
visible_virtuals = true

[[rule]]
from = "md"
to   = "pdf"
pipeline = ["parse_md", "render_html", "render_pdf_simple"]
```

---

# 8. Caching Strategy

zlipstream may maintain:

- In-memory LRU cache for frequently requested conversions  
- Optional temporary filesystem cache for large artifacts  
- Content hashes to avoid repeated conversions  

Caching is purely optional and transparent.

---

# 9. Error Semantics

The system preserves POSIX semantics:

- If conversion fails → return appropriate `errno` (e.g., `ENOENT`, `EIO`)
- If multiple source candidates exist → deterministic selection
- If user disables an extension → behave as if file does not exist

---

# 10. Security Model

- All operations run in user space under the user’s UID  
- No elevated permissions  
- No kernel extensions
- All generated files have correct ownership and mode
- External tool usage is disabled unless explicitly allowed

---

# 11. Summary

zlipstream is architected as:

- A global user-space file virtualization layer  
- A robust LD_PRELOAD interposer injected transparently  
- A powerful internal conversion engine  
- A POSIX-compliant transformer for reads, writes, and renames  
- A system that acts like a “format‑shifting filesystem layer” without touching the kernel

This architecture enables transparent, safe, and universal access to virtual file formats for any Unix user process.
