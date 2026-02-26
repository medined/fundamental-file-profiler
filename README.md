# fundamental-file-profiler

Creates metadata for a static set of files.

# File Profiling Tool Requirements

## Command-line Arguments
- `--source-directory`: Path to the directory to scan.
- `--extracted-text-directory`: Path to save extracted text.
- `--expanded-file-directory`: Path for expanded archive contents.
- `--sqlite-file`: Path for SQLite database.
- `--log-file`: Path for logging exceptions.

## Process Steps
1. Scan the source directory recursively.
2. Expand archives (into expanded directory).
3. Extract metadata and text using Apache Tika.
4. Populate the SQLite database.
5. Assign a sequential `file_id`.

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

