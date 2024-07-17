---
title: "Our lib `clj-rq` is finally stable"
date: 2024-07-16T01:00:00+10:00
tags: ["foss", "clj-rq", "release", "redis"]
images: ["/blog/our-lib-clj-rq-is-finally-stable.png"]
description: "We have recently released the latest stable version for our lib `clj-rq`."
author: "<a href='https://github.com/j0suetm' alt='Josué Teodoro content author' target='_blank'>Josué Teodoro</a>"
---

![Our lib `clj-rq` is finally stable](/blog/our-lib-clj-rq-is-finally-stable.png?width=50%)

We are thrilled to announce that we've hit an important milestone for `clj-rq`, an in-house library of ours that has been developed and implemented in our services for the past couple of months and is finally being used and tested in production. The current version [`v0.1.4`](https://github.com/moclojer/clj-rq/releases/tag/v0.1.4) brings implementations intrinsic to both queue and pubsub mechanisms that weren't available in the past versions.

## What's `clj-rq`

In case you don't know `clj-rq` yet, it's described, in verbatim from its [Github Page](https://github.com/moclojer/clj-rq) as follows:

> RQ (Redis Queue) is a simple Clojure package for queueing jobs and processing them in the background with workers. It is backed by Redis and it is designed to have a low barrier to entry.

We began developing it back in May, as an in-house alternative to the currently existing Redis Clojure libraries and wrappers, which were causing significant runtime errors.

> We eventually reached out to `carmine`, one of the libraries that weren't working for us, and reported the issues, which actually got solved.

It's currently being used in our services, and it has been working greatly. Overall, the main goal of the library, if not already understood through the given description, is to be a simple Clojure wrapper over Redis' Jedis. We haven't reached 100% coverage yet, but we're working on it.

## Installation

We distribute the library via [Clojars](https://clojars.org/com.moclojer/rq).

[![Clojars Project](https://img.shields.io/clojars/v/com.moclojer/rq.svg)](https://clojars.org/com.moclojer/rq)

```edn
com.moclojer/rq {:mvn/version "0.1.4"}
```

```clojure
[com.moclojer/rq "0.1.4"]
```

> Be sure to check the latest versions incase you're reading this in the future.

## Walkthrough

The following examples summarize how `clj-rq`'s API can be used.

```clojure
(ns rq.example
  (:require [com.moclojer.rq :as rq]
            [com.moclojer.rq.queue :as queue]
            [com.moclojer.rq.pubsub :as pubsub]))

;; handling clients
(def client (rq/create-client "redis://localhost:6379/0"))
(rq/close-client client)
```

### Queue

```clojure
;; pushing 
(queue/push! client "my-queue" {:now (java.time.LocalDateTime/now)
                                      :foo "bar"})

(println :size (queue/llen client "my-queue"))
(prn :popped (queue/pop! client "my-queue"))
```

### Pub/Sub

```clojure
;; pub/sub
(def my-workers
  [{:channel "my-channel"
    :handler (fn [msg]
               (prn :msg :my-channel msg))}
   {:channel "my-other-channel"
    :handler (fn [{:keys [my data hello]}]
               (my-function my data hello))}])

(pubsub/subscribe! client my-workers)
(pubsub/publish! client "my-channel" "hello world")
(pubsub/publish! client "my-other-channel" {:my "moclojer team"
                                            :data "app.moclojer.com"
                                            :hello "maybe you'll like this website"})


```

---

Operations by default follow the Queue order (I mean, look at the name of the library). However, you can pass an option that changes that behaviour.

```clojure
(queue/push! client "my-queue" :direction :r)
(queue/pop! client "my-queue" :direction :r)
```

## Using `stuartsierra`'s component lib

If you use stuartsierra's component lib, we've also recently released our `components` bundle version `v0.1.0`. It wraps `clj-rq` and presents you with a seamless and simple integration.

### Installation

We distribute `components` via [Clojars](https://clojars.org/com.moclojer/components).

[![Clojars Project](https://img.shields.io/clojars/v/com.moclojer/components.svg)](https://clojars.org/com.moclojer/components)

```edn
com.moclojer/components {:mvn/version "0.1.0"}
```

```clojure
[com.moclojer/components "0.1.0"]
```

> Be sure to check the latest versions incase you're reading this in the future.
