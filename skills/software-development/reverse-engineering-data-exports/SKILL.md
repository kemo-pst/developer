---
name: reverse-engineering-data-exports
description: Reverse engineer CSV/flat exports into the repository's canonical XML/JSON model by inspecting schema, generated examples, and converter code; handle malformed source fields and verify round-trips.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [csv, xml, reverse-engineering, export, conversion, schema, jaxb, data-forensics, sanitization, verification]
    related_skills: [spike, plan, writing-plans]
---

# Reverse-engineering data exports

Use this skill when the user gives you a flat export or vendor dump and asks you to reconstruct the repo’s canonical input format, especially when the source is denormalized (CSV, TSV, spreadsheet, database export) and the target is structured XML/JSON.

Typical signals:
- "convert this CSV to XML/JSON"
- "reverse engineer the input format"
- "derive the source mapping from the repository"
- "build an input file that matches the existing converter"
- "why is validTo / delisted / status not appearing"

## Core workflow

1. **Find the canonical model first**
   - Inspect generated sources, schema files, tests, and any checked-in example outputs.
   - Prefer repository-generated examples over assumptions.
   - Identify root element, required attributes, element order, and grouping keys.
   - If the repo already contains a checked-in sample for the exact export date, compare against it before inventing a new mapping.

2. **Find the actual conversion path**
   - Locate converter classes, mappers, helper utilities, and tests.
   - Determine whether the repo expects direct XML construction, JAXB output, or an intermediate model.
   - Extract the field-to-field mapping from code, not from the raw sample alone.

3. **Recover the grouping logic**
   - Flat exports often have one row per attribute or comment.
   - Reconstruct grouping keys such as entity ID, address ID, or program ID.
   - Deduplicate repeated rows while preserving the source order where the repo expects it.

### Special-state encoding
- Delist / removal / status-end markers may be represented by dedicated XML nodes or modification records.
- Do not guess the semantic marker; confirm the exact enum/value from the schema or generated enum.
- If the repository’s current output model expects a delist marker for active entities, synthesize that marker at the output boundary rather than mutating upstream source semantics.

5. **Sanitize source text before serialization**
   - Flat exports often contain invalid XML characters, bad control bytes, or unescaped punctuation.
   - Strip XML-1.0-invalid control characters before writing XML.
   - Keep meaningful whitespace, but normalize only what would break well-formedness.

6. **Verify the output by parsing it back**
   - Parse the generated file with the same XML parser/JAXB path if possible.
   - Check root tag, counts, required attributes, and one or two sample records.
   - When possible, compare the generated output against a known-good sample from the repo.

## Practical heuristics

### Flat CSV exports
- Treat the first occurrence of a record ID as the anchor row.
- Use auxiliary IDs as child collection keys (e.g. address ID, name ID, info ID).
- Prefer explicit type columns (`type`, `code`, `status`) over row positions.
- When multiple rows share the same anchor, keep them in source order unless the repo sorts them.

### XML generation
- Use the repo’s own serializer if available.
- If generating manually, match the repository’s observed element order and attribute names.
- Preserve required fields even when optional sections are empty.
- For CHSAL-style sanction outputs, keep `<value>` text as the raw literal only; do not serialize debug wrappers such as `MultiLanguageText[...]`.
- If the repository already has a generated XML for the same export, update that artifact in-place when appropriate and verify counts/root attrs instead of re-inventing the serializer.
- For CHSAL-style sanction lists, keep `<value>` elements as raw literals only; do not serialize debug strings such as `MultiLanguageText[...]`.
- When a downstream history pipeline expects both valid-from and valid-to, emit both a `listed` modification (from the source entry/valid-from date) and a `de-listed` modification (from the source valid-to date, or the file/task date if blank) so the entity is not dropped.
- In CHSAL CSV→input generation, honor the task/source mapping for target SSIDs. If the user or source file says `entityid`, use `entityid` as the target `ssid`; only use `entityauthorityid` when that is the explicit rule for the dataset.
- If the repo already contains a CSV-driven XML artifact for the same export, treat it as the starting point and normalize it rather than rebuilding the structure from the consolidated list alone.
- If the repo’s current business rule requires an entity to be delisted, emit a de-list modification with the effective date set to the current day unless the source already provides a delist date.

### Malformed text
- If XML parse fails, inspect the offending line and look for:
  - control characters below U+0020 (except TAB, LF, CR)
  - unescaped `&`, `<`, `>` inside text
  - broken quotes in CSV cells that were copied verbatim into XML
- Fix text sanitation first before changing the mapping.

## Verification checklist

- [ ] Root element matches the schema
- [ ] Required attributes are present
- [ ] Record count matches expectation
- [ ] Grouping keys produce one canonical entity per source entity
- [ ] Delist/status markers appear in the expected XML structure
- [ ] The output parses cleanly after generation

## Literal Preservation Pitfall

When the source contains wrapper objects (`MultiLanguageText`, `Optional`, etc.), writing `.toString()` into a schema field produces debug-style output like `MultiLanguageText[languageText={...}]` instead of the raw literal. Extract the underlying text before writing to the schema field. See `references/literal-preservation-pitfall.md` for the exact fix pattern.

## Pitfalls

- Do not infer field semantics from a single sample row when the repo already contains a converter or schema.
- Do not encode temporary repair assumptions as permanent rules; always verify against a parseable output.
- Do not stop after writing a file—always parse the result back or compare against a repository fixture.

## References

- See `references/gsa-csv-to-xml.md` for the CSV→XML mapping, invalid-character sanitization note, and verification pattern discovered in the CHSAL export conversion session.
- See `references/gsa-csv-to-xml-chsal.md` for the CHSAL-specific mapping notes, delist rule, and sample artifact cross-checks from this session.
- See `references/chsal-merge-from-consolidated.md` for the section-wise merge pattern, zero-overlap observation, and verification steps when combining CHSAL with the consolidated-list artifact.
- See `references/chsal-input-date-mapping.md` for the concrete `entityid` target-SSID rule and `entityvalidfrom`/`entityvalidto` normalization used in this session.
