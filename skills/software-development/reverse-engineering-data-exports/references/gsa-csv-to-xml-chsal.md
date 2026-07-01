# CHSAL CSV → input XML notes

Session-derived notes for the CHSAL export reverse-engineering path.

## Source facts
- CSV export: `CHSAL_2026-05-08_GSA_export.csv`
- Consolidated XML fixture: `CHConsolidatedList_scrapy_20240826_140202.xml`
- CSV is denormalized: one entity spans many rows.
- The CSV contains 8,007 unique entities.
- `entityauthorityid` is the correct key to use as target `ssid` for GSA→input conversion.
- If `entityauthorityid` is blank, fall back to `entityid` only to avoid dropping entities.

## Output rules discovered
- Use a listed modification (`modification-type="listed"`) with `effective-date` taken from `entityvalidfrom` when present.
- Add a de-listed modification (`modification-type="de-listed"`) dated to the current day boundary used by the task.
- Downstream `InputToHistory` keeps entities when both valid-from and valid-to exist; targets with only a de-listed marker may be discarded.
- `CommonDataConverter` in the PCE stage subtracts one day from the extracted `validTo` date when building the final PCE validTo.
  - Practical implication: if the downstream PCE validTo should end up on day N, the input XML de-listed date often needs to be day N+1.

## Structural notes
- One target per entity.
- Preserve raw literal name text in `<value>`.
- Use the repository’s JAXB builder style and existing generated model classes.
- `entityauthorityid` should also drive `sanctions-set-id` for the target.

## Verification notes
- The generated CHSAL input XML should parse cleanly and contain one target per entity.
- Verify that the output includes both a listed and de-listed modification so history conversion retains the entity.
