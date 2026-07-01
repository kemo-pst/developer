# CHSAL XML merge notes

Session notes for merging a GSA-converted CHSAL XML with a consolidated CHSAL XML.

## Recommended merge pattern
- Use the GSA-converted XML as the base file for entity/target data.
- Use the consolidated XML as a structural overlay for missing top-level sections such as `sanctions-program` and `place`.
- Merge top-level nodes by `ssid`.
- For matching targets, recurse into child nodes and append only missing repeatable entries.
- Preserve base values when both files contain the same field.

## Delisting behavior
- If you need every target delisted, add or update a `modification` node with `modification-type="de-listed"`.
- Set `effective-date`, `enactment-date`, and `publication-date` consistently.
- A CLI flag like `--delist-date YYYY-MM-DD` is a good pattern for this so the merge script stays reusable.

## Verification
- Parse the merged XML back with the same XML parser after writing it.
- Check root tag, record counts, and a few sample targets.
- If the merged file is very large, verify on a small set of sample `ssid` values before trusting the full artifact.
