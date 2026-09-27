# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

Execute and iterate on Python data analysis scripts from Visual Studio Code.

## Directory Layout

- `code\` — Python scripts and notebooks, numbered in run order (e.g. `01.Run_commands.ipynb`, `02.Run_commands.ipynb`)
- `output\` — generated files: logs, tables, figures
- `data\` — input datasets

## Environment

`.venv` in the project root runs Python 3.14. Use it for all script execution:

```powershell
.venv\Scripts\python.exe "code\01.Run_commands.py"
```

## Running Scripts

Notebooks in `code\` are numbered (`01.Run_commands.ipynb`, `02.Run_commands.ipynb`, ...) and must be run **in sequence**, 01 first, then 02, and so on. Later notebooks may depend on output from earlier ones, so stop if a notebook fails rather than continuing to the next.

Always run from the **project root**: file paths inside notebooks are relative to it (e.g. `output/df1.parquet`, `data/...`). Do not use absolute paths such as `C:\CLAUDE\...`. For interactive runs in the VS Code notebook editor, `.vscode\settings.json` sets `"jupyter.notebookFileRoot": "${workspaceFolder}"` so the kernel also starts in the project root.

Convert each notebook to a plain script and run it, in order:

```powershell
.venv\Scripts\python.exe -m jupyter nbconvert --to script "code\01.Run_commands.ipynb"
.venv\Scripts\python.exe "code\01.Run_commands.py"
.venv\Scripts\python.exe -m jupyter nbconvert --to script "code\02.Run_commands.ipynb"
.venv\Scripts\python.exe "code\02.Run_commands.py"
```

To run every numbered notebook in order, stopping at the first failure:

```powershell
foreach ($nb in Get-ChildItem code\[0-9][0-9].*.ipynb | Sort-Object Name) {
    .venv\Scripts\python.exe -m jupyter nbconvert --to script $nb.FullName
    .venv\Scripts\python.exe ($nb.FullName -replace '\.ipynb$', '.py')
    if ($LASTEXITCODE -ne 0) { Write-Host "FAILED: $($nb.Name)"; break }
}
```

Execute a notebook with papermill (cell outputs saved to a new notebook in `output\`; these are ignored by Git and not pushed):

```powershell
.venv\Scripts\python.exe -m papermill "code\01.Run_commands.ipynb" "output\01.Run_commands_out.ipynb"
```

## Script Template

```python
# Libraries
import pandas as pd

# Commands
# ... analysis code ...

# Timestamp
import datetime
print(f"Program completed on: {datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
```

## Available Libraries

Installed in `.venv`: `pandas`, `numpy`, `requests`, `papermill`, `jupyter`, `ipykernel`, `nbformat`, `nbclient`, `tqdm`, `jsonschema`, `pyyaml`, `pyarrow` (needed for `df.to_parquet` / `pd.read_parquet`).

Note: `matplotlib` and `scipy` are **not** installed — add them with `.venv\Scripts\pip.exe install <package>` if needed.

## Git and GitHub

Remote: `https://github.com/shauns11/Claude---Project3.git` (branch `main`). The GitHub CLI (`gh`) is not installed, so use plain `git` and the GitHub website.

### Day-to-day

```powershell
git status                 # see what changed
git add .                  # stage changes
git commit -m "Message"    # commit
git push                   # upload to GitHub
```

Notes:
- Never use `git push --force` against a repository that already has history unless you intend to permanently replace it.
- Warnings like "LF will be replaced by CRLF" are Windows line-ending notices and can be ignored.
- Do not put double quotes inside a commit message in PowerShell 5.1 (e.g. `-m 'Update "What is tracked"'`): they split the message into separate arguments and the commit fails with `pathspec ... did not match`. Use single quotes or no quotes inside the message.

### What is tracked

- Tracked: `code\` (python notebooks), `CLAUDE.md`, `.gitignore`, `.vscode\settings.json` (VS Code workspace settings), and any non-ignored files in `output\` (e.g. tables, figures)
- Ignored (see `.gitignore`):
  - `*.dta` — Stata datasets, anywhere in the project
  - `*.rds` — R datasets, anywhere in the project
  - `*.txt` — log files, anywhere in the project (so logs in `output\` are not pushed)
  - `*.parquet` — Parquet datasets, anywhere in the project (e.g. `output\df1.parquet`)
  - `output/*.ipynb` — papermill output notebooks (the source notebooks in `code\` are still tracked)
  - `*.py` — Python scripts, anywhere in the project (the `.py` files made by `nbconvert` are regenerated on each run; the `.ipynb` notebooks are the source)
  - `.venv\` — the virtual environment (ignored by its own internal `.gitignore`)
- If `output\` holds only ignored files, Git skips the folder entirely; it will appear once a tracked file type (e.g. `.csv`, `.png`) is added.

### First-time setup

**Already done (2026-09-27): skip this section.** The repository is initialised, the remote is set and `main` is pushed. Re-running it would fail (`git remote add origin` errors because the remote exists, and `git ls-remote` is no longer empty). It is kept for reference only.

1. Create `.gitignore` in the project root **before** the first commit, so ignored files are never committed:

```text
# Stata datasets (anywhere in the project)
*.dta
# R datasets (anywhere in the project)
*.rds
# log files (anywhere in the project)
*.txt
# Parquet (anywhere in the project)
*.parquet
# Python scripts (anywhere in the project)
*.py
# Papermill output notebooks
output/*.ipynb
```

2. Initialise the repository, check what will and won't be committed, then commit:

```powershell
git init -b main
git add .
git status --short             # files to be committed
git status --short --ignored   # lines starting "!!" are ignored
git commit -m "Initial commit"
```

3. User will create an **empty** repository on github.com (no README, .gitignore or licence) and choose Public or Private.
4. Before pushing, confirm the remote exists and is empty. `git ls-remote` returns nothing for an empty repo and "Repository not found" if the URL is wrong, deleted or private without access:

```powershell
git ls-remote https://github.com/shauns11/Claude---Project3.git
```

5. Add the remote and push `main`:

```powershell
git remote add origin https://github.com/shauns11/Claude---Project3.git
git push -u origin main
git status -sb                 # should show: ## main...origin/main
```

6. Update the `Remote:` line at the top of the Git section.

