# DOCX Preprocessor - Limitations and Exclusions

This document specifies what the DOCX Preprocessor intentionally does NOT handle.

## Excluded by Policy

These constructs are **out of scope** for v1.0.1:

| Construct | Source | Behavior | Impact |
|-----------|--------|----------|--------|
| Images (non-textbox) | `w:drawing` image blip, `w:pict` (VML) | `<d:img alt="..."/>` placeholder | No pixel/vector data |
| OLE objects | `w:object` | dropped | Embedded spreadsheets/Visio lost |
| Charts | chart drawing parts | dropped | Data visualizations lost |
| SmartArt/diagrams | smartart drawing parts | dropped | Flowcharts/org-charts lost |
| Office Math | `w:Math` (OMML) | dropped | Equations absent |
| External chunks | `w:altChunk` | dropped | Embedded HTML lost |

**Rationale**: These require specialized renderers and/or carry binary payloads. Deterministic conversion would bloat token budget.

**Textboxes (`w:txbxContent`) are NOT excluded** - their text content is extracted into `<write>` (CRIT-1). Images embedded inside a textbox are still excluded as `<d:img>`.

## Lossless Metadata Preserved

These are preserved in `LOSSLESS_METADATA` category for round-tripping:

| Construct | Source | Preservation | Notes |
|-----------|--------|--------------|-------|
| Paragraph alignment | `w:pPr/w:jc` | `<s:align>` in `<style>` | Useful for layout-aware processing |
| Tracked changes | `w:ins`/`w:del` | `<d:change>` (lossless mode only) | Revision tracking preserved |

## Language Limitations

| Issue | Impact | Resolution |
|-------|--------|------------|
| Inline language changes | Language context lost for runs with different language than paragraph | First run's language takes precedence |
| No `<d:span>` element | Cannot preserve inline language or styling changes | Block-level `lang` only |

## Image Handling

Images remain a **hard limitation**:
- Only `<d:img alt="..."/>` placeholder emitted
- No pixel/vector data extracted
- No caption, title, or positioning preserved

For image processing, run separate OCR/Vision pass and inject captions before downstream processing.

## Table Styling

| Aspect | Preserved | Lost |
|--------|-----------|------|
| Borders | NO | Border width/style/color |
| Shading | NO | Fill color/pattern |
| Column widths | YES (via `<s:col>`) | Per-cell `w:tcW` overrides |

## Footnote/Endnote Limitations

| Aspect | Status |
|--------|--------|
| Markers | `<d:fn-ref id="n" type="..."/>` |
| Bodies | `<d:fn id="n" type="...">` in `<notes>` |
| Cross-references | Not resolved |

## Style Resolution

| Aspect | Resolution |
|--------|------------|
| Built-in styles | Mapped to semantic elements |
| Custom styles | `c="<customName>"` attribute on element |
| Style inheritance | `w:basedOn` chain walked to find semantic match |

## Metadata Loss

| Metadata | Status |
|----------|--------|
| Author, title, dates | Preserved in `<meta>` |
| Revision history | Only tracked changes (`<d:change>`) |
| Comments | Dropped |
| Bookmarks | Dropped |
| Field codes | Dropped (except HYPERLINK) |

## Performance Constraints

| Limit | Reason |
|-------|--------|
| Maximum nesting depth | Prevents stack overflow |
| Maximum table depth | Prevents deeply nested structures |
| Maximum footnote/endnote depth | Prevents deeply nested notes |

## Known Limitations

1. No DOCX regeneration capability
2. No pixel-perfect layout preservation
3. No language preservation at inline/run level
4. No comment preservation
5. No field code preservation (except HYPERLINK)
6. No revision tracking outside `mode="lossless"`

## External Dependencies

The preprocessor requires:
- `word/document.xml` - main content
- `word/styles.xml` - style resolution (when needed)
- `word/numbering.xml` - list numbering
- `word/document.xml.rels` - relationship resolution
- `word/footnotes.xml` / `word/endnotes.xml` - footnote/endnote bodies

Missing files result in graceful degradation or omitted elements.
