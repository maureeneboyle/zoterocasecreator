---
name: cases-to-zotero
description: >-
  Convert one or more U.S. case citations into a CSL-JSON file for import into
  Zotero as Case items. Parses each citation into structured fields and reduces
  parallel citations to the single Bluebook-preferred reporter (academic style).
  Use this whenever the user has case citations (pasted, in a list, or handed off
  from a citation-extraction step) and wants them in Zotero, or whenever the user
  mentions Zotero, CSL-JSON, importing cases, or building a case library — even if
  they do not say "CSL-JSON" explicitly.
---

# Cases to Zotero

## What this does

Turn a list of U.S. case citations into a CSL-JSON file that Zotero imports
directly as Case items (File > Import > A file). Each citation is parsed into
structured fields, normalized to Bluebook academic style (a single preferred
reporter), and written as one JSON array.

The hard part is judgment, not mechanics: read each citation, decide how it maps
to fields, and decide which reporter survives. Do that carefully rather than
pattern-matching, and flag anything uncertain instead of guessing.

## Input

A list of case citations as text. Format varies — do not assume uniformity.
Citations may be pasted by the user or handed off from the `extract-case-citations`
skill.

## Steps

1. Parse each citation into: case name, volume, reporter, first page, court, year,
   and docket number if present.
2. Normalize per Bluebook — see **Reporter selection**.
3. Map the fields to CSL-JSON `legal_case` — see **Field mapping**.
4. Emit a single JSON array to a `.json` file in the outputs directory.
5. Before the file, print a verification table and a Flags list so errors surface
   before import, not after.

## Reporter selection (Bluebook academic style)

This library targets academic / law-review footnote style. Under Bluebook Rule
10.3.1(b), academic writing cites a single preferred reporter; parallel citations
belong only in documents filed in a court whose local rules require them
(Rule 10.3.1(a)). So:

- If a citation lists more than one reporter (a parallel citation), keep only the
  Bluebook Table T1 preferred reporter and **discard** the others. Do not preserve
  the dropped reporters anywhere.
  - **U.S. Supreme Court:** keep `U.S.` (United States Reports); drop `S. Ct.` and
    `L. Ed.`
  - **State cases:** keep the regional reporter (e.g., `N.E.`, `P.`, `A.`, and their
    numbered series) per Table T1; drop the official state reporter.
- After dropping a reporter, re-check the court/date parenthetical. Dropping a
  reporter can remove the signal that conveyed the jurisdiction, so add the court or
  state abbreviation back if the surviving reporter's title no longer conveys it
  (Rule 10.2(b); Bluepages B10.1.3). Follow Rule 10.4 for the abbreviation: for the
  **highest court** of a state, the state alone suffices (e.g., `N.Y.`); for a lower
  court, name the court (e.g., `N.Y. App. Div.`, `9th Cir.`, `D. Mass.`).
- If the preferred reporter for a jurisdiction is unclear — an unusual court, a very
  old case, or a reporter not in Table T1 — do not guess. Keep the citation as given
  and flag it for review.

## Field mapping (CSL `legal_case` -> Zotero Case)

| CSL variable      | Zotero Case field | Value                                           |
|-------------------|-------------------|-------------------------------------------------|
| `type`            | (item type)       | always `"legal_case"`                           |
| `title`           | Case Name         | e.g., `"Pierson v. Post"`                        |
| `container-title` | Reporter          | abbreviated, e.g., `"U.S."`, `"F. Cas."`, `"N.E."` |
| `volume`          | Reporter Volume   | e.g., `"3"`                                      |
| `page`            | First Page        | e.g., `"175"`                                    |
| `authority`       | Court             | abbreviation, e.g., `"D. Mass."`; omit if conveyed by reporter |
| `number`          | Docket Number     | e.g., `"No. 13,720"` (Federal Cases numbers)     |
| `issued`          | Date Decided      | `{"date-parts": [[YEAR]]}`; year-only is fine    |

Give each record a short unique `id` (e.g., `"pierson-post-1805"`).

## Rules and conventions

- U.S. Reports implies the U.S. Supreme Court. Leave `authority` empty — the
  reporter conveys the court.
- Federal Cases (`F. Cas.`) citations conventionally carry a case number in
  parentheses, e.g., `(No. 13,720)`. If present, put it in `number`; if absent, omit
  it and note it in Flags.
- Citations usually supply only a year. Store year-only in `issued`; never fabricate
  a month or day.
- Keep reporter abbreviations in Bluebook form; do not expand them.

## Output

- One JSON array with all records, saved as a `.json` file the user can import.
- Immediately before the file, print:
  - a verification table with columns `Case Name | Vol | Reporter | Page | Court | Year`, and
  - a **Flags** list of anything dropped, assumed, or needing review.
- Report a count: "N records, M flagged."

## Import steps (give these to the user)

1. In Zotero: **File > Import > A file**, and select the `.json`.
2. Import into a **new collection** to isolate the batch for checking.
3. Confirm the items appear with Item Type **Case**.
4. Spot-check by generating a bibliography entry in the target citation style — the
   fields can look right yet still render wrong, and that is the real test.

## Example

Input:

```
MacPherson v. Buick Motor Co., 217 N.Y. 382, 111 N.E. 1050 (1916)
```

This is a parallel citation (official `N.Y.` reporter plus regional `N.E.`
reporter) from the New York Court of Appeals, New York's highest court. Academic
style keeps the regional reporter and drops the official one. `N.E.` does not
convey the state, so the parenthetical gains `N.Y.`; because it is the highest
court, the state alone suffices (no court name).

Output record:

```json
{
  "id": "macpherson-buick-1916",
  "type": "legal_case",
  "title": "MacPherson v. Buick Motor Co.",
  "container-title": "N.E.",
  "volume": "111",
  "page": "1050",
  "authority": "N.Y.",
  "issued": { "date-parts": [[1916]] }
}
```

Flag: dropped parallel cite `217 N.Y. 382`.
