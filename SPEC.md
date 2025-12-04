# SPEC.md — zlipstream Specification

This document defines the formal specification of **zlipstream**, including behavior, guarantees, edge-case semantics, and system constraints.

## 1. Purpose

zlipstream provides **transparent file-extension virtualization**, allowing any process to access files in formats that do not exist on disk by generating them dynamically through built-in conversion pipelines.

This specification defines:
- Expected system behavior
- File access semantics
- Error handling
- Conversion rules
- Injection requirements

---

## 2. System Guarantees

### 2.1 Transparency Guarantee
All processes launched by the user *must* experience zlipstream behavior without:
- Shell wrappers  
- PATH modifications  
- Modified binaries  
- Kernel extensions  

### 2.2 POSIX Compatibility
All overridden calls must preserve POSIX semantics:
- Correct `errno` propagation  
- Respect file flags (O_RDONLY, O_WRONLY, O_CREAT, etc.)  
- Respect permissions and umask  
- Correct behavior on concurrency and race conditions  

### 2.3 Non-Invasiveness
zlipstream never:
- Alters libc  
- Patches binaries  
- Modifies kernel state  
- Writes to disk unless explicitly triggered by writes or renames  

### 2.4 Deterministic Behavior
Conversion selection must be deterministic based on:
1. Virtual extension requested  
2. Source file existence  
3. Matching configured rules  

---

## 3. Virtual Extension Semantics

### 3.1 Resolution Rules
Given a requested `target.extB`:

1. If a real file exists → open normally.  
2. Else if a rule exists mapping extA → extB and `target.extA` exists:
   - Generate a virtual file using the conversion pipeline  

### 3.2 Visibility Modes

#### Visible Mode
Virtual extensions injected into directory listings.

#### Invisible Mode
Virtual files appear only when opened.

---

## 4. Conversion Specification

### 4.1 Pipeline Model
A pipeline is ordered list of transformation primitives:
```
input → step1 → step2 → ... → output
```

### 4.2 Atomicity
- Entire pipeline must succeed before exposing output  
- Failures produce correct errno and no partial output  

### 4.3 Persistence on Write
Opening a virtual file with write flags materializes a real file.

### 4.4 Persistence on Rename
Renaming a file to a virtual extension triggers conversion and materialization.

---

## 5. Error Handling

- No matching rule → `ENOENT`
- Pipeline errors → `EIO`
- Permission denied → `EACCES` or `EPERM`
- Ambiguous source extension → `EINVAL`

---

## 6. Injection Requirements

zlipstream relies on:
- A user-space injection daemon  
- LD_PRELOAD insertion before execve  
- Shared library interposition  

---

## 7. Boundaries of Operation

zlipstream does NOT virtualize:
- Sockets  
- Pipes  
- Devices  
- /proc or /sys  
- Special files  

Only regular files and dirs.

---

## 8. Security Model

- Runs under user UID  
- No privilege escalation  
- Converts only user-writable paths  

---

# End of SPEC.md
