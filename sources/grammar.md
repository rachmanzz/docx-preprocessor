# DOCX Preprocessor - Grammar Reference

This document defines the complete `words` XML grammar for the DOCX Preprocessor.

## Root Element

```xml
<words 
  xmlns="urn:words:v1" 
  version="1.0.1" 
  mode="semantic">
```

### Attributes
- `xmlns="urn:words:v1"` - namespace (REQUIRED)
- `version="1.0.1"` - format version (REQUIRED)
- `mode="semantic|lossless"` - processing mode (REQUIRED)

## Structure Hierarchy

```
<words>
  <meta>?                          # document metadata
  <style>?                         # layout configuration
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
```
Attributes:
- `c="..."` - original style name (REQUIRED)
- `lang="..."` - BCP 47 language tag

### `<d:p>` - Paragraph
```xml
<d:p lang="en">...</d:p>
```
Attributes:
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
<d:table id="1">
  <d:tr><d:th colspan="2" lang="en">...</d:th></d:tr>
  <d:tr><d:td lang="en">...</d:td></d:tr>
</d:table>
```
Attributes:
- `id="n"` - 1-based table index (REQUIRED)
- Child elements: `<d:tr>` containing `<d:th>` or `<d:td>`

## Inline Elements

### Text Styling
- `<d:b>` - bold
- `<d:i>` - italic
- `<d:u>` - underline
- `<d:s>` - strikethrough
- `<d:smallcaps>` - small caps
- `<d:uppercase>` - all caps
- `<d:sub>` - subscript
- `<d:sup>` - superscript

### Special
- `<d:a href="...">` - hyperlink
- `<d:br type="textWrapping|page|column|clear">` - line break
- `<d:fn-ref id="n" type="footnote|endnote"/>` - footnote marker (self-closing)
- `<d:change type="insert|delete">...</d:change>` - tracked change

## Layout Elements

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
<style unit="pt">
  <s:page size="A4" w="595" h="842" mt="54" mb="54" ml="54" mr="54" mh="36" mf="36"/>
  <s:gap el="d:p" before="0" after="8"/>
  <s:indent el="d:p" left="0" right="0" firstLine="0" hanging="0"/>
  <s:align el="d:p" value="left|center|right|both"/>
  <s:col ref="1" w="100"/>
  <s:theme bg="FFFFFF" fg="000000"/>
</style>
```

### `<header id="n">` / `<footer id="n">` - Header/Footer
```xml
<header id="1">
  <d:p>...</d:p>
</header>
```

### `<notes>` - Footnote/Endnote Container
```xml
<notes>
  <d:fn id="1" type="footnote">...</d:fn>
</notes>
```

## Attribute Ordering

Attributes MUST be emitted in canonical order:
1. `id`
2. `lang`
3. `type`
4. `c`
5. `start`
6. `ref`
7. `alt`
8. `href`
9. `title`
10. `value`
11. `size`
12. `w`, `h`
13. `mt`, `mb`, `ml`, `mr`
14. `mh`, `mf`
15. `el`, `before`, `after`
16. `left`, `right`, `firstLine`, `hanging`
17. `colspan`, `rowspan`
18. `dir`
19. `mode`

## Empty Element Syntax

Empty elements MUST use self-closing syntax:
- `<d:img alt="..."/>` ✓
- `<d:br/>` ✓
- `<d:fn-ref id="n"/>` ✓

NOT: `<d:img></d:img>`
