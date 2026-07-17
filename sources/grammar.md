# DOCX Preprocessor - Grammar Reference

This document defines the complete `words` XML grammar for the DOCX Preprocessor.

## Root Element

```xml
<words 
  xmlns="urn:words:v1"
  xmlns:d="urn:words:v1:doc"
  xmlns:s="urn:words:v1:style"
  version="1.0.1" 
  mode="semantic">
```

### Attributes
- `xmlns="urn:words:v1"` - default namespace for structural elements (REQUIRED)
- `xmlns:d="urn:words:v1:doc"` - namespace for block-level document content elements (REQUIRED)
- `xmlns:s="urn:words:v1:style"` - namespace for style/layout elements (REQUIRED)
- Inline elements (`<b>`, `<i>`, `<u>`, `<s>`, `<span>`, `<a>`, `<br>`, etc.) use **no prefix**
- `version="1.0.1"` - format version (REQUIRED)
- `mode="semantic|lossless"` - processing mode (REQUIRED)

## Structure Hierarchy

```
<words>
  <meta>?                          # document metadata
  <style>                          # layout configuration (required)
  <header id="n">*                 # header content (per section)
  <footer id="n">*                 # footer content (per section)
  <write>                          # main content
    <!-- block elements -->
    <!-- inline elements -->
  </write>
  <notes>?                         # footnote/endnote bodies
```

## Block Elements

### `<d:h>` - Heading
```xml
<d:h c="Heading1" lang="en">...</d:h>
<d:h c="Heading1" at="bb 12 s1 #000000" lang="en">...</d:h>
```
Attributes:
- `c="..."` - original style name (REQUIRED)
- `at="..."` - compact border representation (see Border Attribute section)
- `lang="..."` - BCP 47 language tag

### `<d:p>` - Paragraph
```xml
<d:p lang="en">...</d:p>
<d:p at="bb 12 s1 #000000" lang="en">...</d:p>
```
Attributes:
- `at="..."` - compact border representation (see Border Attribute section)
- `lang="..."` - BCP 47 language tag

### `<d:quote>` - Blockquote
```xml
<d:quote lang="en">...</d:quote>
```
Attributes:
- `lang="..."` - BCP 47 language tag

### `<d:code>` - Code Block
```xml
<d:code>...</d:code>
```
Whitespace preserved verbatim (see: §3.5 Text cleanup)

### `<d:ul>` - Unordered List
```xml
<d:ul type="bullet">
  <d:li>...</d:li>
</d:ul>
```
Attributes:
- `type="bullet|lowerRoman|upperRoman|..."` - list style

### `<d:ol>` - Ordered List
```xml
<d:ol type="decimal" start="1">
  <d:li>...</d:li>
</d:ol>
```
Attributes:
- `type="decimal|lowerLetter|upperLetter|..."` - list style
- `start="n"` - numbering restart value (optional)

### `<d:table>` - Table
```xml
<d:table id="1" at="bb 4 s1 #000000">
  <d:tr><d:th colspan="2" lang="en">...</d:th></d:tr>
  <d:tr><d:td lang="en" at="bb 4 s1 #000000">...</d:td></d:tr>
</d:table>
```
Attributes:
- `id="n"` - 1-based table index (REQUIRED)
- `at="..."` - compact border representation (see Border Attribute section)
- Child elements: `<d:tr>` containing `<d:th>` or `<d:td>` (both support `at`)

## Inline Elements (no prefix)

### Text Styling
- `<b>` - bold
- `<i>` - italic
- `<u>` - underline
- `<s>` - strikethrough
- `<smallcaps>` - small caps
- `<uppercase>` - all caps
- `<sub>` - subscript
- `<sup>` - superscript

### Font/Style Span
```xml
<span font="DejaVu Sans" size="12" color="FF0000" highlight="yellow">...</span>
```
Attributes (all optional, at least one required):
- `font="..."` - font family name
- `size="..."` - font size in pt
- `color="..."` - text color (hex)
- `highlight="..."` - highlight color name

### Special
- `<a href="...">` - hyperlink
- `<br type="textWrapping|page|column|clear">` - line break
- `<fn-ref id="n" type="footnote|endnote"/>` - footnote marker (self-closing)
- `<change type="insert|delete">...</change>` - tracked change

## Layout Elements

### Border Attribute (`at`)
Compact syntax for borders on `<d:p>`, `<d:h>`, `<td>`, `<th>`, `<table>`:
```xml
<d:p at="bb 12 s1 #000000"/>
<d:p at="bt 8 d2 #FF0000; bb 4 s1 #000000"/>
```
Format: `at="[side] [width] [style][space] [color]; ..."`
- Sides: `bt` (top), `bb` (bottom), `bl` (left), `br` (right)
- Styles: `s` (single), `d` (double), `ds` (dashed), `dt` (dotted), `n` (none)
- Space value appended to style code (e.g., `s1` = single, space 1)
- Color: hex with `#` prefix (e.g., `#000000`)
- Multiple borders separated by `;`

### `<meta>` - Metadata
```xml
<meta>
  <title>...</title>
  <author>...</author>
  <created>...</created>
  <modified>...</modified>
  <keywords>...</keywords>
</meta>
```

### `<style>` - Layout Configuration
```xml
<style unit="in">
  <s:page size="A4" mt="0.75" mb="0.75" ml="0.75" mr="0.75" mh="0.5" mf="0.5"/>
  <s:gap el="d:p" before="0" after="0.11"/>
  <s:indent el="d:p" left="0" right="0" firstLine="0" hanging="0"/>
  <s:align el="d:p" value="left|center|right|both"/>
  <s:col ref="1" w="1.39"/>
  <s:theme bg="FFFFFF" fg="000000"/>
</style>
```

### `<header id="n">` / `<footer id="n">` - Header/Footer
```xml
<header id="1">
  <d:p>...</d:p>
</header>
```

### `<notes>` - Notes Container
```xml
<notes>
  <fn id="1" type="footnote">...</fn>
  <bm id="bookmark1"/>
  <comment id="1" author="Author Name" date="2024-01-15T10:30:00Z">Comment text.</comment>
</notes>
```

## Attribute Ordering

Attributes MUST be emitted in canonical order:
1. `id`
2. `lang`
3. `type`
4. `c`
5. `at`
6. `start`
7. `ref`
8. `alt`
9. `href`
10. `title`
11. `value`
12. `font`
13. `size`
14. `color`
15. `highlight`
16. `w`, `h`
17. `mt`, `mb`, `ml`, `mr`
18. `mh`, `mf`
19. `el`, `before`, `after`
20. `left`, `right`, `firstLine`, `hanging`
21. `colspan`, `rowspan`
22. `dir`
23. `mode`

## Empty Element Syntax

Empty elements MUST use self-closing syntax:
- `<img alt="..."/>` ✓
- `<br/>` ✓
- `<fn-ref id="n" type="footnote"/>` ✓

NOT: `<img></img>`
