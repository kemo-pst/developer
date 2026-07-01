# Literal preservation pitfall

When converting name data through the CHSAL pipeline, never serialize wrapper objects with `.toString()` into schema text fields.

## Symptom
The output XML or JSONL contains text like:

`MultiLanguageText[languageText={LanguageTextDescription[...]}]`

instead of the actual name literal.

## Cause
A `MultiLanguageText` object was created correctly, but the code wrote `multiLanguageText.toString()` into the destination field.

## Fix pattern
- Keep the wrapper object only as a transport/helper object.
- Extract the inner literal string and write that into `NamePart.value` / XML `<value>`.
- For CHSAL name generation, use the raw source text from the CSV row or the generated literal field, not the wrapper object string.

## Verified example
In the CHSAL GSA-side converter, the bug was corrected by changing:

`withValue(name.toString())`

to:

`withValue(nameText)`

where `nameText` is the raw literal from the source row.
