# DOCX Preprocessor - Introduction

This document describes the **DOCX Preprocessor**, a deterministic transformation tool that converts raw Microsoft Word (`.docx`) OOXML into a compact, LLM-friendly intermediate markup called **`words`**.

## Purpose

The preprocessor extracts semantic content from Word documents while stripping away verbose presentation-oriented XML. The result is a clean, token-efficient representation optimized for:

- AI/LLM training datasets
- Text mining and analysis
- Document understanding research
- Downstream processing pipelines

## Scope

This specification defines the **semantic transformation** from DOCX to `words` XML:

- **Input**: Raw `.docx` (a zip archive containing `word/document.xml` and supporting parts)
- **Output**: `words` XML - a flat, versioned, semantic representation
- **Not in scope**: DOCX regeneration, pixel-perfect reproduction. Only images and binary objects (OLE, charts, SmartArt, Math) are dropped. All text formatting (bold, italic, underline, font, size, color), borders, bookmarks, and comments are preserved in compact `words` syntax.

## Modes

The preprocessor operates in one of two modes:

- `mode="semantic"` (default): Stripped-down representation for AI training and downstream consumption. Presentation attributes removed, whitespace normalized.
- `mode="lossless"`: Preserves additional metadata for round-tripping or document reconstruction. Layout details preserved, tracked changes maintained.

## Key Principles

1. **Deterministic**: Identical DOCX input → identical `words` output
2. **Single responsibility**: Each transformation rule has one clear purpose
3. **Versioned format**: Format is versioned (`version="1.0.1"`) to allow evolution
4. **Clear separation**: Layout metadata in `<style>`, content in `<write>`

## Target Audience

This specification is intended for:

- Software engineers implementing the preprocessor
- Downstream system developers consuming `words` XML
- Technical writers documenting the format
