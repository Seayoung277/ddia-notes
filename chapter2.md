# Chapter 2

## Relational Model VS Document Model

Driving forces behind the adoption of NoSQL:
- Greater scalability (large datasets and high write throughput)
- Preference for free and open source software over commercial databases
- Specialized query operations not supported by SQL
- Restrictiveness of relational schemas and desire of more dynamic and expressive data model
- Relational database usually requires an extra ORM layer between database and application codes

Document model might seems closes the gap between storage layer and application codes, but since most document models are schemaless, it still has encoding issues

### One-to-many

Approaches to handle one-to-many relationships in relational model:
- Create separate tables and use foreign key
- Advanced datatype support from modern relational databases (e.g. JSON)
- Encode data into a JSON or XML format and store it as plain text, and let application codes to handle it

One-to-many relationship can be easily handled with document model by JSON/XML tree structure

### Many-to-one and Many-to-many

Reasons to normalize many-to-one relationships:
- Consistent style and spelling across profiles
- Avoiding ambiguity
- Ease of updating
- Localization support
- Better for searching

Handling many-to-one/many-to-many relationship is easy with relational model with separate tables and foreign keys

Most document databases do not support join operation, thus many-to-one/many-to-many has to be handled in application layer

### How to Choose


#### Relational Model

- Pros:
    - Better support for join and many-to-one/many-to-many relations
    - Strict schema, schema on write
- Cons:
    - Schema upgrade might be slow and might requires down time
    - Even with query optimizer and indexing, frequent multi-way joins could be expensive

#### Document Model

- Pros:
    - Schema flexibility, schema on read
    - Better performance due to locality
    - Closer to application layer
    - Easier for data locality for frequently used documents

- Cons:
    - Cannot refer directly to nested item
    - Huge documents seriously impact performance
