# DOCX Preprocessor - Transformation Rules

This document defines the transformation rules from DOCX OOXML to `words` XML.

## Classification Categories

| Category | Description | Examples |
|----------|-------------|----------|
| **KEEP** | Mapped to semantic `words` elements | `w:p` → `<d:p>`, `w:r` → text content |
| **LOSSLESS_METADATA** | Preserved as non-lossy metadata | `w:pPr/w:jc` → `<s:align>`, `w:ins`/`w:del` → `<change>` |
| **DROP** | Presentation noise safely removed | `w:pPr/w:shd`, `w:rPr/w:spacing`, `w:pPr/w:tabs` |
| **EXCLUDE** | Out of scope / too complex | `w:drawing` (images), `w:object` (OLE), `w:Math` |

## Paragraph Transformation

### Element Mapping

| Source Style | Target Element | Notes |
|--------------|----------------|-------|
| `Heading1`-`Heading9` | `<d:h>` | Level inferred from heading number |
| `Title` | `<d:h>` | Title as heading level 0 |
| `ListParagraph` + numPr | `<d:li>` | Inside `<d:ul>` or `<d:ol>` |
| `Quote`/`IntenseQuote`/`BlockText` | `<d:quote>` | MIN-2 requirement |
| `Normal` or none | `<d:p>` | Default paragraph |
| Code-like styles | `<d:code>` | See: §3.1 Code Detection |

### Code Block Detection

A paragraph maps to `<d:code>` when:
1. Style name contains `"Code"`, `"Source"`, or `"Output"` as a word, OR
2. First run uses monospace font (`Courier New`, `Consolas`, `Lucida Console`, `Menlo`, `Monaco`, `monospace`)

When code block is detected:
- All inline formatting (`<b>`, `<i>`, etc.) is suppressed
- Original whitespace preserved (see: §3.5 Text cleanup)

## Run Transformation

### Inline Formatting

| Source | Target |
|--------|--------|
| `w:rPr/w:b` | `<b>` |
| `w:rPr/w:i` | `<i>` |
| `w:rPr/w:u` | `<u>` |
| `w:rPr/w:strike`/`dstrike` | `<s>` - CRIT-3 |
| `w:rPr/w:smallCaps` | `<smallcaps>` - MOD-4 |
| `w:rPr/w:caps` | `<uppercase>` - MOD-4 |
| `w:rPr/w:vertAlign="superscript"` | `<sup>` |
| `w:rPr/w:vertAlign="subscript"` | `<sub>` |
| `w:rPr/w:rFonts` | `<span font="..">` (font family from `@w:ascii` or `@w:hAnsi`) |
| `w:rPr/w:sz` | `<span size="..">` (font size in pt, `w:val ÷ 2`) |
| `w:rPr/w:color` | `<span color="..">` (hex color from `@w:val`) |
| `w:rPr/w:highlight` | `<span highlight="..">` (color name from `@w:val`) |

### Hyperlink Resolution

Resolve target in order:
1. `w:hyperlink/@r:id` → look up `document.xml.rels` → `<a href="...">`
2. `w:hyperlink` with `w:instrText` containing `HYPERLINK "..."` → extract URL → `<a href="...">`
3. Internal/bookmark targets → `<a href="#bookmarkName">`

## List Transformation

### Grouping Rules

Consecutive `ListParagraph` paragraphs with same `w:numId` form one list. Grouping MUST consider:
- `w:numId` - numeric ID of numbering definition
- `w:ilvl` - indent level (drives nesting)
- `w:abstractNumId` - resolved from `numbering.xml`
- Restart state: `w:lvlOverride` with `w:startOverride` forces split
- Different `w:abstractNumId` forces split

### Numbering Restart

Detect `w:lvlOverride` in `numbering.xml`:
- When `w:lvlOverride/w:startOverride/@w:val` resets numbering
- Split list into new `<d:ol>` with `start="n"` attribute
- Default `start="1"` if not specified

### List Types

| numFmt | Target Type |
|--------|-------------|
| `bullet` | `<d:ul type="bullet">` |
| `decimal` | `<d:ol type="decimal">` |
| `lowerLetter`/`upperLetter` | `<d:ol type="lowerLetter|upperLetter">` |
| `lowerRoman`/`upperRoman` | `<d:ol type="lowerRoman|upperRoman">` |
| Any other | `<d:ol type="...">` - preserve raw value (MOD-2) |

## Table Transformation

### Grid Reconstruction

Vertical merge (`w:vMerge`) requires grid reconstruction:
1. Parse all rows to build virtual grid
2. `w:vMerge w:val="restart"` starts vertical merge group
3. Subsequent `w:vMerge` (no val, "continue") are part of group
4. Continue-cells omitted from output
5. Restart cell gets `rowspan="n"` where `n` = count of continues + 1
6. Default `rowspan="2"` if no prior restart

### ID Assignment

Tables receive 1-based IDs in **pre-order traversal**:
- Parent table ID assigned before child tables
- Nested tables receive IDs sequentially
- `<s:col ref="n">` matches `<d:table id="n">`

## Textbox Transformation

### Inline Anchor Handling

When textbox anchored inside `<w:r>`:
1. Host paragraph runs before/after anchor merged into single `<d:p>`
2. Textbox content NOT spliced mid-sentence
3. Textbox paragraphs emitted as sibling elements after host `<d:p>`

This prevents sentence fragmentation.

### Document Order

Textboxes within `w:txbxContent` extracted and injected into `<write>` in document order. Only text content kept; shape/frame chrome dropped.

## Footnote/Endnote Transformation

### Marker vs Body

- `<fn-ref id="n" type="footnote|endnote"/>` in `<write>` - marker only (empty)
- `<fn id="n" type="footnote|endnote">...</fn>` in `<notes>` - full body

### Type Distinction

Footnotes and endnotes distinguished by `type` attribute:
- `type="footnote"` for footnotes
- `type="endnote"` for endnotes

This prevents ID collision when both exist with same number.

## Metadata Extraction

### Document Metadata (`docProps/core.xml`)

Extract to `<meta>` block:
- `dc:title` → `<title>`
- `dc:creator` → `<author>`
- `dcterms:created` → `<created>` (ISO 8601)
- `dcterms:modified` → `<modified>` (ISO 8601)
- `cp:keywords` → `<keywords>`

All children optional. Omit `<meta>` if source absent or empty.

### Header/Footer Extraction

Extract from `w:hdrReference`/`w:ftrReference`:
- `<header id="n">` per section (matching `<s:page>` order)
- `<footer id="n">` per section
- Content processed through normal transformation rules
- Only presentation chrome dropped, text content preserved

## Language Propagation

Set `lang` on block elements based on precedence:
1. Paragraph-level `w:pPr/w:pStyle/lang` if present
2. Section default `w:lang` if paragraph-level absent
3. First run's `w:rPr/w:rLang` as fallback

If runs within same paragraph have different languages, first run's language takes precedence. Inline language changes lost (`<span>` supports font/size/color/highlight but not `lang`).

## Border Transformation

Paragraph and table borders are preserved via the compact `at` attribute:
- `w:pPr/w:pBdr` → `at="..."` on `<d:p>` or `<d:h>`
- `w:tblPr/w:tblBorders` → `at="..."` on `<d:table>`
- `w:tcPr/w:tcBorders` → `at="..."` on `<d:td>` or `<d:th>`

Border format: `at="[side] [width] [style][space] [color]; ..."`
- Sides: `bt` (top), `bb` (bottom), `bl` (left), `br` (right)
- Styles: `s` (single), `d` (double), `ds` (dashed), `dt` (dotted), `n` (none)
- Width in declared unit (default `in`), converted from twips
- Color as hex with `#` prefix
- Multiple borders separated by `;`

## Bookmark Transformation

- `w:bookmarkStart` → `<bm id="name"/>` in `<notes>` (self-closing)
- `w:bookmarkEnd` → consumed (no output, paired with `bookmarkStart`)
- `id` = `w:name` attribute from `w:bookmarkStart`

## Comment Transformation

- `w:commentRangeStart`/`w:commentRangeEnd` → mark comment span in text
- `w:commentReference` → link to comment body
- Comment body extracted from `word/comments.xml`
- `<comment id="n" author="..." date="...">text</comment>` in `<notes>`
- `id` = 1-based index; `author` from `w:comment/@w:author`; `date` from `w:comment/@w:date`

## Shading (Dropped)

Shading is intentionally dropped as presentation noise:
- `w:pPr/w:shd` → DROP
- `w:rPr/w:shd` → DROP
- `w:tcPr/w:shd` → DROP
- `w:tblPr/w:shd` → DROP
