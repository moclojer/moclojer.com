---
title: "clj-rq"
date: 2023-12-03T12:00:00+10:00
featured: true
weight: 2
description: "RQ (Redis Queue) is a simple Clojure package for queueing jobs and processing them in the background with workers"
github_url: "https://github.com/moclojer/clj-rq"
---

RQ (Redis Queue) is a simple Clojure package for queueing jobs and processing them in the background with workers. It is backed by Redis and is designed to have a low barrier to entry.

## Features

- **Simple**: Easy to use with a clean API
- **Redis-backed**: Leverages Redis for reliable queue management
- **Background Processing**: Process jobs asynchronously with workers
- **Clojure Native**: Built specifically for Clojure applications
- **Low Overhead**: Minimal impact on your application's performance

## Installation

Add the dependency to your project:

```clojure
;; deps.edn
{:deps
 {com.moclojer/clj-rq {:mvn/version "0.2.0"}}}

;; Leiningen/Boot
[com.moclojer/clj-rq "0.2.0"]
```

## Usage Example

```clojure
(ns my-app.core
  (:require [com.moclojer.clj-rq.core :as rq]))

;; Define a job function
(defn process-data [data]
  (println "Processing:" data))

;; Enqueue a job
(rq/enqueue :default process-data {:user-id 123})

;; Start a worker
(rq/start-worker :default)
```

[View on GitHub](https://github.com/moclojer/clj-rq)
