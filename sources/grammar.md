# DOCX Preprocessor - Grammar Reference

This document defines the complete `words` XML grammar for the DOCX Preprocessor.

## Root Element

```xml
<words 
  xmlns="urn:words:v1"
  xmlns:s="urn:words:v1:style"
  version="1.0.1" 
  mode="semantic">
```

### Attributes
- `xmlns="urn:words:v1"` - default namespace for all elements (REQUIRED)
- `xmlns:s="urn:words:v1:style"` - namespace for style/layout elements (REQUIRED)
- Inline elements (`<b>`, `<i>`, `<u>`, `<s>`, `<span>`, `<a>`, `<br>`, etc.) use **no prefix**
- Block elements (`<p>`, `<h1>`-`<h9>`, `<ul>`, `<ol>`, `<table>`, etc.) also use **no prefix**
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

### `<h1>`-`<h9>` - Headings
```xml
<h1 lang="en">...</h1>
<h1 at="bb 12 s1 #000000" lang="en">...</h1>
```
Attributes:
- `c="..."` - original style name (only emitted when style name ≠ element name, e.g., custom styles)
- `at="..."` - compact border representation (see Border Attribute section)
- `lang="..."` - BCP 47 language tag

### `<p>` - Paragraph
```xml
<p lang="en">...</p>
<p at="bb 12 s1 #000000" lang="en">...</p>
<p lang="en" valign="top">...</p>
```
Attributes:
- `at="..."` - compact border representation (see Border Attribute section)
- `lang="..."` - BCP 47 language tag
- `valign="top|center|baseline"` - vertical text alignment

### `<blockquote>` - Blockquote
```xml
<blockquote lang="en">...</blockquote>
```
Attributes:
- `lang="..."` - BCP 47 language tag

### `<pre>` - Code Block
```xml
<pre>...</pre>
```
Whitespace preserved verbatim (see: §3.5 Text cleanup)

### `<ul>` - Unordered List
```xml
<ul type="bullet">
  <li>...</li>
</ul>
```
Attributes:
- `type="bullet|lowerRoman|upperRoman|..."` - list style

### `<ol>` - Ordered List
```xml
<ol type="decimal" start="1">
  <li>...</li>
</ol>
```
Attributes:
- `type="decimal|lowerLetter|upperLetter|..."` - list style
- `start="n"` - numbering restart value (optional)

### `<table>` - Table
```xml
<table id="1" at="bb 4 s1 #000000" width="8" align="center" indent="0.5" cellSpacing="0.1" caption="Table 1" summary="Data summary">
  <tr><th colspan="2" lang="en" valign="top" textDir="ltr" noWrap="true">...</th></tr>
  <tr><td lang="en" at="bb 4 s1 #000000" valign="center" textDir="ltr" noWrap="true">...</td></tr>
</table>
```
Attributes:
- `id="n"` - 1-based table index (REQUIRED)
- `at="..."` - compact border representation (see Border Attribute section)
- `width="..."` - table width (in declared unit)
- `align="left|center|right"` - table alignment
- `indent="..."` - table indentation (in declared unit) — P10
- `cellSpacing="..."` - cell spacing (in declared unit) — P11
- `caption="..."` - accessibility caption
- `summary="..."` - accessibility description
- Child elements: `<tr>` containing `<th>` or `<td>` (both support `at`, `valign`, `textDir`, `noWrap`)

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
- `<bcs>` - Complex Script bold — P16
- `<ics>` - Complex Script italic — P17
- `<tab/>` - tab character (self-closing)

### Font/Style Span
```xml
<span font="DejaVu Sans" size="12" color="FF0000" highlight="yellow" lang="en" hidden="true" fontEA="MS Mincho" fontCS="Arial" sizeCS="11">...</span>
```
Attributes (all optional, at least one required):
- `font="..."` - font family name
- `size="..."` - font size in pt
- `color="..."` - text color (hex)
- `highlight="..."` - highlight color name
- `lang="..."` - BCP 47 language tag (run-level)
- `hidden="true"` - hidden text
- `fontEA="..."` - East Asian font family — P14
- `fontCS="..."` - Complex Script font family — P15
- `sizeCS="..."` - Complex Script font size in pt — P18

### Special
- `<a href="...">` - hyperlink
- `<br type="textWrapping|page|column|clear">` - line break
- `<fn-ref id="n" type="footnote|endnote"/>` - footnote marker (self-closing)
- `<ins>...</ins>` / `<del>...</del>` - tracked change

## Layout Elements

### Border Attribute (`at`)
Compact syntax for borders on `<p>`, `<h1>`-`<h9>`, `<td>`, `<th>`, `<table>`:
```xml
<p at="bb 12 s1 #000000"/>
<p at="bt 8 d2 #FF0000; bb 4 s1 #000000"/>
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
  <s:gap el="p" before="0" after="0.11"/>
  <s:indent el="p" left="0" right="0" firstLine="0" hanging="0"/>
  <s:align el="p" value="left|center|right|both"/>
  <s:cols n="2" space="0.5"/>
  <s:col ref="1" w="1.39"/>
  <s:theme font="Calibri" fontEA="MS Mincho" fontCS="Arial" bg="FFFFFF" fg="000000"/>
  <s:custom name="MyHeading" basedOn="Heading1" type="paragraph"
            font="Arial" size="16" color="FF0000" bold="true"/>
</style>
```

### `<s:theme>` - Global Defaults

```xml
<s:theme font="Calibri" fontEA="MS Mincho" fontCS="Arial" bg="FFFFFF" fg="000000"/>
```
Attributes (all optional):
- `font="..."` - global default font family (from docDefaults/theme fontScheme)
- `fontEA="..."` - global default East Asian font family
- `fontCS="..."` - global default Complex Script font family
- `bg="..."` - background color hex
- `fg="..."` - foreground color hex

### `<s:cols>` - Multi-Column Layout
```xml
<s:cols n="2" space="0.5"/>
```
Attributes:
- `n="n"` - number of columns — P19
- `space="..."` - column spacing (in declared unit) — P19

### `<s:tab>` - Tab Stop Definition
```xml
<s:tab el="p" pos="1.0" align="left" leader="none"/>
<s:tab el="p" pos="2.0" align="center" leader="dot"/>
<s:tab el="h1" pos="3.0" align="right" leader="dash"/>
```
Attributes:
- `el="p|h1|h2|..."` - element type this tab stop applies to
- `pos="..."` - tab stop position (in declared unit)
- `align="left|center|right|decimal"` - tab alignment
- `leader="none|dot|dash|underscore|bar"` - leader character

### `<s:custom>` - Custom Style Definition
```xml
<s:custom name="MyHeading" basedOn="Heading1" type="paragraph"
          font="Arial" fontEA="MS Mincho" fontCS="Arial"
          size="16" sizeCS="14" color="FF0000"
          bold="true" italic="true" underline="single"
          strikethrough="true" smallCaps="true" uppercase="true"
          alignment="center" spacingBefore="0.17" spacingAfter="0.08"
          lineSpacing="1.5" lineRule="auto"
          indentLeft="0.5" indentRight="0.5" indentFirst="0.25" indentHanging="0.25"
          borderWidth="0.5" borderColor="000000" borderStyle="single"
          cellSpacing="0.1" width="8"/>
```
Attributes (all optional except `name`):
- `name="..."` - style name (REQUIRED)
- `basedOn="..."` - parent style name (optional)
- `type="paragraph|character|table"` - style type (optional)
- Run properties: `font`, `fontEA`, `fontCS`, `size`, `sizeCS`, `color`, `bold`, `italic`, `underline`, `strikethrough`, `smallCaps`, `uppercase`
- Paragraph properties: `alignment`, `spacingBefore`, `spacingAfter`, `lineSpacing`, `lineRule`, `indentLeft`, `indentRight`, `indentFirst`, `indentHanging`
- Table properties: `borderWidth`, `borderColor`, `borderStyle`, `cellSpacing`, `width`

### `<header id="n">` / `<footer id="n">` - Header/Footer
```xml
<header id="1">
  <p>...</p>
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
