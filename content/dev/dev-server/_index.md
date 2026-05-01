---
title: Setting up a development server
weight: 10
last_reviewed: 2026-04-29
---

This guide will help you get your local development environment up and
running with Hugo, which is the open-source static site generator that
we use.

---

The Hugo development server allows you to preview your changes to the
website locally. It features _LiveReload_, meaning any changes you
make to your files will automatically refresh your browser in
real-time.

## 1. Prerequisites

Before starting, ensure you have Hugo installed on your machine.
* **Windows:** `choco install hugo-extended`
* **macOS:** `brew install hugo`
* **Linux:** `sudo apt install hugo`

*Note: It is highly recommended to use the **extended** version of Hugo for Sass/SCSS support.*

## 2. Create a fork the velociraptor-docs repo and then clone it locally

```bash
gh repo fork Velocidex/velociraptor-docs
git clone https://github.com/<your_user>/velociraptor-docs
```

## 3. Launching the Dev Server

To start the server, run the following command in your local repo's
root:

```bash
hugo server
```

During development, you can customize the server behavior using
specific flags:

- `-D` or `--buildDrafts`: Includes posts marked as drafts in the
  preview.

- `--renderToMemory`: Serves the generated pages from memory (which is
  slightly faster) instead of from the `public` folder in the repo
  root.

### What happens next?

1. Hugo builds the site and performs validation tasks such as checking
   internal links.
2. The terminal output will provide a local URL, by default:
   `http://localhost:1313/`
3. Open that URL in your browser to see preview your local copy of the
   site.
4. Make your changes and review them in the live preview.

## 4. Stopping the Server

To shut down the development server, go back to your terminal and press:
**`Ctrl + C`**

---

You can then commit your local changes, push them to your GitHub repo
and open a pull request.

