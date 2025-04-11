---
title: "Implementing SQL JOIN Support in ChronDB via PostgreSQL Protocol"
date: 2025-04-11
tags: ["sql join", "chrondb", "database", "git database", "postgres protocol"]
images: ["/blog/implementing-sql-join-support-in-chrondb-via-postgresql-protocol.png"]
url: "/blog/implementing-sql-join-support-in-chrondb-via-postgresql-protocol"
description: "Learn how ChronDB, a Git-based database, now supports SQL JOIN operations through PostgreSQL protocol. This technical deep dive covers INNER JOIN and LEFT JOIN implementation, performance optimizations, and how to query related data across tables in a version-controlled database environment."
author: "<a href='https://github.com/avelino' alt='Thiago Avelino' target='_blank'>Avelino</a>"
---

![SQL JOIN visualization](/blog/implementing-sql-join-in-chrondb.png)

SQL JOIN operations are a fundamental part of any robust database system, allowing developers to combine data from multiple tables based on related columns. In this blog post, we'll explore how we implemented INNER JOIN and LEFT JOIN support in ChronDB, our Git-based chronological database with PostgreSQL protocol compatibility.

## Background: ChronDB Architecture

Before diving into the JOIN implementation, let's briefly review ChronDB's architecture. ChronDB is a chronological key-value database built on Git's internal structure, providing versioning capabilities by default. We've implemented a PostgreSQL-compatible protocol layer that allows clients to connect using standard PostgreSQL drivers and execute SQL queries.

The database uses Git repositories as the storage engine:

- **Git repository:** database
- **Git branch:** schema
- **Directory in the repository:** table
- **JSON files:** documents (records)

## The Challenge: Implementing JOIN Operations

JOIN operations are complex because they require:

1. **Parsing and understanding the JOIN syntax in SQL queries**
2. Finding the related records across different tables
3. Combining the records based on join conditions
4. Handling different types of joins (INNER, LEFT, RIGHT)
5. Supporting filtering and projection on joined data

Our implementation in [PR #24](https://github.com/moclojer/chrondb/pull/24) focused on supporting the most common types: INNER JOIN and LEFT JOIN.

## Implementation Approach

### 1. SQL Parsing Enhancements

The first step was improving our SQL parser to recognize JOIN clauses. This required:

- Adding JOIN type detection in the tokenizer
- Implementing join condition parsing with support for table.field format
- Updating the SELECT query parser to handle JOIN clauses

### 2. Data Retrieval and Join Execution

The core of the implementation is in the query execution logic:

```clojure
;; Pseudocode for the JOIN implementation
(defn handle-select [index storage query branch]
  (let [primary-table (:from query)
        join-clauses (:join query)
        primary-docs (get-docs-from-table index storage primary-table branch)]

    (if (empty? join-clauses)
      ;; Regular select without joins
      (filter-and-project primary-docs (:where query) (:select query))

      ;; Handle joins
      (let [joined-docs
            (for [primary-doc primary-docs
                  :let [result-doc (process-joins primary-doc join-clauses index storage branch)]]
              result-doc)]
        (filter-and-project joined-docs (:where query) (:select query))))))

(defn process-joins [primary-doc join-clauses index storage branch]
  (reduce
    (fn [doc join-clause]
      (let [join-type (:type join-clause)  ;; "INNER" or "LEFT"
            join-table (:table join-clause)
            join-condition (:condition join-clause)

            ;; Find matching documents in the secondary table
            matching-docs (find-matching-docs doc join-table join-condition index storage branch)]

        (case join-type
          "INNER" (when (seq matching-docs)
                    (merge-docs doc (first matching-docs) join-table))
          "LEFT"  (if (seq matching-docs)
                    (merge-docs doc (first matching-docs) join-table)
                    (merge-with-nulls doc join-table))
          ;; Default case - no join
          doc)))
    primary-doc
    join-clauses))
```

### 3. Join Condition Evaluation

The join condition parser supports the standard syntax:

```sql
SELECT * FROM users INNER JOIN orders ON users.id = orders.user_id
```

We parse this into a condition structure that can be evaluated against documents:

```clojure
(defn parse-join-condition [condition-tokens]
  (let [left-side (parse-field (first condition-tokens))
        operator (second condition-tokens)
        right-side (parse-field (nth condition-tokens 2))]
    {:left left-side
     :operator operator
     :right right-side}))

(defn evaluate-join-condition [primary-doc secondary-doc condition]
  (let [left-value (get-nested-value primary-doc (:left condition))
        right-value (get-nested-value secondary-doc (:right condition))
        operator (:operator condition)]
    (case operator
      "=" (= left-value right-value)
      "<>" (not= left-value right-value)
      ;; Handle other operators
      )))
```

### 4. Full-Text Search Integration

We also improved the full-text search capability to work seamlessly with JOINs:

```clojure
(defn- fts-get-matching-docs
  "Find documents matching FTS condition"
  [index storage fts-condition branch]
  (let [search-terms (extract-search-terms fts-condition)
        normalized-terms (normalize-search-terms search-terms)]
    (try
      (let [matching-ids (index/search index normalized-terms)]
        (when (seq matching-ids)
          (log/debug "Found" (count matching-ids) "matching documents for terms" search-terms)
          (storage/get-documents storage matching-ids branch)))
      (catch Exception e
        (log/error "Error in FTS search:" (.getMessage e))
        []))))
```

## Implementation Challenges

### 1. Table and Field Name Resolution

One challenge was handling table-qualified field names. When referencing `users.name`, we needed to distinguish this from the `name` field in other tables.

Our solution was to prefix fields from joined tables with the table name in the result set:

```clojure
(defn merge-docs [primary-doc secondary-doc join-table]
  (let [secondary-map (into {}
                       (keep (fn [[k v]]
                               (when (not= k :_table)
                                 [(keyword (str join-table "." (name k))) v])))
                       secondary-doc)]
    (merge primary-map secondary-map)))
```

### 2. LEFT JOIN Null Handling

For LEFT JOINs, we needed to handle cases where no matching records exist in the secondary table. We solved this by creating a map with null values for all fields in the secondary table:

```clojure
(defn create-null-secondary-map [join-table secondary-fields]
  (into {}
    (map (fn [field]
           [(keyword (str join-table "." (name field))) nil]))
    secondary-fields))
```

### 3. Multiple Matches Handling

In real databases, a JOIN might match multiple records in the secondary table. Our initial implementation only used the first match, but we later enhanced it to handle multiple matches correctly:

```clojure
(if (seq matching-secondary-docs)
  ;; For matches, merge primary with each secondary doc
  (map (fn [sec-doc]
         (let [secondary-map (into {}
                               (keep (fn [[k v]]
                                       (when (not= k :_table)
                                         [(keyword (str join-table "." (name k))) v])))
                               sec-doc)]
           (merge primary-map secondary-map)))
       matching-secondary-docs)
  ;; For non-matches in LEFT JOIN, merge with nulls
  [(merge primary-map null-secondary-map)])
```

## Performance Considerations

JOIN operations can be expensive, especially with large datasets. To optimize performance:

1. We implemented indexes for commonly joined fields
2. Added logging to help diagnose slow queries
3. Limited the result set size for joined queries to prevent memory issues
4. Used lazy sequences where possible to avoid loading all data at once

## Testing the Implementation

We wrote comprehensive tests to verify the JOIN functionality:

```sql
-- Basic INNER JOIN test
SELECT users.name, orders.amount
FROM users
INNER JOIN orders ON users.id = orders.user_id
WHERE orders.amount > 100

-- LEFT JOIN test with nulls
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders ON users.id = orders.user_id

-- Multi-table join
SELECT users.name, orders.id, items.name
FROM users
INNER JOIN orders ON users.id = orders.user_id
INNER JOIN items ON orders.item_id = items.id
```

## Conclusion

Implementing JOIN support in ChronDB significantly enhances its capabilities as a database system. While our Git-based storage is primarily designed for document storage *(example json)*, the addition of JOIN operations brings relational capabilities that many applications require.

The implementation leverages Clojure's strengths in data processing and transformation while maintaining the core benefits of ChronDB - versioning, auditability, and a simple data model.

In future updates, we plan to:

1. Add support for FULL OUTER JOINs
2. Optimize JOIN performance with better indexing
3. Support more complex join conditions with AND/OR operators
4. Implement aggregate functions across joined tables

We're excited about how this enhancement makes ChronDB more versatile while maintaining its unique chronological, Git-based architecture.

## References

- [ChronDB GitHub Repository](https://github.com/moclojer/chrondb)
- [PostgreSQL JOIN Documentation](https://www.postgresql.org/docs/current/tutorial-join.html)
- [Git as a Database, about ChronDB](/blog/git-as-database-harnessing-hidden-power-internals-chronological-data-storage)
