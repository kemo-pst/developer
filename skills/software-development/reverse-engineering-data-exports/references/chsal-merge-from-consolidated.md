# CHSAL merge from consolidated fixture

Notes from the CHSAL consolidation flow.

## Input shape
- `CHConsolidatedList_scrapy_20240826_140202.xml` is a structured downstream fixture, not the CSV export.
- It already contains `<modification>` blocks with `listed` and `de-listed` events.
- It can be used as a structural reference for the target XML hierarchy.

## Merge guidance
- Use the consolidated XML only as the schema/shape reference.
- Do not assume the same SSID namespace as the GSA CSV export.
- Preserve the hierarchy: root → sanctions-program / target → individual/entity → identity → name.
- When generating a fresh input XML from the CSV, reconcile the CSV authority ID with the target SSID rule instead of copying the consolidated fixture’s numeric IDs.

## Practical verification
- Compare the first target block and the modification block structure against the fixture.
- Check that the regenerated file includes one listed and one de-listed modification per entity.
- Confirm the root date and modification dates use the intended task date.
