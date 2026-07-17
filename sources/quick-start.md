# DOCX Preprocessor - Installation & Quick Start

## Requirements

- Node.js 16+ (for JavaScript implementation)
- Rust 1.60+ (for Rust implementation)
- Python 3.9+ (for Python implementation)

## Installation

### Node.js

```bash
npm install @docx-preprocessor/core
```

### Rust

```bash
cargo add docx-preprocessor
```

### Python (planned)

```bash
pip install docx-preprocessor
```

## Quick Start

### Basic Usage

```javascript
const { processDocx } = require('@docx-preprocessor/core');

const input = fs.readFileSync('document.docx');
const output = processDocx(input);

console.log(output); // words XML as string
```

### With Options

```javascript
const output = processDocx(input, {
  mode: 'lossless',  // or 'semantic' (default)
  preserveWhitespace: true,
  includeMetadata: true
});
```

### CLI Usage

```bash
# Basic
docx-preprocess input.docx -o output.words.xml

# With mode
docx-preprocess input.docx -m lossless -o output.words.xml

# With metadata
docx-preprocess input.docx --with-metadata -o output.words.xml
```

## Example Output

```xml
<?xml version="1.0" encoding="UTF-8"?>
<words xmlns="urn:words:v1" xmlns:d="urn:words:v1:doc" xmlns:s="urn:words:v1:style" version="1.0.1" mode="semantic">
  <meta>
    <title>Document Title</title>
    <author>John Doe</author>
  </meta>
  <style unit="in">
    <s:page size="A4" mt="0.75" mb="0.75" ml="0.75" mr="0.75" mh="0.5" mf="0.5"/>
  </style>
  <write>
    <d:h c="Heading1" at="bb 12 s1 #000000">Introduction</d:h>
    <d:p lang="en">This is a <b>bold</b> and <i>italic</i> paragraph.</d:p>
  </write>
  <notes>
    <bm id="Section1"/>
    <comment id="1" author="Reviewer" date="2024-01-15T10:30:00Z">Good start.</comment>
  </notes>
</words>
```

## Error Handling

```javascript
try {
  const output = processDocx(input);
  console.log(output);
} catch (error) {
  if (error.code === 'INVALID_DOCX') {
    console.error('Invalid DOCX file');
  } else if (error.code === 'MISSING_PART') {
    console.error('Missing document.xml');
  }
  console.error(error.message);
}
```

## Supported DOCX Features

| Feature | Status |
|---------|--------|
| Paragraphs | ✓ |
| Headings | ✓ |
| Lists | ✓ |
| Tables | ✓ |
| Images | ✗ (placeholder only) |
| Footnotes | ✓ |
| Hyperlinks | ✓ |
| Textboxes | ✓ |
| Code blocks | ✓ |
| Table merge | ✓ |
| Borders | ✓ (compact `at` attribute) |
| Bookmarks | ✓ (in `<notes>`) |
| Comments | ✓ (in `<notes>`) |
| Bold/Italic/Underline | ✓ |
| Font/Size/Color | ✓ (via `<span>`) |

## Limitations

- No DOCX regeneration
- No pixel-perfect layout preservation (layout metadata preserved, not rendering)
- No image content extraction
- Shading intentionally dropped (borders preserved)

## Support

- Documentation: See `sources/` directory
- Issues: https://github.com/rachmanzz/docx-preprocessor/issues
- License: MIT
