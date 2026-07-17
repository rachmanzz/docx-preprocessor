# DOCX Preprocessor - Target Format (`words`)

This document specifies the **`words`** intermediate markup format - the output of the DOCX preprocessor.

## Overview

`words` is a semantic, versioned XML format designed for:
- Token efficiency (no presentation junk)
- Parser simplicity (flat structure, minimal nesting)
- Downstream processing (AI/LLM training, text mining)

## XML Structure

```xml
<words xmlns="urn:words:v1" xmlns:s="urn:words:v1:style" version="1.0.1" mode="semantic">
  <meta/>                      # document metadata (optional)
  <style unit="in">            # layout configuration (required)
    <s:page size="A4" mt="0.75" mb="0.75" ml="0.75" mr="0.75" mh="0.5" mf="0.5"/>
  </style>
  <header id="n"/>             # header content per section (optional)
  <footer id="n"/>             # footer content per section (optional)
  <write>                      # main document content (required)
    <!-- block elements: h, p, quote, code, table, ul, ol -->
    <!-- inline elements: b, i, u, s, smallcaps, uppercase, sub, sup, span -->
    <!-- special: a, br, fn-ref, change, img -->
  </write>
  <notes>                      # footnote/endnote bodies, bookmarks, comments (optional)
    <fn id="n" type="..."/>
    <bm id="name"/>
    <comment id="n" author="..." date="...">text</comment>
  </notes>
</words>
```

## Element Types

### Metadata (`<meta>`)
Document-level provenance from `docProps/core.xml`:
- `<title>`, `<author>`, `<created>`, `<modified>`, `<keywords>`

### Layout (`<style>`)
Layout configuration extracted from `w:sectPr`:
- `<s:page>` - page geometry
- `<s:gap>` - spacing rules
- `<s:line>` - line spacing
- `<s:indent>` - paragraph indentation
- `<s:align>` - paragraph alignment
- `<s:cols>` - multi-column layout
- `<s:col>` - table column widths
- `<s:theme>` - color tokens

### Content Blocks (`<write>`)
Semantic content elements:
- `<h1>`-`<h9>` - headings (Level 1-9), supports `at` attribute for borders
- `<p>` - paragraphs, supports `at` and `valign` attributes
- `<blockquote>` - blockquotes
- `<pre>` - code blocks (whitespace preserved)
- `<ul>` - unordered lists
- `<ol>` - ordered lists
- `<table>` - tables (with colspan/rowspan, supports `at`, `width`, `align`, `indent`, `cellSpacing`, `caption`, `summary` attributes)

### Inline Elements (no prefix)
- `<b>` - bold
- `<i>` - italic
- `<u>` - underline
- `<s>` - strikethrough
- `<smallcaps>` - small caps
- `<uppercase>` - all caps
- `<sub>` - subscript
- `<sup>` - superscript
- `<bcs>` - Complex Script bold
- `<ics>` - Complex Script italic
- `<a>` - hyperlinks
- `<br>` - line breaks
- `<fn-ref id="n" type="footnote|endnote">` - footnote/endnote marker (self-closing, type required)
- `<ins>`/`<del>` - tracked changes (lossless mode only)
- `<span>` - font/style span (font, size, color, highlight, lang, hidden, fontEA, fontCS, sizeCS)
- `<img>` - image placeholder

### Notes Elements (in `<notes>`, no prefix)
- `<fn>` - footnote/endnote body
- `<bm>` - bookmark position marker
- `<comment>` - comment text with author/date metadata

## Attributes

### Common Attributes
- `lang="..."` - BCP 47 language tag (block elements and `<span>`)
- `dir="rtl|ltr"` - text direction
- `c="..."` - original style name
- `at="..."` - compact border representation (e.g., `at="bb 12 s1 #000000"`)

### Border Attribute (`at`)
Compact syntax for borders on `<p>`, `<h1>`-`<h9>`, `<td>`, `<th>`, `<table>`:
- Format: `at="[side] [width] [style][space] [color]; ..."`
- Sides: `bt` (top), `bb` (bottom), `bl` (left), `br` (right)
- Styles: `s` (single), `d` (double), `ds` (dashed), `dt` (dotted), `n` (none)
- Multiple borders separated by `;`
- Example: `at="bt 8 d2 #FF0000; bb 4 s1 #000000"`

### Layout Attributes
- `size="A4"` - page preset (A3, A4, A5, Letter, Legal, etc.)
- `mt`, `mb`, `ml`, `mr` - margins
- `mh`, `mf` - header/footer margins
- `before`, `after` - spacing
- `value`, `rule` - line spacing (value = multiplier, rule = auto|exact|atLeast)
- `left`, `right` - indentation

### Table Attributes
- `id="n"` - 1-based table index
- `colspan="n"` - horizontal merge
- `rowspan="n"` - vertical merge
- `width="..."` - table width (in declared unit)
- `align="left|center|right"` - table alignment
- `indent="..."` - table indentation (in declared unit)
- `cellSpacing="..."` - cell spacing (in declared unit)
- `caption="..."` - accessibility caption
- `summary="..."` - accessibility description
- `valign="top|center|bottom"` - cell vertical alignment (on `<td>`/`<th>`)
- `textDir="..."` - text direction in cell (on `<td>`/`<th>`)
- `noWrap="true"` - no-wrap flag (on `<td>`/`<th>`)

### List Attributes
- `type="bullet|decimal|lowerLetter|..."` - list style
- `start="n"` - numbering restart value

## Child Element Ordering

Elements MUST appear in this order:
1. `<meta>` (optional)
2. `<style>` (required — must contain at minimum `<s:page>` with size and margins)
3. `<header id="n">` (optional, per section)
4. `<footer id="n">` (optional, per section)
5. `<write>` (required)
6. `<notes>` (optional)

## Versioning

Format version: `version="1.0.1"`
Mode: `mode="semantic"` or `mode="lossless"`

Downstream processors MAY branch behavior based on these attributes.
