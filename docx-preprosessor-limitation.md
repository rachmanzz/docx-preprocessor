# DOCX Preprocessor — Limitations

This document consolidates the **limitations and exclusions** of the DOCX Preprocessor
(`docx-preprosessor.md`, `words` v1.0.1). It is the authoritative reference for what the
preprocessor intentionally does **not** handle.

The preprocessor is a **deterministic, LLM-free, noise-removing transformer**. Its job is to
emit a semantic, token-efficient `words` document — not to faithfully reproduce every
binary or presentation detail of the source `.docx`.

---

## 1. Excluded by Policy (binary / too complex)

These are **out of scope** for v1.0.1. They are emitted as placeholders or dropped entirely.

| Excluded construct | OOXML source | Behavior in `words` | Impact |
|--------------------|--------------|--------------------|--------|
| **Images (non-textbox)** | `w:drawing` image blip, `w:pict` (VML) | `<img alt="..."/>` placeholder; no pixel/vector data | Visual content (photos, diagrams, scanned specs) lost |
| **OLE objects** | `w:object` | dropped | Embedded spreadsheets/Visio lost |
| **Charts** | chart drawing parts | dropped | Data visualizations lost |
| **SmartArt / diagrams** | smartart drawing parts | dropped | Flowcharts/org-charts lost |
| **Office Math** | `w:Math` (OMML) | dropped (not converted to LaTeX/MathML) | Equations absent |
| **External chunks** | `w:altChunk` | dropped | Embedded HTML lost |
| ~~**Headers/footers content**~~ | ~~`w:hdrReference`/`w:ftrReference`~~ | ~~dropped~~ | **RESOLVED**: kept in `<header>`/`<footer>` blocks (see spec §2.6) |

> Rationale: these require specialized renderers (image decoding, OLE/Visio parsers, math
> layout engines) and/or carry binary payloads. Converting them deterministically is
> out of scope and would bloat the token budget.

> **Textboxes are NOT excluded** (resolved, CRIT-1): `w:txbxContent` text is extracted
> into `<write>` by the normal rules. Only the image *inside* a textbox remains excluded.

**Workarounds**
- Images: describe manually, or run a separate OCR/Vision pass and inject captions before curation.
- Math: convert OMML → LaTeX in a dedicated preprocessing step if equations are required.
- OLE/charts: extract source data (e.g., the embedded workbook) and re-represent as text/table.

---

## 2. Dropped as Noise (presentation / revision)

Removed on purpose to keep the semantic body clean. They do **not** appear in `words`.

- **Paragraph presentation**: shading (`w:shd`), tabs (`w:tabs`),
  frame (`w:framePr`), `pageBreakBefore`, `keepNext`,
  `widowControl`, `outlineLvl`.
  Note: `w:jc` (justification) is **preserved** as `<s:align>` in `<style>` (LOSSLESS_METADATA).
  Note: `w:pBdr` (borders) is **preserved** via compact `at` attribute on `<d:p>`/`<d:h>`.
- **Run presentation**: `w:rPr/w:spacing`,
  `w:shadow`, `w:emboss`, `w:imprint`, `w:effect`, `w:border` (run), `w:shd` (run).
  Note: `w:rFonts`, `w:sz`, `w:color`, `w:highlight` are **preserved** via `<span>` (see spec §3.2).
- **Fields**: `w:fldSimple`, `w:fldChar`, `w:instrText` (TOC, PAGE, REF, …) and their
  cached results — except HYPERLINK (resolved via `r:id` **or** `instrText`), kept as `<a>`.
- **Anchors**: `w:bookmarkStart` / `w:bookmarkEnd` — **preserved** in `<notes>` as `<bm id="name"/>`.
- **Review**: `w:commentRange*`, `w:commentReference` — **preserved** in `<notes>` as `<comment>`.
  `w:proofError` — dropped.
- **Track changes (semantic mode)**: `w:ins`, `w:del`, `w:delText` — dropped in `mode="semantic"` (default); preserved as `<change>` in `mode="lossless"`.
- **Wrappers**: `w:sdt`, `w:smartTag`, `w:customXml` — unwrapped to children.

> These removals are intentional noise reduction, not defects.
> Note: strikethrough (`w:strike`/`w:dstrike`), small/ALL caps, and RTL direction are
> **preserved** (`<s>`, `<smallcaps>`, `<uppercase>`, `dir="rtl"`).

---

## 3. Partially Handled (kept but lossy)

| Construct | Handling | Lost detail |
|-----------|----------|-------------|
| **Table styling** (`w:tblPr`/`w:tcPr`) | borders preserved via `at` attribute; shading dropped; `w:tblGrid`/`w:gridCol` → `<s:col ref="n">` linked to `<d:table id="n">`; `colspan`/`rowspan` preserved | shading/color lost |
| **Section layout** (`w:sectPr`) | page size + margins → `<s:page>`; **multiple sections** (`<s:page>` per section); headers/footers → `<header>`/`<footer>` | — |
| **Vertical alignment** (`w:vertAlign`) | mapped to `<sup>`/`<sub>` | only sup/sub; other vertAlign values dropped |
| **Multilingual** (`w:lang`) | propagated to `lang` attribute per element (BCP 47) | resolved in spec v1.0.1 |
| **Code detection** (style/font heuristics) | pattern-matched to `<d:code>`; monospace font OR style name | false positives/negatives possible (custom styles named "Code" for non-code content) |

---

## 4. Formatting Constraints

- **Units**: all layout values resolved to the declared `unit` (default `in`); twips
  converted. Inconsistent source units are normalized — original unit labels are not kept.
- **Processing mode**: `mode="semantic"` (default) normalizes whitespace; `mode="lossless"` preserves more layout detail for round-tripping.
- **Whitespace**: in `semantic` mode, repeated spaces collapsed, `w:tab` → single space,
  `w:br`/`w:cr` → `<br/>`. Honors `xml:space="preserve"` when present on `<w:t>`.
  **Exception**: content inside `<d:code>` preserves original spacing verbatim in both modes.
- **Text**: kept verbatim (no translation/summarization). Tracked-deletion text is wrapped
  in `<change type="delete">` in `lossless` mode; dropped in `semantic` mode.
- **XML escaping**: `&`, `<`, `>`, `"` are escaped per XML 1.0. Forbidden control characters
  (0x00–0x08, 0x0B–0x0C, 0x0E–0x1F, 0x7F–0x84) are stripped. No CDATA sections emitted.

---

## 5. Correctness Boundaries

- The transform is **idempotent** (same `.docx` → identical `words`) only when source
  relationships (`document.xml.rels`, `numbering.xml`, `styles.xml`) are present and valid.
- Output validity depends on a well-formed `.docx` package; corrupt or partial packages may
  produce partial `words` and are not auto-repaired.
- The preprocessor performs **no semantic understanding** — it maps structure, not meaning.

---

## 6. Planned Follow-ups

- Optionally preserve rendered field results (TOC/PAGE snapshot) instead of dropping.
- Revisit image handling: pluggable extractor interface (OCR/Vision) feeding `<img>`.
- Consider extracting textbox images via OCR (currently the textbox *text* is kept, the
  embedded image is still `<img>`).

### Resolved in v1.0.1

The following gaps were identified and addressed during specification review:

| Issue | Resolution |
|-------|-----------|
| **Column identity ambiguity** in multiple/nested tables | `<s:col ref="n">` paired with `<d:table id="n">` (1-based index) |
| **Grid integrity loss** from flatten `gridSpan`/`vMerge` | `colspan`/`rowspan` preserved as attributes on `<d:td>`/`<d:th>`, content not concatenated |
| **Whitespace contradiction** with `<d:code>` | Content inside `<d:code>` excluded from whitespace normalization; literal tab preserved |
| **Permanent footnote content loss** | `<notes>` container added after `</write>` to store footnote/endnote body text |
| **Blind to list numbering restart** (`w:lvlOverride`) | `w:lvlOverride` detected; list split into separate `<d:ol>` elements with `start="n"` attribute |
| **`<d:code>` trigger undefined** | Style name matching (`Code`, `Source`, `Output`) + fallback monospace font detection; inline formatting suppressed inside `<d:code>` |
| **Nested list structure unclear** | `<d:ul>`/`<d:ol>` placed inside parent `<d:li>`; arbitrary depth, mixed types allowed |
| **Sentence fragmentation** from inline textbox anchors | Runs before/after textbox anchor merged into single `<d:p>`; textbox content emitted as sibling elements afterward |
| **Malformed XML** from verbatim text & URLs | All text and attributes escaped per XML 1.0 (`&`, `<`, `>`, `"`); forbidden control chars (0x00–0x08, etc.) stripped |
| **Namespace XML undefined** | Three namespaces declared: `xmlns="urn:words:v1"` (structural), `xmlns:d="urn:words:v1:doc"` (block content), `xmlns:s="urn:words:v1:style"` (layout); inline elements use no prefix |
| **DROP too aggressive** (justification, tracked changes) | `w:jc` → `<s:align>` in `<style>` (LOSSLESS_METADATA); `w:ins`/`w:del` → optional `<change>` |
| **Header/footer dropped** | Now KEPT in `<header>`/`<footer>` blocks per section, processed with normal transformation rules |
| **Ambiguity marker vs body footnote** | Split: `<fn-ref id="n" type="footnote|endnote"/>` (marker) and `<fn id="n" type="...">` (body), `id` as connector |
| **vMerge restart/continue unresolved** | Grid reconstruction algorithm: restart → rowspan, continue → omit |
| **Inaccurate list grouping** | Grouping considers `numId` + `ilvl` + `abstractNumId` + restart state |
| **Whitespace normalization violates 1:1** | `mode="semantic"` (default) for AI; `mode="lossless"` for minimal transformation; `xml:space="preserve"` honored |
| **Style inheritance not resolved** | Inheritance chain (`w:basedOn`) walked until finding known semantic style |
| **Missing document metadata** | `<meta>` block added with title/author/created/modified/keywords from `docProps/core.xml` |
| **Language preservation optional** | `lang` attribute (BCP 47) now REQUIRED on all block elements |
| **Font/size/color/highlight dropped** | Now preserved via `<span font=".." size=".." color=".." highlight="..">` inline element |
| **`<style>` optional** | Now REQUIRED with at minimum `<s:page>` (size + margins); default unit changed to `in` |
| **Borders dropped** | Now preserved via compact `at` attribute on `<d:p>`, `<d:h>`, `<td>`, `<th>`, `<table>` |
| **Bookmarks dropped** | Now preserved in `<notes>` as `<bm id="name"/>` |
| **Comments dropped** | Now preserved in `<notes>` as `<comment id="n" author="..." date="...">text</comment>` |

---

*See also:* `docx-preprosessor.md` (§2.1, §3.0, §8).
