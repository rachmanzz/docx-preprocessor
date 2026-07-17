# DOCX Preprocessor - Target Format (`words`)

This document specifies the **`words`** intermediate markup format - the output of the DOCX preprocessor.

## Overview

`words` is a semantic, versioned XML format designed for:
- Token efficiency (no presentation junk)
- Parser simplicity (flat structure, minimal nesting)
- Downstream processing (AI/LLM training, text mining)

## XML Structure

```xml
<words xmlns="urn:words:v1" xmlns:d="urn:words:v1:doc" xmlns:s="urn:words:v1:style" version="1.0.1" mode="semantic">
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
- `<s:indent>` - paragraph indentation
- `<s:align>` - paragraph alignment
- `<s:col>` - table column widths
- `<s:theme>` - color tokens

### Content Blocks (`<write>`)
Semantic content elements:
- `<d:h>` - headings (Level 1-9), supports `at` attribute for borders
- `<d:p>` - paragraphs, supports `at` attribute for borders
- `<d:quote>` - blockquotes
- `<d:code>` - code blocks (whitespace preserved)
- `<d:ul>` - unordered lists
- `<d:ol>` - ordered lists
- `<d:table>` - tables (with colspan/rowspan, supports `at` attribute for borders)

### Inline Elements (no prefix)
- `<b>` - bold
- `<i>` - italic
- `<u>` - underline
- `<s>` - strikethrough
- `<smallcaps>` - small caps
- `<uppercase>` - all caps
- `<sub>` - subscript
- `<sup>` - superscript
- `<a>` - hyperlinks
- `<br>` - line breaks
- `<fn-ref id="n" type="footnote|endnote">` - footnote/endnote marker (self-closing, type required)
- `<change>` - tracked changes (lossless mode only)
- `<span>` - font/style span (font, size, color, highlight)
- `<img>` - image placeholder

### Notes Elements (in `<notes>`, no prefix)
- `<fn>` - footnote/endnote body
- `<bm>` - bookmark position marker
- `<comment>` - comment text with author/date metadata

## Attributes

### Common Attributes
- `lang="..."` - BCP 47 language tag (block elements)
- `dir="rtl|ltr"` - text direction
- `c="..."` - original style name
- `at="..."` - compact border representation (e.g., `at="bb 12 s1 #000000"`)

### Border Attribute (`at`)
Compact syntax for borders on `<d:p>`, `<d:h>`, `<d:td>`, `<d:th>`, `<d:table>`:
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
- `left`, `right` - indentation

### Table Attributes
- `id="n"` - 1-based table index
- `colspan="n"` - horizontal merge
- `rowspan="n"` - vertical merge

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
