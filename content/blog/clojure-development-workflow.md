---
title: "Essential tips for developing with Clojure"
date: 2024-07-24T01:00:00+10:00
tags: ["clojure", "tip", "team", "dev"]
images: ["blog/clojure-development-workflow.png"]
description: "Discover essential tips and best practices for optimizing your Clojure development workflow, focusing on command line tools and efficient coding techniques."
author: "<a href='https://github.com/Felipe-gsilva' alt='Felipe Gomes da Silva content author' target='_blank'>Felipe Gomes</a>"
---

![Clojure Development Workflow Tips](/blog/clojure-development-workflow.png?width=50%)

We'd like to share our general development workflow and how we work with clojure, mainly on CLI. As a great team, we all share the tasteful and meaningful life of using `Linux` (although some part of the crew uses macOS, we will talk mainly about the penguin system because it is just better). So, we decided to share the way we use our linux machines to test clojure while coding.


## nREPL on runtime
A common practice you must have while developing Clojure applications is to serve your code editor with your [nREPL](https://github.com/nrepl/nREPL) and test all your functions while writing your code.
In order to accomplish that, we must set some things up.

You will need to run nREPL on your clojure project and there are a few ways to deal with it.

> if you use emacs, jump to [emacs section](#Emacs).

`deps.edn`:

```clj
;; ...
{:aliases {:nrepl {:extra-deps {cider/cider-nrepl {:mvn/version "0.30.0"}}
          :main-opts ["-m" "nrepl.cmdline" "--middleware" "[cider.nrepl/cider-middleware]"]}}}
;; ...
```

this way, you will be able to run a nREPL with `clj -M:nrepl`

and `leiningen`:
``` bash
lein repl :headless :host some-host :port 1234
```
run this in your shell and it open a repl on port *1234*, as simple as possible.

## Neovim

To evaluate all code in [Neovim](https://neovim.io/), we will use [Conjure](https://github.com/Olical/conjure), which works as a customized environment for executing Clojure code during nvim runtime.

> To install Conjure, follow step-by-step the [official documentation](https://github.com/Olical/conjure?tab=readme-ov-file#installation).

### Setting up the environment

We can create a simple remap, defining a mapleader, in this case, the `space`, and the necessary binds in normal mode.

```lua
-- defining the map leader
vim.g.mapleader = " "

-- eval a single function
vim.keymap.set("n", "<leader>ee", vim.cmd.ConjureEval)

-- eval an entire file
vim.keymap.set("n", "<leader>ef", vim.cmd.ConjureEvalFile)

-- Default Conjure settings
-- Disable doc mapping
vim.g["conjure#mapping#doc_word"] = false

-- Remap from K to <prefix>gk
vim.g["conjure#mapping#doc_word"] = "gk"

-- Reset to the default unprefixed K (note the special table syntax)
vim.g["conjure#mapping#doc_word"] = {"K"}
```

Done, now just restart Nvim and use the defined functions alongside your nREPL.

## Emacs

The development process using Emacs is just as simple. You only need to have the [`cider`](https://github.com/clojure-emacs/cider) package installed. The [`clojure-lsp`](https://clojure-lsp.io) and `clojure-mode` packages are technically optional but also help in development.

It is recommended, as we aim to follow TDD methodology, to enable the `auto-test` mode of cider. This way, when you evaluate a file or group of files, cider will automatically re-evaluate and run any related tests.

```lisp
(add-hook 'clojure-mode-hook #'cider-auto-test-mode)
```

The default keybindings are:

- **Open Cider nREPL**: `C-c M-j`
- **Open editable Cider nREPL**: `C-u C-c C-x j j`
- **Evaluate Form**: `C-x C-e`
- **Evaluate File**: `C-c C-l`
- **Evaluate Folder Recursively**: `C-c M-l`

> and you are now done, you can sit back, relax and enjoy your hacker lifestyle while coding in Clojure just like we do.

## Extra

- You can find our latest devlogs and some other content on [Moclojer.com](https://www.moclojer.com/blog/) 

- Want to go a step ahead on neovim? try using the [paredit-nvim](https://github.com/julienvincent/nvim-paredit) plugin.

- Also, if you are starting on clojure, follow the [official spec](https://clojure.org/guides/install_clojure) and maybe give a try to [stuartsierra's](https://www.youtube.com/watch?v=13cmHf_kt-Q) video on component pattern.

That's all, folks! Thanks for reading.
