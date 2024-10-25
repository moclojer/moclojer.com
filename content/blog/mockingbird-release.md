---
title: "Introducing Mockingbird: The Design System for ClojureScript"
date: 2024-10-25T01:00:00+10:00
tags: ["clojure", "clojurescript", "design-system", "helix", "tailwind", "mockingbird"]
images: ["/blog/mockingbird/mockingbird-logo-full-hd.png"]
description: "We are excited to announce the release of Mockingbird, a cutting-edge ClojureScript design system built with Helix and Tailwind. In this post, you'll learn how Mockingbird can streamline your UI development and bring consistency to your projects."
author: "<a href='https://github.com/Felipe-gsilva' alt='Felipe Gomes da Silva content author' target='_blank'>Felipe Gomes</a>"
---

![mockingbird](/blog/mockingbird/mockingbird-logo-full-hd.png)

We're excited to introduce **Mockingbird**, a design system that seamlessly integrates [Tailwind CSS](https://tailwindcss.com/) and [Helix](https://github.com/lilactown/helix) into [ClojureScript](https://clojurescript.org/). Mockingbird simplifies user interface development, offering a consistent user experience without the need for excessive CSS embedded within your ClojureScript code—unless, of course, you want to! 
> This is not a stable build yet, but we want to share what the team is currently working on to make our dear moclojer-app.

## The Name Reference

"Mockingbird" isn't just a catchy name. It’s inspired by the famous novel *[To Kill a Mockingbird](https://en.wikipedia.org/wiki/To_Kill_a_Mockingbird)* by Harper Lee, symbolizing how this design system brings harmony and consistency, much like the "mockingbird" sings the same song everywhere. The name also nods to Eminem’s song *[Mockingbird](https://www.youtube.com/watch?v=S9bCLPwzSC0)*, expressing love, care, and dedication.

In that spirit, Mockingbird is our love letter to developers, providing a toolkit to enhance front-end development, even if you're more comfortable on the backend.

## Why Mockingbird?

Mockingbird combines functional programming with reusable components to create visually consistent, scalable, and fast web applications. With **Tailwind** for styling, **Helix** for React-based rendering, and **ReFx** for state management, Mockingbird offers a complete solution to modern ClojureScript UI development.

## Getting Started

To start using Mockingbird in your project, follow these steps:

### Prerequisites

Ensure you have the following installed:
- [Clojure](https://clojure.org/guides/install_clojure)
- [Node.js](https://nodejs.org/en/download/)
- [npm](https://www.npmjs.com/package/downloads)

### Installation

You can install Mockingbird from Clojars source:

**deps.edn**:
```clj
moclojer/mockingbird {:mvn/version "0.0.1"}
```

or using our npm package:
```sh 
npm i mockingbird-cljs #this npm package is a project on going. Feel free to contribute.
```

### Usage 

For npm, follow this setup:
```sh
npx create-cljs-project your-project
npm i react react-dom --save
npm i --save-dev shadow-cljs rimraf autoprefixer postcss postcss-cli tailwindcss cssnano 
```

We also provide a sample **package.json** for easier setup.

### Shadow-CLJS Configuration

To use Mockingbird, configure your **shadow-cljs.edn**:

```clj
{:deps {:aliases [:dev]}
 :builds {:app {:target :browser
                :output-dir "resources/public/assets/js"
                :asset-path "/assets/js"
                :devtools {:reload-strategy :full
                           :http-port 8080
                           :http-root "resources/public"}
                :dev {:modules {:core {:init-fn your.ns.core/init}}}}}}
```

### Rendering Components

Mockingbird components are easy to use. Here's how you can load and render a button component in your project:

```clj
(:require 
  [mockingbird.components.button :refer [button]]
  [helix.core :refer [$]])
  
($ button {:type :submit :theme :mockingbird :size :lg} "Submit")
```

## Styling
> This feature is only working for jar installation, if you are using npm, you will need to manually copy our target.css from the node_modules installation path of mockingbird

Mockingbird leverages Tailwind CSS classes. You can easily pass size, roundness, and other styles as parameters instead of manually adding CSS classes. Use the following build hook to copy the styles:

```clj
:build-hooks [(mockingbird.dev.shadow.hooks/get-target-css {:path "resources/public/assets/css/target.css"})]
```

In your HTML file, ensure the styles are properly linked:

```html
<link rel="stylesheet" href="./assets/css/target.css">
```

## Example Project Setup

Here's an example of how to render a page using Mockingbird components:

```clj
(ns your-project.core
  (:require
   ["react-dom/client" :as rdom]
   [mockingbird.examples.main :as ex]
   [helix.core :refer [$ <>]]))

(defn app []
  (<>
   ($ ex/app)))

(defonce root
  (rdom/createRoot (js/document.getElementById "app")))

(defn render []
  (.render root ($ app)))

(defn ^:export init []
  (render))
```

For more information, check out the [official Shadow-CLJS guide](https://github.com/thheller/shadow-cljs).

## NPM Usage

We also distribute Mockingbird as an npm package, though it's currently only tested with ClojureScript projects.

To install the package:
```sh
npm i mockingbird-cljs
```

Here's an example of how to use Mockingbird with npm:

```clj
(ns main.core
  (:require 
    ["mockingbird-cljs" :refer [button]]
    ["react-dom/client" :as rdom]
    [helix.core :refer [$ <>]]))

(defn app []
  (<>
    (button {:type  :text :theme :mockingbird :size  :md } "My Button")))

(defonce root
  (rdom/createRoot (js/document.getElementById "app")))

(defn render []
  (.render root ($ app)))

(defn ^:export init []
  (render))
```

## Props and Parameters

Mockingbird components support intuitive props for size, roundness, shadows, padding, and margins. Here's how you can easily apply different styles:

```clj
(:require [mockingbird.components.image :refer [pfp]])

($ pfp {:theme :mockingbird :size :lg :roundness :full})
```

### Parameter Overview:
- **Size**: `:none`, `:sm`, `:md`, `:lg`, `:xl`, `:full`
- **Roundness**: `:none`, `:sm`, `:md`, `:full`
- **Shadow**: `:none`, `:sm`, `:md`, `:lg`
- **Padding**: `:none`, `:sm`, `:md`, `:lg`
- **Margin**: `:none`, `:sm`, `:md`, `:lg`

## Contributing

Mockingbird is open-source! We welcome contributions in the form of new features, bug fixes, or suggestions. Visit our repository to contribute: [Mockingbird on GitHub](https://github.com/moclojer/mockingbird).

Mockingbird makes UI development in ClojureScript faster, more efficient, and more beautiful. Start using it today and see the difference it makes in your next project!
