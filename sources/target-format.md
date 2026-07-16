# DOCX Preprocessor - Target Format (`words`)

This document specifies the **`words`** intermediate markup format - the output of the DOCX preprocessor.

## Overview

`words` is a semantic, versioned XML format designed for:
- Token efficiency (no presentation junk)
- Parser simplicity (flat structure, minimal nesting)
- Downstream processing (AI/LLM training, text mining)

## XML Structure

```xml
<words xmlns="urn:words:v1" version="1.0.1" mode="semantic">
  <meta/>                      # document metadata (optional)
  <style/>                     # layout configuration (optional)
  <header id="n"/>             # header content per section (optional)
  <footer id="n"/>             # footer content per section (optional)
  <write>                      # main document content (required)
    <!-- block elements: h, p, quote, code, table, ul, ol -->
    <!-- inline elements: b, i, u, s, smallcaps, uppercase, sub, sup -->
    <!-- special: a, br, fn-ref, change, img -->
  </write>
  <notes>                      # footnote/endnote bodies (optional)
    <d:fn id="n" type="..."/>
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
- `<d:h>` - headings (Level 1-9)
- `<d:p>` - paragraphs
- `<d:quote>` - blockquotes
- `<d:code>` - code blocks (whitespace preserved)
- `<d:ul>` - unordered lists
- `<d:ol>` - ordered lists
- `<d:table>` - tables (with colspan/rowspan)

### Inline Elements
- `<d:b>` - bold
- `<d:i>` - italic
- `<d:u>` - underline
- `<d:s>` - strikethrough
- `<d:smallcaps>` - small caps
- `<d:uppercase>` - all caps
- `<d:sub>` - subscript
- `<d:sup>` - superscript
- `<d:a>` - hyperlinks
- `<d:br>` - line breaks
- `<d:fn-ref>` - footnote marker
- `<d:change>` - tracked changes (lossless mode only)
- `<d:img>` - image placeholder

## Attributes

### Common Attributes
- `lang="..."` - BCP 47 language tag (block elements)
- `dir="rtl|ltr"` - text direction
- `c="..."` - original style name

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
2. `<style>` (optional)
3. `<header id="n">` (optional, per section)
4. `<footer id="n">` (optional, per section)
5. `<write>` (required)
6. `<notes>` (optional)

## Versioning

Format version: `version="1.0.1"`
Mode: `mode="semantic"` or `mode="lossless"`

Downstream processors MAY branch behavior based on these attributes.
