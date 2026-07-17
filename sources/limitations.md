# DOCX Preprocessor - Limitations and Exclusions

This document specifies what the DOCX Preprocessor intentionally does NOT handle.

## Excluded by Policy

These constructs are **out of scope** for v1.0.1:

| Construct | Source | Behavior | Impact |
|-----------|--------|----------|--------|
| Images (non-textbox) | `w:drawing` image blip, `w:pict` (VML) | `<img alt="..."/>` placeholder | No pixel/vector data |
| OLE objects | `w:object` | dropped | Embedded spreadsheets/Visio lost |
| Charts | chart drawing parts | dropped | Data visualizations lost |
| SmartArt/diagrams | smartart drawing parts | dropped | Flowcharts/org-charts lost |
| Office Math | `w:Math` (OMML) | dropped | Equations absent |
| External chunks | `w:altChunk` | dropped | Embedded HTML lost |

**Rationale**: These require specialized renderers and/or carry binary payloads. Deterministic conversion would bloat token budget.

**Textboxes (`w:txbxContent`) are NOT excluded** - their text content is extracted into `<write>` (CRIT-1). Images embedded inside a textbox are still excluded as `<img>`.

## Lossless Metadata Preserved

These are preserved in `LOSSLESS_METADATA` category for round-tripping:

| Construct | Source | Preservation | Notes |
|-----------|--------|--------------|-------|
| Paragraph alignment | `w:pPr/w:jc` | `<s:align>` in `<style>` | Useful for layout-aware processing |
| Line spacing | `w:pPr/w:spacing/@w:line` | `<s:line>` in `<style>` — P1 | Keyed by `el` + optional `c` |
| Tracked changes | `w:ins`/`w:del` | `<ins>`/`<del>` (lossless mode only) | Revision tracking preserved |
| Bookmarks | `w:bookmarkStart` | `<bm id="name"/>` in `<notes>` | Bookmark positions preserved |
| Comments | `w:commentRange` | `<comment>` in `<notes>` | Comment text preserved |
| Paragraph borders | `w:pPr/w:pBdr` | `at="..."` on `<p>`/`<h1>`-`<h9>` | Compact border representation |
| Table/cell borders | `w:tblPr`/`w:tcPr` | `at="..."` on `<table>`/`<td>`/`<th>` | Compact border representation |
| Run language | `w:rPr/w:lang` | `<span lang="...">` — P2 | BCP 47 language tag |
| Hidden text | `w:rPr/w:vanish` | `<span hidden="true">` — P9 | Visibility flag |
| Table caption | `w:tblPr/w:tblCaption` | `<table caption="...">` — P3 | Accessibility |
| Table summary | `w:tblPr/w:tblDescription` | `<table summary="...">` — P4 | Accessibility |
| Table width | `w:tblPr/w:tblW` | `<table width="...">` — P5 | In declared unit |
| Table alignment | `w:tblPr/w:jc` | `<table align="...">` — P6 | left|center|right |
| Cell vertical alignment | `w:tcPr/w:vAlign` | `<td valign="...">` / `<th valign="...">` — P7 | top|center|bottom |
| Text vertical alignment | `w:pPr/w:textAlignment` | `<p valign="...">` — P8 | top|center|baseline |
| Table indentation | `w:tblPr/w:tblInd` | `<table indent="...">` — P10 | In declared unit |
| Cell spacing | `w:tblPr/w:tblCellSpacing` | `<table cellSpacing="...">` — P11 | In declared unit |
| Text direction in cell | `w:tcPr/w:textDirection` | `<td textDir="...">` / `<th textDir="...">` — P12 | bttRtl, tbRl, etc. |
| No-wrap flag | `w:tcPr/w:noWrap` | `<td noWrap="true">` / `<th noWrap="true">` — P13 | Boolean flag |
| Multi-column layout | `w:sectPr/w:cols` | `<s:cols n=".." space=".."/>` — P19 | Column count + spacing |
| Tab stops | `w:pPr/w:tabs` | `<s:tab el=".." pos=".." align=".." leader=".."/>` | Position + alignment + leader |
| Tab character | `w:tab` | `<tab/>` | Moves to next tab stop |

## Language Limitations

| Issue | Impact | Resolution |
|-------|--------|------------|
| Inline language changes | `<span lang="...">` captures run-level language (P2) | Full language preservation |
| `<span>` scope | `<span>` supports font/size/color/highlight/lang/hidden | Full inline coverage |

## Image Handling

Images remain a **hard limitation**:
- Only `<img alt="..."/>` placeholder emitted
- No pixel/vector data extracted
- No caption, title, or positioning preserved

For image processing, run separate OCR/Vision pass and inject captions before downstream processing.

## Table Styling

| Aspect | Preserved | Lost |
|--------|-----------|------|
| Borders | YES (via `at` attribute) | — |
| Shading | NO | Fill color/pattern |
| Column widths | YES (via `<s:col>`) | Per-cell `w:tcW` overrides |
| Table width | YES (`<table width="...">`) — P5 | — |
| Table alignment | YES (`<table align="...">`) — P6 | — |
| Table indentation | YES (`<table indent="...">`) — P10 | — |
| Cell spacing | YES (`<table cellSpacing="...">`) — P11 | — |
| Table caption/summary | YES (`<table caption="..." summary="...">`) — P3/P4 | — |
| Cell vertical alignment | YES (`<td valign="...">` / `<th valign="...">`) — P7 | — |
| Text direction in cell | YES (`<td textDir="...">` / `<th textDir="...">`) — P12 | — |
| No-wrap flag | YES (`<td noWrap="true">` / `<th noWrap="true">`) — P13 | — |

## Footnote/Endnote Limitations

| Aspect | Status |
|--------|--------|
| Markers | `<fn-ref id="n" type="..."/>` |
| Bodies | `<fn id="n" type="...">` in `<notes>` |
| Cross-references | Not resolved |

## Style Resolution

| Aspect | Resolution |
|--------|------------|
| Built-in styles | Mapped to semantic elements |
| Custom styles | `c="<customName>"` attribute on element + `<s:custom>` in `<style>` |
| Style inheritance | `w:basedOn` chain walked to find semantic match |
| Style properties | Preserved in `<s:custom>` (font, size, color, alignment, spacing, etc.) |

## Metadata Loss

| Metadata | Status |
|----------|--------|
| Author, title, dates | Preserved in `<meta>` |
| Revision history | Only tracked changes (`<ins>`/`<del>`) |
| Comments | Preserved in `<notes>` as `<comment>` |
| Bookmarks | Preserved in `<notes>` as `<bm>` |
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
3. Language preservation now full (block-level `lang` + `<span lang="...">` for inline)
4. No field code preservation (except HYPERLINK)
5. No revision tracking outside `mode="lossless"`
6. Shading intentionally dropped (borders preserved via `at`)

## External Dependencies

The preprocessor requires:
- `word/document.xml` - main content
- `word/styles.xml` - style resolution (when needed)
- `word/numbering.xml` - list numbering
- `word/document.xml.rels` - relationship resolution
- `word/footnotes.xml` / `word/endnotes.xml` - footnote/endnote bodies

Missing files result in graceful degradation or omitted elements.
