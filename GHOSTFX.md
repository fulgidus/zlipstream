# GHOSTFX.md — Internal Conversion Engine

GhostFX is the internal transformation engine of **zlipstream**, enabling on-the-fly conversion between file formats.

---

# 1. Purpose

GhostFX provides:
- Dependency-free conversions  
- Predictable, deterministic transformations  
- Pipeline-based composition  
- Optional fallback to external tools  

---

# 2. Pipeline Architecture

A pipeline is defined as:

```
input → primitive1 → primitive2 → ... → output
```

Each primitive transforms bytes into:
- Another byte buffer, or  
- A structured IR object  

Pipelines are executed entirely in memory.

---

# 3. Built-In Primitives

## 3.1 Text / Markup
- parse_md → Markdown AST  
- render_html → HTML  
- render_pdf_simple → Minimal PDF  

## 3.2 Image
- decode_png / encode_png  
- decode_jpeg / encode_jpeg  
- decode_webp / encode_webp  
- raster_svg_basic → Bitmap  

## 3.3 Structured Data
- parse_json / render_json  
- parse_yaml / render_yaml  
- parse_toml / render_toml  

## 3.4 Archives
- zip_extract, zip_pack  
- tar_extract, tar_pack  
- gzip, gunzip  

---

# 4. Execution Flow

1. Resolver requests conversion.  
2. GhostFX loads applicable rule.  
3. Executes pipeline.  
4. Returns output bytes or temp file path.  

---

# 5. Rule Example

```toml
[[rule]]
from = "md"
to   = "pdf"
pipeline = ["parse_md", "render_html", "render_pdf_simple"]
```

---

# 6. Error Handling

- Missing rule → error  
- Primitive failure → propagate `EIO`  
- Incompatible pipeline → validation error  

---

# End of GHOSTFX.md
