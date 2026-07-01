# GSA CSV → XML mapping notes

General mapping pattern learned while reverse engineering the repository’s GSA export pipeline.

## Workflow
1. Inspect the repository’s checked-in XML sample for the target shape.
2. Inspect generated JAXB/model classes for exact builder/property names.
3. Inspect the conversion code path for field grouping and normalization.
4. Map the denormalized CSV rows to one canonical entity tree.
5. Generate XML and verify by parsing the result back.

## Common pitfalls
- Do not use `.toString()` on wrapper objects when the XML schema wants raw text.
- Preserve the repo’s element order and required attributes.
- Treat source export rows as denormalized facts; aggregate before emitting target entities.
- When the user asks for a specific date cutoff, ensure all derived delist/status records use that date consistently.

## Verification tips
- Parse the generated XML with the repo’s JAXB model or a standard XML parser.
- Count the resulting targets/programs/records and compare against the grouped source entity count.
- Search for object debug strings like `MultiLanguageText[` in the output; they should not appear in raw value nodes.
