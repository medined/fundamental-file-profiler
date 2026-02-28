# fundamental-file-profiler

Creates metadata for a static set of files.

# File Profiling Tool Requirements

## Command-line Arguments
- `--source-directory`: Path to the directory to scan.
- `--extracted-text-directory`: Path to save extracted text.
- `--expanded-file-directory`: Path for expanded archive contents.
- `--sqlite-file`: Path for SQLite database.
- `--log-file`: Path for logging exceptions.

## Edge Cases

Encoding Issues: Malformed Unicode and its handling (skip, replace, or log references).

Structural Skew: Double-row headers in spreadsheets; mid-file schema shifts (e.g., mixing people and cars).

Missing or Sparse Data: Partial rows missing fields; zero-length files; header-only files.

Pattern Detection: Auto-incrementing fields (ID columns).

Large Volume: 50,000+ files or massive sets impacting performance.

Delimiters: Ambiguous or multiple delimiters; best-guess needed.

Headerless Files: No header line present in CSV.


## Process Steps

* Collect file details.
* Set ignore flag.
    * Thumbs.db
* Calculate the MD5.
    * if duplicate
        * find earliest file. Update duplicate-of field
        * set ignore flag
* Set file category.
    * "image" - gif, bmp
    * "audio" - mp3, wav
    * "video" - mp4, mkv
    * "njdson"
    * "json"
* Expand files.
    * zip, rar, tar, tgz, 7z
* Copy MDB to parquet
    * Each table gets its own parquet file.
    * then file is ignored
* Copy Spreadsheet to parquet
    * Each tab gets its own parquet file
    * then file is ignored
* Calculate the MD5 again (for expanded files).
    * now we can identify duplicate database tables and spreadshee tabs.
* Extract column names.
    * csv, parquet
* Generate column sets.
    * Files with the same column set can be combined.
* Run Tika.
    * Extract metadat
    * Extract text
* Count records.

## Database Schema

### file_detail
- created_at, p0, p1, file_id, duplicate_of, file_basename, file_category, file_ext, compressed, is_directory, password_protected, file_md5, file_path, file_size, was_expanded, ignored, last_modified, mime_type, parent_uuid, record_count, file_uuid

### column_detail
- earliest_uuid, file_path, sheet_name, sheet_index, column_name, column_index, comment

### json_detail
- earliest_uuid, file_path, json_path, reference_count

### tika_metadata
- earliest_uuid, file_path, field_name, field_value


### column_set

```sql
create table if not exists column_set as
select 
    earliest_uuid,
    '[' || group_concat('"' || replace(column_name, '"', '""') || '"', ', ') || ']' as column_set
from (
    select earliest_uuid, column_name
    from column_detail
    order by 1, 2
)
group by earliest_uuid
order by earliest_uuid


create table if not exists column_set_grouped as
select column_set, count(1) as amount
from column_set
group by 1
```

