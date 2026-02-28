# Synthetic Edge Case Summary Log

- `malformed_unicode.csv`: Contains an invalid UTF-8 byte sequence (`0xC3 0x28`) in a data row.
- `mixed_schema.csv`: Starts with a 3-column schema, then switches to a 4-column schema mid-file.
- `missing_fields.csv`: Includes rows with missing values and truncated rows with fewer fields than header.
- `multiple_delimiters.csv`: Mixes comma-delimited and semicolon-delimited rows in the same file.
- `header_only.csv`: Contains only a header row and no data rows.
- `no_header.csv`: Contains only data rows with no header line.
