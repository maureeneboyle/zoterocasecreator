---
name: extract-case-citations
description: >-
  Extract full U.S. case citations from an article's footnotes or body text. Use
  this whenever the user provides an article (Word .docx, born-digital PDF, or
  pasted text) and wants the cases it cites pulled out — for example "grab all the
  cases from this article", "what cases does this piece cite", or "pull the
  citations from the footnotes". Resolves short-form citations (id., supra, and
  pincite-only forms), deduplicates, splits string citations, and flags
  textual-only references. Cases only. Hand the resulting list to the
  cases-to-zotero skill when the user wants the cases in Zotero.
---

# Extract Case Citations

## What this does

Read an article and return a clean, de-duplicated list of the full U.S. case
citations it contains — ready to hand to the `cases-to-zotero` skill. Legal
citations usually live in the footnotes, so scan those first, then the body.

This step stays deliberately "dumb": collect every case in its fullest form and
leave it intact. All Bluebook normalization (dropping parallel citations,
selecting a reporter) happens downstream in `cases-to-zotero`, not here.

## Input

- **Word (`.docx`):** footnotes are stored separately from the body, in
  `word/footnotes.xml`. Extract both body text and footnotes — a body-only read
  misses most citations. Use the `docx` skill or `python-docx`.
- **PDF (born-digital only):** extract the text layer, including footnotes at the
  bottom of each page. Use the `pdf` skill or a text extractor such as
  `pdfplumber`. **Scanned / image PDFs are out of scope** — if the file has no
  extractable text, stop and tell the user OCR is needed first.
- **Pasted text:** process directly.

## Steps

1. Extract all text, including footnotes.
2. Identify every case citation. A full case citation has the shape
   `Party v. Party, VOL REPORTER PAGE (COURT YEAR)`.
3. Resolve short forms to their antecedent full citation:
   - `id.` refers to the immediately preceding citation.
   - `supra note N` and `Name, supra` refer to a full citation given earlier —
     find it and use its full form.
   - Pincite-only short forms (`Pierson, 3 Cai. R. at 178`) map to the full cite of
     that case.
   - If a short form cannot be traced to a full citation anywhere in the document,
     flag it rather than emit a partial record.
4. Split string citations — several cases in one footnote separated by semicolons —
   into separate entries.
5. Deduplicate: a case cited many times yields one entry, built from the fullest
   form found.
6. Cases only: ignore statutes, regulations, constitutional provisions, books,
   articles, and other secondary sources.
7. Flag textual-only references — a case named in prose with no citation — instead
   of fabricating a citation for it.

## Output

- A plain numbered list of full case citations, one per line.
- A separate **Flags** section: unresolved short forms, textual-only mentions, and
  anything ambiguous.
- A count: "N unique cases found, M flagged."
- Do **not** convert to CSL-JSON here. If the user wants the cases in Zotero, hand
  this list to `cases-to-zotero`.

## Notes

- Preserve each citation's full form exactly as needed for downstream parsing:
  parties, volume, reporter, page, court, year. Leave parallel citations intact;
  `cases-to-zotero` handles them.

## Example

Footnote text:

```
See Pierson v. Post, 3 Cai. R. 175 (N.Y. Sup. Ct. 1805); Ghen v. Rich, 8 F. 159
(D. Mass. 1881). Pierson, 3 Cai. R. at 178, is the classic capture case. See also
id. at 180.
```

Output:

```
1. Pierson v. Post, 3 Cai. R. 175 (N.Y. Sup. Ct. 1805)
2. Ghen v. Rich, 8 F. 159 (D. Mass. 1881)
```

2 unique cases found, 0 flagged. (The "Pierson ... at 178" and "id. at 180" short
forms resolved to entry 1.)
