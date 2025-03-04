---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
featured: false
weight: 100
description: ""
github_url: "https://github.com/moclojer/{{ .Name }}"
docs_url: ""
---

Short description of the project.

## Features

- **Feature 1**: Description of feature 1
- **Feature 2**: Description of feature 2
- **Feature 3**: Description of feature 3
- **Feature 4**: Description of feature 4
- **Feature 5**: Description of feature 5

## Installation

Add the dependency to your project:

```clojure
;; deps.edn
{:deps
 {com.moclojer/{{ .Name }} {:mvn/version "latest-version"}}}

;; Leiningen/Boot
[com.moclojer/{{ .Name }} "latest-version"]
```

## Usage Example

```clojure
(ns my-app.core
  (:require [com.moclojer.{{ .Name }}.core :as {{ .Name }}]))

;; Example code here
```

[View on GitHub]({{ .Params.github_url }}){{ if .Params.docs_url }} | [Documentation]({{ .Params.docs_url }}){{ end }}
