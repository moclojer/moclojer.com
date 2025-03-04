---
title: "codemirror-cljs"
date: 2023-12-03T12:00:00+10:00
featured: true
weight: 5
description: "A ClojureScript wrapper for CodeMirror"
github_url: "https://github.com/moclojer/codemirror-cljs"
---

A ClojureScript wrapper for CodeMirror, making it easy to integrate this powerful code editor into your ClojureScript applications.

## Features

- **ClojureScript Integration**: Use CodeMirror seamlessly in ClojureScript projects
- **Syntax Highlighting**: Support for various programming languages
- **Customizable**: Configure the editor to match your needs
- **Reactive**: Works well with React-based frameworks
- **Lightweight**: Minimal overhead on top of CodeMirror

## Installation

Add the dependency to your project:

```clojure
;; deps.edn
{:deps
 {com.moclojer/codemirror-cljs {:mvn/version "latest-version"}}}

;; Leiningen/Boot
[com.moclojer/codemirror-cljs "latest-version"]
```

## Usage Example

```clojure
(ns my-app.core
  (:require [com.moclojer.codemirror-cljs.core :as cm]))

(defn code-editor []
  [cm/editor
   {:value "(defn hello [name]\n  (str \"Hello, \" name \"!\"))"
    :options {:mode "clojure"
              :theme "dracula"
              :lineNumbers true}
    :on-change #(js/console.log "Code changed:" %)}])
```

[View on GitHub](https://github.com/moclojer/codemirror-cljs)
