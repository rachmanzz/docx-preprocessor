# DOCX Preprocessor - Determinism & Conformance

This document specifies determinism requirements and conformance criteria for the DOCX Preprocessor.

## Determinism Requirements

### Output Consistency

The preprocessor MUST produce **identical XML** for identical DOCX input:

```text
DOCX A → words XML X
DOCX A → words XML X  (identical)
```

### Output Ordering Constraints

Output ordering MUST NOT depend on:
- Map iteration order (use sorted iteration)
- Filesystem order (sort filenames before processing)
- Relationship order (sort by relationship ID)
- XML parser implementation (use deterministic parsing)
- Thread scheduling (single-threaded or synchronized)
- Memory address allocation
- Hash table bucket ordering

### Atomic Operations

Each transformation is atomic:
- Parse input → build internal AST
- Transform AST → build output tree
- Serialize output tree → XML string
- No side effects between operations

## Deterministic Processing Pipeline

```
1. Open .docx (zip archive)
2. Read [content_types].xml (deterministic parse)
3. Read word/document.xml (deterministic parse)
4. Read supporting XML files (styles.xml, numbering.xml, etc.)
5. Build internal AST
6. Apply transformation rules (deterministic)
7. Build output tree (deterministic structure)
8. Serialize to XML (deterministic formatting)
```

## Output Determinism Requirements

### XML Declaration

Output MUST include:
```xml
<?xml version="1.0" encoding="UTF-8"?>
```

### Namespace Declarations

The root `<words>` element MUST declare two namespaces:
```xml
<words xmlns="urn:words:v1" xmlns:s="urn:words:v1:style" version="1.0.1" mode="semantic">
```

- `xmlns="urn:words:v1"` — default namespace for all elements
- `xmlns:s="urn:words:v1:style"` — prefix for style/layout elements

### Character Encoding

Output MUST be UTF-8 encoded. All text normalized to UTF-8 regardless of source encoding.

### Line Endings

Output MUST use LF (Unix-style line endings) consistently. CR and CRLF sequences converted to LF.

### Indentation

Indentation MUST be exactly 2 spaces per level. XML attributes indented on new lines.

### Empty Element Syntax

Empty elements MUST use self-closing syntax:
- `<img alt="..."/>` ✓
- `<br/>` ✓
- `<fn-ref id="n" type="footnote"/>` ✓

NOT: `<img></img>` ✗

### Whitespace in Text Content

- `semantic` mode: Collapse repeated spaces, normalize whitespace
- `lossless` mode: Preserve original whitespace
- `xml:space="preserve"`: Always honored regardless of mode

## Conformance Testing

### Test Structure

```
tests/
  ├── 001-basic.docx          → 001-basic.words.xml
  ├── 002-table.docx          → 002-table.words.xml
  ├── 003-list.docx           → 003-list.words.xml
  ├── 004-nested.docx         → 004-nested.words.xml
  └── manifest.json           # Test cases metadata
```

### Test Cases

Each test case should cover:
- Basic paragraph transformation
- Heading structure
- List handling
- Table with merge
- Nested list
- Footnote/endnote
- Textbox extraction
- Code block detection
- Tracked changes (lossless mode)
- Multiple sections

### Assertion Criteria

Test assertions MUST verify:
1. XML is well-formed (valid XML 1.0)
2. All required elements present
3. Element ordering correct
4. Attributes in canonical order
5. Text content matches expected
6. No unintended whitespace
7. No XML escape issues
8. `at` attribute border format valid

### Diff Tolerance

Tests MAY allow:
- Attribute order differences (normalized)
- Whitespace differences in text content (mode-dependent)
- Empty attribute omission (when value matches default)

Tests MUST NOT allow:
- Element ordering differences
- Missing required elements
- Different text content
- Different ID assignments
- Different rowspan/colspan values

## Implementation Checklist

Implementations MUST:

- [ ] Use deterministic XML parsing (sorted node order)
- [ ] Sort files by name before processing
- [ ] Sort relationships by ID
- [ ] Use sorted iteration for collections
- [ ] Produce consistent indentation (2 spaces)
- [ ] Use LF line endings
- [ ] Include XML declaration
- [ ] Use self-closing syntax for empty elements
- [ ] Normalize encoding to UTF-8
- [ ] Respect `xml:space="preserve"`
- [ ] Sort attributes in canonical order (including `at`)
- [ ] Use pre-order traversal for table IDs
- [ ] Produce idempotent output (same input → same output)
- [ ] Convert border widths from twips to declared unit
- [ ] Preserve bookmarks in `<notes>` as `<bm>`
- [ ] Preserve comments in `<notes>` as `<comment>`

## Version Compatibility

The preprocessor MUST support DOCX from:
- Word 2007 (Office Open XML)
- Word 2010
- Word 2013
- Word 2016
- Word 2019
- Word 365

Format version `1.0.1` is compatible with all these versions.
