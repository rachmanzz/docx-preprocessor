# DOCX Preprocessor - Transformation Rules

This document defines the transformation rules from DOCX OOXML to `words` XML.

## Classification Categories

| Category | Description | Examples |
|----------|-------------|----------|
| **KEEP** | Mapped to semantic `words` elements | `w:p` → `<p>`, `w:r` → text content |
| **LOSSLESS_METADATA** | Preserved as non-lossy metadata | `w:pPr/w:jc` → `<s:align>`, `w:ins`/`w:del` → `<ins>`/`<del>`, `w:pPr/w:tabs` → `<s:tab>` in `<style>`, `w:tab` → `<tab/>` |
| **DROP** | Presentation noise safely removed | `w:pPr/w:shd`, `w:rPr/w:spacing` |
| **EXCLUDE** | Out of scope / too complex | `w:drawing` (images), `w:object` (OLE), `w:Math` |

## Paragraph Transformation

### Element Mapping

| Source Style | Target Element | Notes |
|--------------|----------------|-------|
| `Heading1`-`Heading9` | `<h1>`-`<h9>` | Level inferred from heading number |
| `Title` | `<h1>` | Title as heading level 0 |
| `ListParagraph` + numPr | `<li>` | Inside `<ul>` or `<ol>` |
| `Quote`/`IntenseQuote`/`BlockText` | `<blockquote>` | MIN-2 requirement |
| `Normal` or none | `<p>` | Default paragraph |
| Code-like styles | `<pre>` | See: §3.1 Code Detection |

### Vertical Text Alignment

- `w:pPr/w:textAlignment` → `<p valign="...">` — P8
- Values: `top`, `center`, `baseline` (maps from OOXML `top`, `center`, `auto`)
- Emitted only when non-default (`auto`/baseline is default, omitted)

### Code Block Detection

A paragraph maps to `<pre>` when:
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
| `w:rPr/w:lang` | `<span lang="..">` (BCP 47 from `@w:val`) — P2 |
| `w:rPr/w:vanish` | `<span hidden="true">` — P9 |
| `w:rPr/w:rFonts/@w:eastAsia` | `<span fontEA="..">` (East Asian font from `@w:eastAsia`) — P14 |
| `w:rPr/w:rFonts/@w:cs` | `<span fontCS="..">` (Complex Script font from `@w:cs`) — P15 |
| `w:rPr/w:bCs` | `<bcs>...</bcs>` — P16 |
| `w:rPr/w:iCs` | `<ics>...</ics>` — P17 |
| `w:rPr/w:szCs` | `<span sizeCS="..">` (Complex Script font size in pt, `w:val ÷ 2`) — P18 |

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
- Split list into new `<ol>` with `start="n"` attribute
- Default `start="1"` if not specified

### List Types

| numFmt | Target Type |
|--------|-------------|
| `bullet` | `<ul type="bullet">` |
| `decimal` | `<ol type="decimal">` |
| `lowerLetter`/`upperLetter` | `<ol type="lowerLetter|upperLetter">` |
| `lowerRoman`/`upperRoman` | `<ol type="lowerRoman|upperRoman">` |
| Any other | `<ol type="...">` - preserve raw value (MOD-2) |

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
- `<s:col ref="n">` matches `<table id="n">`

### Table Properties

- `w:tblPr/w:tblW` → `<table width="...">` — P5 (converted to declared unit)
- `w:tblPr/w:jc` → `<table align="...">` — P6 (left|center|right)
- `w:tblPr/w:tblInd` → `<table indent="...">` — P10 (converted to declared unit)
- `w:tblPr/w:tblCellSpacing` → `<table cellSpacing="...">` — P11 (converted to declared unit)
- `w:tblPr/w:tblCaption` → `<table caption="...">` — P3
- `w:tblPr/w:tblDescription` → `<table summary="...">` — P4
- `w:tcPr/w:vAlign` → `<td valign="...">` / `<th valign="...">` — P7 (top|center|bottom)
- `w:tcPr/w:textDirection` → `<td textDir="...">` / `<th textDir="...">` — P12 (bttRtl|tbRl|tbRlV|rtLmV|ltRtV)
- `w:tcPr/w:noWrap` → `<td noWrap="true">` / `<th noWrap="true">` — P13

## Textbox Transformation

### Inline Anchor Handling

When textbox anchored inside `<w:r>`:
1. Host paragraph runs before/after anchor merged into single `<p>`
2. Textbox content NOT spliced mid-sentence
3. Textbox paragraphs emitted as sibling elements after host `<p>`

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

If runs within same paragraph have different languages, first run's language takes precedence. Inline language changes now captured via `<span lang="...">` (P2).

## Line Spacing (LOSSLESS_METADATA)

- `w:pPr/w:spacing/@w:line` → `<s:line>` — P1
- `w:pPr/w:spacing/@w:lineRule` → `rule` attribute on `<s:line>`
- Values: `auto` (line height multiplier), `exact` (fixed), `atLeast` (minimum)
- `<s:line>` keyed by `el` + optional `c` (style name)
- Only emitted when line spacing is non-default (single-spaced is default)

## Border Transformation

Paragraph and table borders are preserved via the compact `at` attribute:
- `w:pPr/w:pBdr` → `at="..."` on `<p>` or `<h1>`-`<h9>`
- `w:tblPr/w:tblBorders` → `at="..."` on `<table>`
- `w:tcPr/w:tcBorders` → `at="..."` on `<td>` or `<th>`

Border format: `at="[side] [width] [style][space] [color]; ..."`
- Sides: `bt` (top), `bb` (bottom), `bl` (left), `br` (right)
- Styles: `s` (single), `d` (double), `ds` (dashed), `dt` (dotted), `n` (none)
- Width in declared unit (default `in`), converted from twips
- Color as hex with `#` prefix
- Multiple borders separated by `;`

## Section Properties

- `w:sectPr/w:cols` → `<s:cols n=".." space=".."/>` in `<style>` — P19 (multi-column layout)
- `w:sectPr/w:cols/@w:num` → `n` attribute (number of columns)
- `w:sectPr/w:cols/@w:space` → `space` attribute (column spacing, converted to declared unit)
- Only emitted when column count > 1

## Style Transformation

### Custom Style Extraction

When a paragraph or table has a custom style (`w:pPr/w:pStyle` or `w:tblPr/w:tblStyle`):

1. Look up style in `styles.xml`
2. Resolve inheritance chain (`w:basedOn`) to find semantic element
3. Emit `<s:custom>` in `<style>` block with formatting properties
4. Add `c="styleName"` attribute to element (only when style name ≠ element name)

### `<s:custom>` Mapping

- `w:style/@w:styleId` → `name` attribute
- `w:style/w:basedOn/@w:val` → `basedOn` attribute
- `w:style/@w:type` → `type` attribute (paragraph|character|table)
- `w:style/w:rPr/*` → run properties (font, size, color, bold, etc.)
- `w:style/w:pPr/*` → paragraph properties (alignment, spacing, indentation)
- `w:style/w:tblPr/*` → table properties (borders, width)

### Standard vs Custom Styles

| Style Type | Example | Output | `c` emitted? |
|---|---|---|---|
| Standard | Heading1 | `<h1>` | ❌ No |
| Standard | Normal | `<p>` | ❌ No |
| Custom | MyHeading (basedOn Heading1) | `<h1 c="MyHeading">` | ✅ Yes |
| Custom | CustomPara (basedOn Normal) | `<p c="CustomPara">` | ✅ Yes |
| Custom | TableGrid | `<table c="TableGrid">` | ✅ Yes |

## Tab Stop Transformation

### Tab Stops (`w:pPr/w:tabs`)

```
w:pPr/w:tabs/w:tab/@w:val    → <s:tab> align attribute
w:pPr/w:tabs/w:tab/@w:pos    → <s:tab> pos attribute (÷ twips if needed)
w:pPr/w:tabs/w:tab/@w:leader → <s:tab> leader attribute
w:pPr/@w:pStyle               → <s:tab> el attribute (maps style to element)
```

| `w:val` | `align` | `leader` |
|---------|---------|----------|
| `left` | `left` | — |
| `center` | `center` | — |
| `right` | `right` | — |
| `decimal` | `decimal` | — |
| `num` | `decimal` | — |
| `clear` | `left` | — |
| `dot` | — | `dot` |
| `hyphen` | — | `dash` |
| `underscore` | — | `underscore` |
| `heavy` | — | `bar` |

### Tab Character (`w:tab`)

- `w:tab` → `<tab/>` (self-closing inline element)
- Exception: inside `<pre>` block, `w:tab` → literal tab character (`\t`)

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
