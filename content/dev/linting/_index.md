---
title: "Linting with Vale"
menutitle: "Linting"
date: 2026-05-01
draft: false
weight: 50
last_reviewed: 2026-05-01
---

## What is Vale?

https://vale.sh/docs

> Vale is a command-line tool that brings code-like linting to prose.
> Vale is cross-platform (Windows, macOS, and Linux), written in Go,
> and available on GitHub.

## Vale files and folders

By cloning the repo you will already have the files and folders
necessary to run the Vale tests. All you need to do is ensure that you
have the Vale binary in your path on your computer so that you can
actually run the tests.

`.vale.ini` is the key file, located in the project root.

This file defines which file targets to lint, what linting rules to
apply, and the overall alert threshold that should be reported.

The `styles` **folder**, which in `.github/` in the project root,
contains the linting rules (each defined as YAML - much like
Velociraptor artifacts) plus support files such as dictionaries and
"views" definitions that the rules need to operate. The folder
structure we use is
[as recommended by Vale](https://github.com/vale-cli/vale-action#repository-structure).


## Running Vale locally

1. Install the latest `vale` binary into your path. Test it by running
   `vale --ls-dirs` or `vale version`.

   NOTE: despite what their documentation says you DO NOT need to run
   `vale sync`. We don't use external styles and therefore don't need
   anything outside of our own repo. Everything you need, except for
   the Vale binary, is already in the repo.

2. `cd` to the project root - when you run Vale it will look for the
   `.vale.ini` file in the `CWD`, which in turn will tell Vale where
   to find the Vale `styles` directory (which is in `.github/`).

3. Run `vale .` is all you need to do to perform all the checks.

