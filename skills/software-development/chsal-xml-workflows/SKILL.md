---
name: chsal-xml-workflows
description: Merge, normalize, and delist CHSAL XML exports by combining GSA-converted files with consolidated CHSAL fixtures.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [chsal, xml, merge, delist, sanctions, export, gsa]
---

# CHSAL XML workflows

Use this skill when working with CHSAL XML exports that need to be merged, normalized, or delisted.

## Typical inputs
- A GSA-converted CHSAL XML file containing target/entity data.
- A consolidated CHSAL XML file containing structural context such as program and place sections.

## Recommended workflow
1. Treat the GSA-converted XML as the base file for target content.
2. Use the consolidated XML as an overlay for missing top-level structure.
3. Merge top-level nodes by `ssid`.
4. Recurse into matching targets and append only missing repeatable child nodes.
5. Preserve base values when both files contain the same field.
6. If required, add or update `de-listed` modifications in the output.
7. Parse the merged file back after writing it.

## Delisting rule
- To mark a target as delisted, ensure there is a `modification` node with `modification-type="de-listed"`.
- Set `effective-date`, `enactment-date`, and `publication-date` consistently.
- A CLI option such as `--delist-date YYYY-MM-DD` is a reusable way to apply the same delist date across the merged output.

## Merge pitfalls
- Do not assume the consolidated file should replace the GSA-derived target data.
- Do not duplicate repeated child nodes such as names, justifications, and other-information unless the value is genuinely new.
- Use the source `ssid` as the stable merge key.
- If the output becomes huge, verify a few sample targets by parsing the result and inspecting their modification blocks.

## Verification checklist
- [ ] Output parses cleanly with the XML parser
- [ ] Root attributes are preserved
- [ ] Top-level program/place sections are present
- [ ] Target data still comes from the GSA file
- [ ] Delist modifications are present when requested

## References
- See `references/chsal-xml-merge.md` for the session notes on merge strategy and delist behavior.
