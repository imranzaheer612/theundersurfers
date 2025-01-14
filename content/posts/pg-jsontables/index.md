---
title: "The Power of JSON in PostgreSQL: Transforming Relational Data with JSON_TABLE()"
date: 2025-01-14T13:10:21+05:00
draft: false

description: "Learn how PostgreSQL 17 enhances JSON support with powerful features like JSON_TABLE(). This article explores the benefits of using JSON in SQL databases, compares SQL and NoSQL, and covers essential PostgreSQL JSON functions for querying, creating, and modifying JSON data efficiently."

resources:
  - name: "featured-image"
    src: "featured-image.webp"

tags: ["JSON in PostgreSQL", "JSON_TABLE()", "PostgreSQL 17", "JSON Querying", "JSON Functions", "NoSQL vs SQL"]
categories: ["Database"]
theme: "full"
images: ["https://theundersurfers.com/pg-jsontables/featured-image.webp"]
seo:
  images: ["https://theundersurfers.com/pg-jsontables/featured-image.webp"]
---

<!--more-->


JSON (JavaScript Object Notation) is a lightweight data interchange format that is human readable and machine parsable. If you are involved in web development then you may have an idea that JSON originated from javascript and so that it is being used in web development, data analysis, or software engineering.

In this article we will discuss the importance for JSON support for the RDMS and the addition of JSON_TABLE() in postgresql 17. 

## Why need JSON

JSON is a free form data type and as it is easily readable and parsable it is being used to transfer data from and to server and web applications. Its independent nature makes it ideal for across different platforms and programming languages.

As it is lightweight, flexible and easy to use, most of the data is being generated and transferred as JSON across Web APIs. Due to this many databases have emerged to store and exchange data in JSON and so does PostgreSQL.

## Why store JSON in SQL instead of NoSQL DB

There is an undeniable increase in the need for the data flexibility and scalability,  and that is the main reason why most of the users go for the NoSQL databases but NoSQL databases have its cons too. NoSQL and Relational models have their own importance and non-relational databases were never meant to replace the relational one.

You can simply use a NoSQL database but the Relational model works with data in a more structured and strict way which aligns more to the most of the enterprise business logics. Relational models use database normalization, relationships and strict data types that enforces ‘rigor’ on data and saves the business logic in a more structured way so that it also increases the processing. The ACID compliance of RDBMS makes it more suitable for most of the enterprise applications.

A lot of data is being generated and received in JSON format so relational databases need to be more friendly in handling and storing the JSON data. Software developers often have a hard time choosing one over another, especially when the format of data is unknown. A compromise solution could be using both relational and non-relational databases and setting up a communication system between them. Providing JSON support in RDBMs can be a way to go.

## Postgres and JSON

With the version 9.2, Postgres started their JSON support to the database by introducing a JSON-type column and some functions and operations on top of that. Initially it wasn’t that great but the community kept on adding more features and performance related fixes. In 9.3 they improved by adding additional constructor methods and extractor methods. In 9.4, they added the ability store “Binary JSON (JSONB)” which improved performance. So far the community has added awesome JSON support and still adding more improvements. Now we can say that Postgres can support JSON well.

This [article](https://www.enterprisedb.com/news/new-benchmarks-show-postgres-dominating-mongodb-varied-workloads) shows how Postgres can be better than MongoDB in handling JSON based documents.

### JSON Improvements in PG-17

The Postgres community has added many JSON functions so far to handle JSON data properly. You can find them [here](https://www.postgresql.org/docs/9.5/functions-json.html). PG-17 [beta1](https://www.postgresql.org/about/news/postgresql-17-beta-1-released-2865/) and [beta](https://www.postgresql.org/about/news/postgresql-17-beta-2-released-2885/) 2 are released, along with other improvements, the community has added some JSON improvements.

- Support for [JSON_TABLE()](https://www.postgresql.org/docs/17/functions-json.html#FUNCTIONS-SQLJSON-TABLE)
- New SQL/JSON constructor functions [JSON()](https://www.postgresql.org/docs/17/functions-json.html#FUNCTIONS-JSON-CREATION-TABLE), JSON_SCALAR(), and JSON_SERIALIZE()
- New SQL/JSON query function [JSON_EXISTS()](https://www.postgresql.org/docs/17/functions-json.html#FUNCTIONS-SQLJSON-QUERYING), JSON_QUERY(), and JSON_VALUE()
- Add [jsonpath](https://www.postgresql.org/docs/17/functions-json.html#FUNCTIONS-SQLJSON-PATH-OPERATORS) methods to convert JSON values to other JSON data types

Among all the new features added in the PG-17, the JSON_TABLES looks like a small improvement but actually it will have a major impact in handling JSON.

## Create Relational Table from JSON Object

### JSON_TABLE and How are they helpful

MySQL, Oracle and SQL added the support for the JSON_TABLE years ago and now postgres have it too. JSON_TABLE is a powerful function that enables the easy decomposition of JavaScript Object Notation (JSON) data into relational format. Doing this will allow you to use structured query language on the table.

They transform JSON data into relational data (rows and columns). This function can be particularly useful when you need to query and manipulate JSON data using SQL

### Key Features

- Transformation: Converts JSON data into a tabular format, allowing you to use SQL queries on JSON data as if it were a traditional table.
- Integration: Facilitates the integration of JSON data with relational data, enabling seamless querying and reporting.
- Flexibility: Supports various data types and structures within JSON, such as arrays and nested objects.

### Working of JSON_TABLE

This gives the power to extract  useful information from the json data. The `JSON_TABLE()` function, introduced in PostgreSQL 17, lets you extract and project JSON data as relational rows and columns, making it easier to query and manipulate semi-structured data. This functionality is particularly useful for integrating JSON data with relational queries, improving both usability and performance.

For example, consider a table `products` with a `details` column of `jsonb` type containing product information:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    details JSONB
);

INSERT INTO products (details)
VALUES 
('{"name": "Laptop", "price": 1500, "specs": {"ram": "16GB", "storage": "512GB"}}'),
('{"name": "Phone", "price": 800, "specs": {"ram": "8GB", "storage": "256GB"}}');

```

You can use `JSON_TABLE()` to extract and query specific fields from the JSON data:

```sql
SELECT *
FROM products,
     JSON_TABLE(
         details,
         '$' COLUMNS (
             product_name TEXT PATH '$.name',
             product_price NUMERIC PATH '$.price',
             ram TEXT PATH '$.specs.ram',
             storage TEXT PATH '$.specs.storage'
         )
     ) AS jt;

```

And boom now you have extracted columns from the json. Not only you can parse through a simple json but also nested json. You can also specify default values for the missing information.This query outputs the following table:

```sql
 id |                                     details                                     | product_name | product_price | ram  | storage 
----+---------------------------------------------------------------------------------+--------------+---------------+------+---------
  1 | {"name": "Laptop", "price": 1500, "specs": {"ram": "16GB", "storage": "512GB"}} | Laptop       |          1500 | 16GB | 512GB
  2 | {"name": "Phone", "price": 800, "specs": {"ram": "8GB", "storage": "256GB"}}    | Phone        |           800 | 8GB  | 256GB
(2 rows)
```

This example demonstrates how `JSON_TABLE()` simplifies working with JSON data by converting it into a relational structure that can be queried using standard SQL, eliminating the complexity of JSON operators for many use cases. Now you can query the JSON data just like a relational data. Other than this JSON_TABLE also support the nested queries. Get full details on JSON_TABLE() from the postgres [documentation](https://www.postgresql.org/docs/current/functions-json.html#FUNCTIONS-SQLJSON-TABLE).

## Imp Postgres JSON functions

Lets now discuss some commonly used json function that will help you with your JSON data. For learning about all the available functions for json take a look at the postgresql documentation.

### **Querying JSON Data**

These functions and operators are used to access or retrieve data from JSON or JSONB columns.

- **`>`**: Retrieves a JSON object field or array element.
- **`->>`**: Retrieves a JSON object field or array element as text.
- **`#>`**: Retrieves a nested JSON sub-object or array by path.
- **`#>>`**: Retrieves a nested JSON sub-object or array by path as text.
- **`jsonb_path_query()`**: Extracts data using JSONPath expressions.

```sql
SELECT data->'key' FROM table;
SELECT data->>'key' FROM table;
SELECT data#>'{key,subkey}' FROM table;
SELECT data#>>'{key,subkey}' FROM table;
SELECT jsonb_path_query(data, '$.items[*].name') FROM table;
```

### Creating JSON

These functions help construct JSON objects and arrays dynamically.

- **`json_build_object()` / `jsonb_build_object()`**: Creates a JSON object from key-value pairs.
- **`json_build_array()` / `jsonb_build_array()`**: Creates a JSON array from a list of values.
- **`row_to_json()`**: Converts a database row into a JSON object.
- **`array_to_json()`**: Converts a PostgreSQL array into a JSON array.

```sql
SELECT json_build_object('name', 'John', 'age', 30);
SELECT json_build_array(1, 'two', true);
SELECT row_to_json(row(1, 'John', true));
SELECT array_to_json(ARRAY[1, 2, 3]);
```

### Modifying JSON

These functions allow updates or modifications to JSON data.

- **`jsonb_set()`**: Updates a JSONB value at a specified path.
- **`jsonb_set_lax()`** *(PostgreSQL 15)*: Enhanced version of `jsonb_set()` with handling for missing keys.
- **`jsonb_delete_path()`** *(PostgreSQL 15)*: Removes a specific path or key from JSONB.

```sql
SELECT jsonb_set('{"a": 1}', '{b}', '2');
SELECT jsonb_set_lax('{"a": 1}', '{b}', '2', true);
SELECT jsonb_delete_path('{"a": {"b": 1}}', '{a,b}');
```

### Validating and Formatting

- **`jsonb_pretty()`**: Formats JSONB for human-readable output.
- **`jsonb_typeof()`**: Returns the type of a JSON value (e.g., object, array, string).

```sql
SELECT jsonb_pretty('{"key": "value"}');
SELECT jsonb_typeof('{"key": "value"}');
```

### Aggregation

- **`json_agg()` / `jsonb_agg()`**: Aggregates rows into a JSON array.
- **`json_object_agg()` / `jsonb_object_agg()`**: Aggregates key-value pairs into a JSON object.

```sql
SELECT jsonb_agg(column) FROM table;
SELECT json_object_agg(key, value) FROM table;
```

### Searching

- **`jsonb_exists()`**: Checks if a key exists in JSONB.
- **`jsonb_exists_any()`**: Checks if any of the specified keys exist.
- **`jsonb_exists_all()`**: Checks if all of the specified keys exist.

```sql
SELECT jsonb_exists('{"key": "value"}', 'key');
SELECT jsonb_exists_any('{"key1": 1, "key2": 2}', ARRAY['key1', 'key3']);
SELECT jsonb_exists_all('{"key1": 1, "key2": 2}', ARRAY['key1', 'key2']);
```

## Conclusion
Postgres is keep improving its support for JSON. After Oracle and MySQL, JSON_TABLE() function is a great addition to the postgres 17. Now handling JSON data will be much easier. As JSON is used almost every where postgres will be much more friendly now. Thanks to the postgres contributors. 

## References
* https://www.percona.com/blog/json_table-will-be-in-postgresql-17/
* https://developer.ibm.com/articles/i-json-table-trs/#json_table-syntax4 
* https://www.postgresql.org/docs/17/release-17.html