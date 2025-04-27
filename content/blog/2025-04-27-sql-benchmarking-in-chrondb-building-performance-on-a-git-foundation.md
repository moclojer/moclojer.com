---
title: "SQL Benchmarking in ChronDB: Building Performance on a Git Foundation"
date: 2025-04-27
tags: ["chrondb", "git database", "sql performance", "database benchmarking", "join optimization"]
images: ["/blog/sql-benchmarking-in-chrondb-building-performance-on-a-git-foundation.png"]
url: "/blog/sql-benchmarking-in-chrondb-building-performance-on-a-git-foundation"
description: "Explore how ChronDB achieves SQL performance on a Git-based database architecture through benchmarking, optimization techniques, and continuous measurement. Learn how we balance Git's versioning benefits with database query efficiency"
author: "<a href='https://github.com/avelino' alt='Thiago Avelino' target='_blank'>Avelino</a>"
---

![SQL Benchmarking in ChronDB: Building Performance on a Git Foundation](/blog/sql-benchmarking-in-chrondb-building-performance-on-a-git-foundation.png)

The [ChronDB](https://github.com/moclojer/chrondb) was born from the idea of combining the reliability and native version control of Git with the flexibility and power of a modern database. While our journey began with basic key-value operations, we quickly evolved to support complete SQL queries, including JOINs and Full-Text Search. However, a critical question remained: how to ensure adequate performance when using Git as the underlying storage mechanism?

In this post, I will explore our recent benchmarking and optimization efforts for ChronDB's SQL protocol, demonstrating how we are building a robust and performant database on the Git architecture.

> "A well-designed benchmark is one that simulates as closely as possible the behavior of the system in a production environment." — Bert Scalzo & Kevin Kline, Database Benchmarking and Stress Testing[^1]

## The Fundamental Challenge: Performance in Git-Based Databases

Git was not originally designed to be a database. It was created for source code version control, with optimizations for that specific purpose. This creates some intrinsic challenges when building a database on top of it:

1. **File access overhead**: Each document in ChronDB is stored as a `JSON` file within the Git repository
2. **Indirect cost of the commit system**: Write operations involve creating Git commits
3. **Lack of native indexes**: Git does not have an indexing system for file contents

To face these challenges, we implemented a Lucene indexing layer and developed a comprehensive benchmarking infrastructure to measure, understand, and optimize performance.

Scalzo and Kline emphasize that "unconventional database systems often require customized benchmarking methodologies, as standard industry benchmarks may not adequately capture their unique strengths and weaknesses"[^1]. This approach was crucial for developing a specific testing methodology for ChronDB's Git-based architecture.

## Benchmark Infrastructure Implementation

Our benchmark infrastructure was designed with several objectives:

1. Test performance under realistic conditions (1GB+ datasets)
2. Evaluate different types of SQL operations
3. Measure relevant metrics, including TPS (Transactions Per Second)
4. Allow consistent comparisons between different versions of ChronDB

The benchmark code includes:

```clojure
(defn- calculate-tps [operations-count elapsed-ms]
  (/ (* operations-count 1000.0) elapsed-ms))

(defn- run-select-benchmark []
  (let [out (java.io.ByteArrayOutputStream.)
        operations-count 1000
        start-time (System/currentTimeMillis)]
    (query/handle-query *test-storage* *test-index* out
                       "SELECT * FROM benchmark_items LIMIT 1000")
    (let [elapsed (- (System/currentTimeMillis) start-time)
          tps (calculate-tps operations-count elapsed)]
      (log/log-info (format "SELECT 1000 items completed in %d ms (%.2f TPS)"
                           elapsed tps))
      {:elapsed elapsed :tps tps})))
```

We followed the principle advocated by Scalzo and Kline that "a benchmark should focus on isolating and measuring specific operations to identify bottlenecks, not just the overall system performance"[^1]. Therefore, we created distinct tests for each type of SQL operation supported by ChronDB.

To simulate real-world conditions, we generated substantial test datasets:

```clojure
(defn- generate-data []
  (log/log-info "Starting benchmark data generation...")
  (let [table-name "benchmark_items"
        branch "main"]
    (fixtures/generate-benchmark-data *test-storage* *test-index*
                                     table-name num-test-docs branch)

    ;; Create additional tables for JOIN tests
    (log/log-info "Creating additional tables for JOIN tests...")
    (let [out (java.io.ByteArrayOutputStream.)]
      ;; Generate users table
      (doseq [idx (range user-limit)]
        (let [id (str (UUID/randomUUID))
              query (str "INSERT INTO users (id, name, email, age) VALUES ('"
                        id "', 'User " idx "', 'user" idx "@example.com', "
                        (+ 20 (rand-int 40)) ")")]
          (query/handle-query *test-storage* *test-index* out query)))

      ;; Generate orders table with references to items and users
      (doseq [_ (range order-limit)]
        (let [id (str (UUID/randomUUID))
              item-index (rand-int num-test-docs)
              user-index (rand-int user-limit)
              query (str "INSERT INTO orders (id, user_id, item_id, order_date, quantity) VALUES ('"
                        id "', 'user" user-index "', 'item" item-index "', '"
                        (+ 20220101 (rand-int 10000)) "', "
                        (inc (rand-int 5)) ")")]
          (query/handle-query *test-storage* *test-index* out query)))
```

## Performance Metrics and Results

Our benchmark tests include:

1. **Basic selection**: `SELECT * FROM table LIMIT n`
2. **Filtered queries**: `SELECT * FROM table WHERE condition`
3. **Simple JOIN**: INNER JOIN between two tables
4. **Complex JOIN**: LEFT JOIN with multiple tables
5. **Insertions**: Measuring document insertion speed

Instead of focusing only on raw time, we adopted TPS (Transactions Per Second) as our main metric, allowing a more objective comparison between different types of operations.

As Scalzo and Kline recommend, "it is crucial to measure both latency and throughput, as systems may optimize one at the expense of the other"[^1]. Our TPS measurement approach allows us to evaluate the processing capacity of the system under different workloads.

Typical benchmark results in our development environment:

| Operation | Average time (ms) | TPS |
|----------|------------------|-----|
| SELECT 1000 records | 115 | 8,695 |
| SELECT with WHERE | 203 | 492 |
| INNER JOIN | 587 | 170 |
| LEFT JOIN | 432 | 231 |
| INSERT | 73 | 137 |

These numbers provide a basis for evaluating performance improvements and identifying bottlenecks.

## Implemented Optimizations

Based on the benchmark results, we implemented several optimizations:

### 1. Indexing Layer Improvement

The most impactful optimization was in the Lucene integration:

```clojure
(defn search
  "Execute a full-text search on the index.
   Parameters:
   - query: The search query expression
   Returns: A sequence of document IDs matching the query"
  [index query]
  (with-open [searcher (.acquire (.createSearcher index))]
    (let [parser (create-query-parser)
          lucene-query (.parse parser query)
          top-docs (.search searcher lucene-query 100)]
      (mapv #(.get % "id")
            (map #(.doc searcher (.doc %))
                 (.scoreDocs top-docs))))))
```

Scalzo and Kline highlight that "efficient indexing is often the most significant factor in determining query performance on large datasets"[^1]. This observation guided our focus on optimizing the indexing layer.

We enhanced the full-text search to support advanced operators like `to_tsquery`, inspired by PostgreSQL:

```clojure
(defn- parse-to-tsquery
  "Convert a to_tsquery expression to a Lucene query"
  [query]
  (-> query
      (str/replace #"&" " AND ")
      (str/replace #"\|" " OR ")
      (str/replace #"!" " NOT ")))
```

### 2. JOIN Optimization

We significantly improved JOIN operations by introducing:

1. **Pre-filtering**: Apply WHERE conditions before executing joins
2. **Indexing common JOIN fields**: Speed up matching
3. **Lazy processing**: Avoid loading all results into memory

```clojure
;; JOIN optimization using hash-join strategy
(defn- optimize-join-execution
  [primary-docs secondary-docs join-condition]
  (let [;; Determine the join field for each side
        primary-field (get-in join-condition [:left :field])
        secondary-field (get-in join-condition [:right :field])

        ;; Create hash index of secondary documents
        secondary-index (reduce (fn [index doc]
                                 (let [key-val (get doc secondary-field)]
                                   (update index key-val
                                          (fn [existing]
                                            (conj (or existing []) doc)))))
                                {}
                                secondary-docs)]

    ;; For each primary document, find matches in the index
    (mapcat (fn [primary-doc]
             (let [join-value (get primary-doc primary-field)
                   matching-docs (get secondary-index join-value [])]
               (if (empty? matching-docs)
                 [] ;; No matches for INNER JOIN
                 (map #(merge primary-doc
                              (prefix-keys % secondary-table))
                      matching-docs))))
           primary-docs)))
```

In their book, Scalzo and Kline emphasize that "JOINs are often the most expensive operations in a database, and their optimization should be prioritized in the benchmarking process"[^1]. Our hash-join implementation was directly inspired by the techniques described in the book for optimizing joins in memory-constrained environments.

### 3. Storage Optimization

We improved data access with:

1. **GZIP compression**: Reduce size and improve I/O speed
2. **Batch loading**: Load documents in batches
3. **Intelligent caching**: Keep frequently accessed documents in memory

As Scalzo and Kline point out, "I/O optimizations often produce the greatest performance gains in disk-based systems"[^1]. This is particularly relevant for ChronDB, given its file-based storage in the Git repository.

## Comparison with Traditional Databases

It's important to contextualize our performance in relation to traditional databases. While ChronDB doesn't achieve the same raw query speed as systems like PostgreSQL or MongoDB for simple operations, we offer unique benefits:

1. **Complete change history**: Each change is a Git commit
2. **Easy backup and replication**: Leverages Git capabilities
3. **Point-in-time queries**: Query the state of the database at any moment

Our goal is not to outperform traditional databases in raw performance, but to offer acceptable performance while maintaining Git benefits.

> "Often, the most useful benchmark is not one that compares your system with others, but one that measures your own system against the specific requirements of your users." — Scalzo & Kline[^1]

## Continuous Benchmark Implementation

In addition to ad-hoc benchmarks during development, we implemented continuous benchmarks via [GitHub Actions](https://github.com/features/actions):

```bash
#!/bin/bash

# Script to run ChronDB SQL benchmarks
# Tests designed for benchmarking with 1GB+ data

echo "ChronDB SQL Protocol Benchmark"
echo "-----------------------------"

# Create timestamp for execution
TIMESTAMP=$(date +"%Y-%m-%d_%H-%M-%S")
RESULTS_FILE="benchmark_results_${TIMESTAMP}.txt"

# Run benchmarks
echo "Starting benchmark at $(date)"
echo "Starting benchmark at $(date)" >> $RESULTS_FILE

# Execute benchmark with Clojure CLI
JAVA_OPTS="-Xms1g -Xmx4g" clojure -M:benchmark 2>&1 | tee -a $RESULTS_FILE

echo "Benchmark completed at $(date)" >> $RESULTS_FILE
echo "Results saved to $RESULTS_FILE"
```

This allows us to track performance over time, ensuring that new features don't degrade performance.

Scalzo and Kline argue that "continuous benchmarking is an essential practice for evolving systems, as it allows identifying performance regressions shortly after their introduction"[^1]. Our integration of automated benchmarks with GitHub Actions implements exactly this principle.

## Lessons Learned

Our ChronDB optimization journey taught us several valuable lessons:

1. **Storage structure is crucial**: The Git model offers benefits but requires a well-designed abstraction layer
2. **Indexing is essential**: Without adequate indexing, queries on large datasets become unfeasible
3. **Specific JOIN strategies**: JOINs in a document-based database require optimized algorithms
4. **Constant measurement**: Continuous benchmarking is vital to maintain quality

These lessons align with what Scalzo and Kline call the "benchmark lifecycle" — an iterative process of measurement, analysis, optimization, and new measurement[^1].

## Expandability and Future

ChronDB was designed with expandability in mind from the beginning. Our modular architecture allows:

1. **Storage plugins**: Beyond Git, we can support other backends
2. **Multiple protocols**: today PostgreSQL and Redis, potentially others in the future
3. **Custom indexes**: Add specialized index types for specific use cases

In the near future, we plan to:

1. **Improve parallelization**: Execute operations in parallel for better hardware utilization
2. **Optimize batch writing**: Group write operations into single commits
3. **Add query statistics**: To help with query planning and optimization

An important recommendation from Scalzo and Kline that we intend to follow is creating "specific benchmarks for future workloads, not just current ones"[^1], ensuring that our optimizations also anticipate new use cases.

## Conclusion

Building a performant database on Git is a significant challenge, but our benchmarking and optimization efforts demonstrate that it is feasible. ChronDB offers a unique combination of version control, auditability, and powerful SQL queries, all with acceptable performance for many use cases.

Our benchmark infrastructure allows us to identify bottlenecks, implement optimizations, and measure progress objectively. As ChronDB evolves, we will continue focusing on building a solid foundation that balances Git benefits with the expected performance of a modern database.

ChronDB proves that it is possible to have the best of both worlds: the reliability and complete history of Git, with the flexibility and power of SQL queries - all with sufficient performance for real-world applications.

As Scalzo and Kline conclude in their book, "the successful benchmark is not just one that produces impressive numbers, but one that leads to tangible improvements in the end-user experience"[^1]. It is with this spirit that we continue to optimize ChronDB.

---

*This post is part of our series on ChronDB development. For more information on JOIN support implementation, check our [previous post](/blog/implementing-sql-join-support-in-chrondb-via-postgresql-protocol/).*

## References

[^1]: Scalzo, Bert & Kline, Kevin. (2012). *Database Benchmarking and Stress Testing*. Apress. [Amazon.com](https://www.amazon.com/Database-Benchmarking-Stress-Testing-Evidence-Based-ebook/dp/B07J5P2C8T/)
