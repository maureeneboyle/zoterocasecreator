# zoterocasecreator

Two Agent Skills for Claude that turn U.S. case citations into clean Zotero
records, following Bluebook academic (law-review footnote) style.

Each skill is a single `SKILL.md` instruction file with no bundled code, so the
full behavior can be read before use.

## The skills

- **cases-to-zotero** — Converts a list of case citations into a CSL-JSON file
  that Zotero imports as Case items. Parses each citation into fields, drops
  parallel citations to the single Bluebook-preferred reporter (Rule 10.3.1(b)),
  and flags anything ambiguous.
- **extract-case-citations** — Pulls full case citations from an article's
  footnotes or body text (Word, born-digital PDF, or pasted text). Resolves short
  forms (`id.`, `supra`, pincite-only), deduplicates, splits string cites, and
  flags textual-only references. Cases only. Its output feeds `cases-to-zotero`.

## Install (claude.ai or Cowork)

Requires a paid Claude plan (Pro, Max, Team, or Enterprise) with code execution
enabled.

1. Download the matching zip from the `dist/` folder — `dist/cases-to-zotero.zip`
   or `dist/extract-case-citations.zip`.
2. In Claude, go to **Settings > Capabilities** (the Skills section, also shown as
   **Customize > Skills**) and upload the zip.
3. Once enabled on your account, the skill is available in both claude.ai and
   Cowork.

Each zip contains the skill's `SKILL.md` in its folder — the format Claude's skill
upload expects. The same zips also work for the Team/Enterprise org-provisioning
path (Organization settings > Skills), which requires a zip containing a
`SKILL.md`.

## Scope and limits

- Targets academic / law-review style; parallel citations are dropped, not kept.
  Not suited as-is for court filings that require parallel citations.
- U.S. cases only. Statutes and secondary sources are out of scope.
- Scanned (image-only) PDFs are not supported for extraction; OCR is needed first.

## License

Released under CC0-1.0 (public domain dedication): reuse, modify, and redistribute
freely, no attribution required.
