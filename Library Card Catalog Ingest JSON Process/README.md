# README for Library Card Catalog Ingest JSON Process

This README explains the setup and usage of the `library_card_catalog` schema for ingesting JSON data into tables, particularly focusing on `author_ingest_json` and `nested_ingest_json` tables. The process involves creating tables, file formats, and using the `COPY INTO` command to load JSON data into Snowflake.

---

## Table of Contents

1. [Author Ingest JSON Table](#author-ingest-json-table)
2. [File Format for JSON Files](#file-format-for-json-files)
3. [Loading JSON Data into `author_ingest_json`](#loading-json-data-into-author_ingest_json)
4. [Querying Data from `author_ingest_json`](#querying-data-from-author_ingest_json)
5. [Nested JSON Ingest](#nested-json-ingest)
6. [Loading JSON Data into `nested_ingest_json`](#loading-json-data-into-nested_ingest_json)
7. [Querying Data from `nested_ingest_json`](#querying-data-from-nested_ingest_json)

---

### Author Ingest JSON Table

The `author_ingest_json` table is designed to store raw author data in JSON format. This table has a single column `raw_author` of type `VARIANT`.

```sql
CREATE TABLE library_card_catalog.public.author_ingest_json
(
  raw_author VARIANT
);
```

### File Format for JSON Files

The `json_file_format` is defined to handle JSON file ingestion. The following properties are used:

* `type = 'JSON'`: Specifies the file type as JSON.
* `compression = 'AUTO'`: Automatically handles compression.
* `enable_octal = FALSE`: Disables octal interpretation.
* `allow_duplicate = FALSE`: Disallows duplicate records.
* `strip_outer_array = TRUE`: Strips outer arrays from the JSON file.
* `strip_null_values = FALSE`: Retains null values in the JSON.
* `ignore_utf8_errors = FALSE`: Does not ignore UTF-8 encoding errors.

```sql
CREATE FILE FORMAT library_card_catalog.public.json_file_format 
TYPE = 'JSON' 
COMPRESSION = 'AUTO' 
ENABLE_OCTAL = FALSE 
ALLOW_DUPLICATE = FALSE 
STRIP_OUTER_ARRAY = TRUE 
STRIP_NULL_VALUES = FALSE 
IGNORE_UTF8_ERRORS = FALSE;
```

### Loading JSON Data into `author_ingest_json`

Once the file format is defined, data can be loaded into the `author_ingest_json` table from a staged file (`author_with_header.json`).

1. First, I examine the staging data using the following query:

```sql
SELECT $1 
FROM @UTIL_DB.PUBLIC.MY_INTERNAL_STAGE/author_with_header.json
(FILE_FORMAT => LIBRARY_CARD_CATALOG.PUBLIC.JSON_FILE_FORMAT);
```

2. Then, load the data using the `COPY INTO` command:

```sql
COPY INTO LIBRARY_CARD_CATALOG.PUBLIC.AUTHOR_INGEST_JSON
FROM @UTIL_DB.PUBLIC.MY_INTERNAL_STAGE
FILES = ('author_with_header.json')
FILE_FORMAT = (format_name = LIBRARY_CARD_CATALOG.PUBLIC.JSON_FILE_FORMAT);
```

### Querying Data from `author_ingest_json`

After loading the data, you can query the `author_ingest_json` table in the following ways:

1. Querying `AUTHOR_UID`:

```sql
SELECT raw_author:AUTHOR_UID
FROM author_ingest_json;
```

2. Selecting all fields from `author_ingest_json`:

```sql
SELECT *
FROM author_ingest_json;
```

3. Extracting `FIRST_NAME`, `MIDDLE_NAME`, and `LAST_NAME` from the `raw_author` JSON object:

```sql
SELECT 
 raw_author:AUTHOR_UID,
 raw_author:FIRST_NAME::STRING AS FIRST_NAME,
 raw_author:MIDDLE_NAME::STRING AS MIDDLE_NAME,
 raw_author:LAST_NAME::STRING AS LAST_NAME
FROM AUTHOR_INGEST_JSON;
```

---

### Nested JSON Ingest

The `nested_ingest_json` table stores nested JSON data related to books and authors. Like the `author_ingest_json` table, it stores raw JSON data in a `VARIANT` column called `raw_nested_book`.

```sql
CREATE OR REPLACE TABLE library_card_catalog.public.nested_ingest_json 
(
  raw_nested_book VARIANT
);
```

### Loading JSON Data into `nested_ingest_json`

Similar to the author data, the nested JSON data can be loaded into the `nested_ingest_json` table from a staged file (`json_book_author_nested.txt`).

```sql
COPY INTO LIBRARY_CARD_CATALOG.PUBLIC.NESTED_INGEST_JSON
FROM @UTIL_DB.PUBLIC.MY_INTERNAL_STAGE
FILES = ('json_book_author_nested.txt')
FILE_FORMAT = (format_name = LIBRARY_CARD_CATALOG.PUBLIC.JSON_FILE_FORMAT);
```

### Querying Data from `nested_ingest_json`

You can query the nested JSON data with the following examples:

1. View all the data in `nested_ingest_json`:

```sql
SELECT *
FROM nested_ingest_json;
```

2. Retrieve the entire nested `raw_nested_book`:

```sql
SELECT raw_nested_book
FROM nested_ingest_json;
```

3. Flattening the nested JSON structure to retrieve `first_name` of authors:

```sql
SELECT value:first_name
FROM nested_ingest_json,
LATERAL FLATTEN(input => raw_nested_book:authors);
```

4. Using `table(flatten())` to extract `first_name` of authors:

```sql
SELECT value:first_name
FROM nested_ingest_json,
TABLE(FLATTEN(raw_nested_book:authors));
```

5. Retrieve `first_name` and `last_name` of authors from the nested data:

```sql
SELECT value:first_name::VARCHAR, value:last_name::VARCHAR
FROM nested_ingest_json,
LATERAL FLATTEN(input => raw_nested_book:authors);
```

6. Get the `first_name` and `last_name` of authors, with custom aliases:

```sql
SELECT value:first_name::VARCHAR AS first_nm,
       value:last_name::VARCHAR AS last_nm
FROM nested_ingest_json,
LATERAL FLATTEN(input => raw_nested_book:authors);
```

---

## Conclusion

This README provides a guide for ingesting and querying JSON data in Snowflake. It covers the creation of tables, file formats, loading data, and querying both simple and nested JSON structures. The provided examples demonstrate how to extract useful information from JSON files efficiently.
