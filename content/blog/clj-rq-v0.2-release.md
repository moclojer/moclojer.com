---
title: "Our lib `clj-rq` now generates itself dynamically"
date: 2024-08-23T01:00:00+10:00
tags: ["foss", "clj-rq", "release"]
images: ["/blog/clj-rq-now-generates-itself-dynamically.png"]
author: "<a href='https://github.com/j0suetm' alt='Josué Teodoro content author' target='_blank'>Josué Teodoro</a>"
description: "`clj-rq` functions are now dynamically generated directly from Jedis' source code."
---

![`clj-rq` now generates itself dynamically](/blog/clj-rq-now-generates-itself-dynamically.png)

A month ago, we published in our blog the thrilling first stable release of our library [clj-rq](https://github.com/moclojer/clj-rq). You can read more of this post [here](./clj-rq-is-finally-stable.md).

> For the newcomers, [clj-rq](https://github.com/moclojer/clj-rq) is a homebrewed library that wraps Jedis, being itself a simple Redis library for the Clojure ecosystem.

## About developing a wrapper library

One of the first comments that got between ourselves, the [moclojer team](https://moclojer.com/team) was how it would be complicated to update every helper function as the jedis/redis command interfaces changed throughout the years. Besides that, other important topics that got under our radar, in no specific order, were also:

- Better compatibility with existing standards;
- Ease of iteration;
- Lower the possibility of human error;
- Not overcomplicate/reinvent the wheel;

## Under the hood

This inspired us to remodel `clj-rq`, in order to dynamically generate itself, in this simplified procedure:

1. By using reflection, load the predefined methods from the Jedis library;
2. Parse these raw methods into a usable data structure;
3. Build through a macro a function that calls the Jedis equivalent one;
4. Wrap these functions into a clojure idiomatic equivalent, like how `lpush` becomes `push!` with the direction configurable through options;

## Installation

We distribute the library via [Clojars](https://clojars.org/com.moclojer/rq).

[![Clojars Project](https://img.shields.io/clojars/v/com.moclojer/rq.svg)](https://clojars.org/com.moclojer/rq)

**deps.edn:**

```clojure
com.moclojer/rq {:mvn/version "0.2.1"}
```

**project.clj:**

```clojure
[com.moclojer/rq "0.2.1]
```

> Be sure to check the latest versions incase you're reading this in the future.

## Showcase

The "source code voyagers" will be pleased to see how this was implemented [here](https://github.com/moclojer/clj-rq/blob/main/src/com/moclojer/internal/reflection.clj).

For those who lack time, here follows a simple showcase of `clj-rq`. Virtually, the lib didn't change. the way it builds and works under the hood did.

```clojure
(ns rq.example
  (:require [com.moclojer.rq :as rq]
            [com.moclojer.rq.queue :as queue]
            [com.moclojer.rq.pubsub :as pubsub]))

(def *redis-pool* (rq/create-client "redis://localhost:6379/0"))

;; queue
(queue/push! *redis-pool* "my-queue"
             ;; has to be an array of the elements to push
             [{:now (java.time.localdatetime/now)
              :foo "bar"}])

(println :size (queue/len *redis-pool* "my-queue"))
(prn :popped (queue/pop! *redis-pool* "my-queue"))

;; pub/sub
(def my-workers
  [{:channel "my-channel"
    :handler (fn [msg]
               (prn :msg :my-channel msg))}
   {:channel "my-other-channel"
    :handler (fn [{:keys [my data hello]}]
               (my-function my data hello))}])

(pubsub/subscribe! *redis-pool* my-workers)
(pubsub/publish! *redis-pool* "my-channel" "hello world")
(pubsub/publish! *redis-pool* "my-other-channel"
                 {:my "moclojer team"
                 :data "app.moclojer.com"
                 :hello "maybe you'll like this website"})

(rq/close-client *redis-pool*)
```

For a more extensive documentation of the supported commands, give a look at our [readme](https://github.com/moclojer/clj-rq/tree/main?tab=readme-ov-file#functions).

It is important to remember that this change doesn't invalidate the work of our open source engineers. There was a lot of thought and time spent to make sure our lib is delivered top-notch and ready for production. It is, nonetheless, easier to be maintained now 😃.

## Using `stuartsierra`'s component lib

If you use stuartsierra's component lib, we've also recently released our `components` bundle version `v0.1.0`. It wraps `clj-rq` and presents you with a seamless and simple integration.

### Installation

We distribute `components` via [Clojars](https://clojars.org/com.moclojer/components).

[![Clojars Project](https://img.shields.io/clojars/v/com.moclojer/components.svg)](https://clojars.org/com.moclojer/components)

**deps.edn:**

```clojure
com.moclojer/components {:mvn/version "0.1.1"}
```

**project.clj:**

```clojure
[com.moclojer/components "0.1.1"]
```

> Be sure to check the latest versions incase you're reading this in the future.

---

Thanks for reading. You can find more about moclojer, [who we are, what we do, what we eat](https://www.youtube.com/watch?v=XPxOI8YyKtw) and more at our [home page](https://moclojer.com), our [github](https://github.com/moclojer), and maybe our [product](https://app.moclojer.com?utm_source=blog&utm_medium=post&utm_campaign=clj-rq-v0.2.0).
