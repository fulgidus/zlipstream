# zlipstream
_Files that become what you need._
## 🚀 Overview
Zlipstream is a user-space virtualization layer that creates transparent virtual formats above any existing filesystem.
It requires no kernel modules, no FUSE, and no binary wrappers, thanks to an advanced injection engine capable of automatically intercepting every file access from any process — shells, GUI apps, services, child processes, and more.

With Zlipstream, applications can open files in formats that do not physically exist on disk.
If file.svg is present, opening file.png or file.webp works seamlessly: Zlipstream generates virtual versions on the fly using an internal transformation engine.

### Core features

- OS-transparent: acts like an OS-level feature, without touching the kernel.
- User-space injection: works anywhere, no admin privileges needed.
- Virtual formats: define which extensions should exist “virtually” in each directory.
- Automatic pipelines: convert files on read/write using a builtin transformation engine (markdown → HTML → PDF, SVG → PNG, JSON ↔ YAML, etc.).
- Materialization: if an application writes into a virtual file (e.g., a.png), Zlipstream produces a real converted file with correct semantics (mv, rename, stat, etc.).
- Configurable through rules: per-directory, per-extension, or per-pipeline logic.
- Optional external tools: integrates with external converters, but does not depend on them.

Zlipstream redefines filesystem interoperability by making file formats fluid, transparent, and instantly interchangeable - without modifying applications, the kernel, or system configuration.

