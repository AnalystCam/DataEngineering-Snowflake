# **Social Media Floodgate – Snowflake Data Engineering Project**

This project demonstrates how to build a Snowflake-based **data ingestion and transformation pipeline** for social media data, using **Twitter JSON data** as an example.  
The workflow covers:  
- Creating the database and schema.  
- Ingesting raw JSON data into Snowflake.  
- Querying and transforming nested JSON fields.  
- Using `FLATTEN` to work with arrays and nested entities.  

---

## **Project Overview**  
This solution simulates the ingestion of tweet data from an internal stage in Snowflake, processes it using SQL queries, and extracts meaningful insights such as hashtags, user details, and URLs from nested JSON structures.  

---

## **Setup Instructions**  

### 1. Create the Database and Table  
```sql
CREATE DATABASE SOCIAL_MEDIA_FLOODGATES;

CREATE OR REPLACE TABLE SOCIAL_MEDIA_FLOODGATES.PUBLIC.TWEET_INGEST (
    RAW_STATUS VARIANT
);
```

---

### 2. Create JSON File Format  
```sql
CREATE FILE FORMAT SOCIAL_MEDIA_FLOODGATES.PUBLIC.json_file_format
TYPE = 'JSON'
COMPRESSION = 'AUTO'
ENABLE_OCTAL = FALSE
ALLOW_DUPLICATE = FALSE
STRIP_OUTER_ARRAY = TRUE
STRIP_NULL_VALUES = FALSE
IGNORE_UTF8_ERRORS = FALSE;
```

---

### 3. Load JSON Data into Snowflake  
```sql
COPY INTO SOCIAL_MEDIA_FLOODGATES.PUBLIC.TWEET_INGEST
FROM @UTIL_DB.PUBLIC.MY_INTERNAL_STAGE
FILES = ('nutrition_tweets.json')
FILE_FORMAT = (FORMAT_NAME = SOCIAL_MEDIA_FLOODGATES.PUBLIC.JSON_FILE_FORMAT);
```

---

## **Queries**  

### 1. Retrieve All Raw Data  
```sql
SELECT RAW_STATUS
FROM TWEET_INGEST;
```

### 2. Extract Entities and Hashtags  
```sql
SELECT RAW_STATUS:entities
FROM TWEET_INGEST;

SELECT RAW_STATUS:entities:hashtags
FROM TWEET_INGEST;
```

---

### 3. Retrieve First Hashtag Text  
```sql
SELECT RAW_STATUS:entities:hashtags[0].text
FROM TWEET_INGEST
WHERE RAW_STATUS:entities:hashtags[0].text IS NOT NULL;
```

---

### 4. Convert Tweet Date  
```sql
SELECT RAW_STATUS:created_at::date
FROM TWEET_INGEST
ORDER BY RAW_STATUS:created_at::date;
```

---

### 5. Flatten Nested Arrays (Extract URLs)  
```sql
SELECT VALUE
FROM TWEET_INGEST,
LATERAL FLATTEN(INPUT => RAW_STATUS:entities:URLs);
```

---

### 6. Flatten Hashtags and Add Tweet Metadata  
```sql
SELECT RAW_STATUS:user:name::text AS user_name,
       RAW_STATUS:id AS tweet_id,
       VALUE:text::varchar AS hashtag_used
FROM TWEET_INGEST,
LATERAL FLATTEN(INPUT => RAW_STATUS:entities:hashtags);
```

---

## **Key Learning Points**  
- Working with **VARIANT** columns to store semi-structured JSON data.  
- Using **Snowflake File Formats** to define ingestion rules.  
- Querying nested JSON fields with Snowflake dot notation (`:`).  
- Leveraging **FLATTEN** for arrays and nested objects.  
- Casting data types for analysis.  

---

## **Requirements**  
- **Snowflake Account** (with access to create databases and load data)  
- Sample JSON file (`nutrition_tweets.json`)  
- Basic SQL knowledge  

---

## **Author**  
👤 *Adedayo Adeola* – Data Engineer / BI Developer  

