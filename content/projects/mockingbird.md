---
title: "mockingbird"
date: 2023-12-03T12:00:00+10:00
featured: true
weight: 3
description: "Frontend components library for Clojure/ClojureScript applications"
github_url: "https://github.com/moclojer/mockingbird"
---

A frontend components library for Clojure/ClojureScript applications, designed to provide a consistent and beautiful UI experience.

## Features

- **React-based Components**: Built on top of React for ClojureScript applications
- **Responsive Design**: Components work across different screen sizes
- **Customizable**: Easily theme and customize components to match your brand
- **Accessible**: Built with accessibility in mind
- **Comprehensive**: Includes a wide range of UI components for various needs

## Installation

Add the dependency to your project:

```clojure
;; deps.edn
{:deps
 {com.moclojer/mockingbird {:mvn/version "latest-version"}}}

;; Leiningen/Boot
[com.moclojer/mockingbird "latest-version"]
```

## Usage Example

```clojure
(ns my-app.core
  (:require [com.moclojer.mockingbird.button :refer [button]]
            [com.moclojer.mockingbird.input :refer [text-input]]))

(defn my-form []
  [:div
   [text-input {:label "Username" :placeholder "Enter your username"}]
   [button {:variant "primary" :on-click #(js/alert "Clicked!")} "Submit"]])
```

[View on GitHub](https://github.com/moclojer/mockingbird)
