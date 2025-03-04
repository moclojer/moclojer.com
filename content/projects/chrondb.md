---
title: "chrondb"
date: 2023-12-03T12:00:00+10:00
featured: true
weight: 4
description: "Chronological key/value Database storing based on database-shaped git (core) architecture"
github_url: "https://github.com/moclojer/chrondb"
---

Chronological key/value Database storing based on database-shaped git (core) architecture. ChronDB provides a git-like versioning system for your data, allowing you to track changes over time.

## Features

- **Git-like Versioning**: Track all changes to your data over time
- **Key/Value Store**: Simple and flexible data storage model
- **Time Travel**: Access historical versions of your data
- **Clojure Native**: Built specifically for Clojure applications
- **Lightweight**: Minimal overhead and dependencies

## Installation

Add the dependency to your project:

```clojure
;; deps.edn
{:deps
 {com.moclojer/chrondb {:mvn/version "latest-version"}}}

;; Leiningen/Boot
[com.moclojer/chrondb "latest-version"]
```

## Usage Example

```clojure
(ns my-app.core
  (:require [com.moclojer.chrondb.core :as chrondb]))

;; Create a new database
(def db (chrondb/create-db "my-database"))

;; Store data
(chrondb/put! db "user:123" {:name "John" :email "john@example.com"})

;; Retrieve current data
(chrondb/get db "user:123")
;; => {:name "John" :email "john@example.com"}

;; Update data
(chrondb/put! db "user:123" {:name "John" :email "john.doe@example.com"})

;; Get historical version (first commit)
(chrondb/get-at db "user:123" 1)
;; => {:name "John" :email "john@example.com"}
```

[View on GitHub](https://github.com/moclojer/chrondb)
