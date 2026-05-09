# ChronoScope Documentation

This repository contains the customer-facing documentation website for ChronoScope. The site is built with [MkDocs](https://www.mkdocs.org/) and the [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) theme.

The documentation source files live in `docs/`. The website configuration lives in `mkdocs.yml`.

## What You Need

Install these tools before you start:

- Python 3.11 or newer
- Git

You can check whether Python is installed with:

```powershell
python --version
```

If this prints a Python version, you are good. If it says that Python was not found, install Python first and reopen your terminal.

## First-Time Setup

Open a terminal in the repository folder:

```powershell
cd C:\Users\Elias\source\vscode\ni0.team-docs
```

Create a local Python virtual environment:

```powershell
python -m venv .venv
```

Activate the virtual environment:

```powershell
Linux: .\.venv\Scripts\activate
Windows: .\.venv\Scripts\Activate.ps1
```

If PowerShell blocks the activation script, run this command once in the same terminal and try activation again:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
.\.venv\Scripts\Activate.ps1
```

Install the project dependencies:

```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

## Run the Docs Locally

Start the local documentation server:

```powershell
mkdocs serve
```

Open this URL in your browser:

```text
http://127.0.0.1:8000/
```

MkDocs watches the files while it is running. When you edit a Markdown file, refresh the browser or wait for the page to reload automatically.

Stop the local server with `Ctrl+C` in the terminal.

## Build the Static Website

Run this before pushing bigger changes:

```powershell
mkdocs build --strict
```

This checks that the documentation can be built and fails on broken links or configuration problems.

The generated website is written to `site/`. Do not edit files inside `site/` manually because they are generated output.

## Where to Edit Content

Most documentation work happens in these places:

```text
docs/index.md                         Start page
docs/getting-started.md               First steps
docs/tutorials/first-plan.md          Example tutorial
docs/concepts/                        Concept pages
docs/faq.md                           Frequently asked questions
docs/writing-guide.md                 Writing and image guidelines
mkdocs.yml                            Navigation and site configuration
docs/assets/stylesheets/brand.css     Custom visual styling
docs/assets/images/                   Screenshots, logos, and other images
```

## Add a New Page

Create a Markdown file somewhere inside `docs/`, for example:

```text
docs/tutorials/create-task.md
```

Then add it to the navigation in `mkdocs.yml`:

```yaml
nav:
  - Start: index.md
  - Loslegen:
      - getting-started.md
      - tutorials/first-plan.md
      - tutorials/create-task.md
```

Restarting `mkdocs serve` is usually not required, but if the navigation looks wrong, stop it with `Ctrl+C` and start it again.

## Add Images

Put images into this folder:

```text
docs/assets/images/
```

Use PNG for screenshots and SVG for logos or simple icons.

From a page directly inside `docs/`, reference an image like this:

```markdown
![Calendar overview](assets/images/calendar-overview.png)
```

From a page inside a subfolder like `docs/tutorials/`, go one folder up first:

```markdown
![Task list](../assets/images/task-list.png)
```

Use short, clear file names without spaces when possible:

```text
calendar-overview.png
task-list.png
create-task-dialog.png
planning-conflict.png
```

## Common Problems

### `mkdocs` Is Not Recognized

The virtual environment is probably not activated. Run:

```powershell
.\.venv\Scripts\Activate.ps1
```

Then try again:

```powershell
mkdocs serve
```

### Activation Is Blocked

Run this in the current PowerShell window:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
.\.venv\Scripts\Activate.ps1
```

This only changes the execution policy for the current terminal session.

### The Browser Does Not Update

Stop the server with `Ctrl+C` and start it again:

```powershell
mkdocs serve
```

### A Link Works Locally but Fails in Strict Build

Run:

```powershell
mkdocs build --strict
```

Then read the error message. It usually tells you exactly which file contains the broken link.

## Recommended Workflow

1. Activate the virtual environment.
2. Start `mkdocs serve`.
3. Edit Markdown files in `docs/`.
4. Check the result in the browser.
5. Run `mkdocs build --strict` before committing.
