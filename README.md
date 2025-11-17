# UV: Step-by-Step Tutorial — fast, practical, and ready to use

Below is a concise, hands-by-step guide that follows Corey Schafer’s tutorial but formatted like an actionable walkthrough. Run the commands in your terminal (macOS/Linux/WSL) or PowerShell on Windows, adjusting paths as needed.

---

## 1) Install UV

Choose one install method. Examples:

Homebrew (macOS):

```bash
brew install uv
```

macOS / Linux (curl installer):

```bash
curl -sSf https://install.astral.sh | sh
```

Windows (PowerShell):

```powershell
iwr -useb https://install.astral.sh | iex
```

Verify:

```bash
uv --version
uv help
```

---

## 2) Create a new project (`uv init`)

Create a project scaffold (default type is `app`):

```bash
uv init myproject
cd myproject
```

What this creates:

* `.git/` and `.gitignore`
* `python-version` (project Python)
* `pyproject.toml` (declares dependencies)
* `main.py` starter file
* `uv.lock` will be created later when you add deps

Tip: If you already have a directory, `cd` into it and run `uv init` (no name) to convert it.

---

## 3) Add dependencies (`uv add`)

Add packages the UV way (auto-manages venv and lockfile):

```bash
uv add flask requests
```

What happens:

* UV creates `.venv/` (if needed) and selects an interpreter
* Installs requested packages into the project environment
* Updates `pyproject.toml`
* Generates/updates `uv.lock` with exact versions (reproducible)

Quick check:

```bash
uv tree             # shows dependency tree
cat pyproject.toml  # see declared deps
cat uv.lock         # see resolved exact versions
```

---

## 4) Run your app (`uv run`)

Run code without activating venv manually:

```bash
uv run python main.py
# or
uv run main.py
```

UV automatically uses the project `.venv/`. If `.venv/` is missing, UV will recreate and install from `uv.lock`/`pyproject.toml` as needed.

---

## 5) Recreate environment on another machine (`uv sync`)

If you share the project (no `.venv/`), instruct others to:

```bash
git clone <repo>
cd repo
uv sync
```

`uv sync` creates the venv and installs exact versions from `uv.lock`. Fast, reproducible setup.

---

## 6) Remove a dependency

To remove:

```bash
uv remove flask
# then
uv sync     # if you want to rebuild environment locally
```

UV updates `pyproject.toml` and `uv.lock`.

---

## 7) Use pip-compatible commands while migrating (optional)

If you want to keep using pip commands but benefit from UV’s environment handling:

```bash
uv pip install numpy
uv pip freeze > requirements.txt   # legacy-style freeze if needed
uv pip list
```

Note: `uv pip` mirrors pip behavior but won’t create `pyproject.toml`/`uv.lock` automatically. Prefer `uv add` for full UV workflow.

---

## 8) Migrate from requirements.txt

If you have an old `requirements.txt`:

```bash
uv init                # converts project to UV structure
uv add -r requirements.txt
# or:
uv pip install -r requirements.txt  # less preferred
uv sync
```

After migration you can remove `requirements.txt` and rely on `pyproject.toml` + `uv.lock`.

---

## 9) Install global CLI tools (pipx replacement)

Install a global tool (isolated but on PATH):

```bash
uv tool install ruff
which ruff   # verifies it's available on PATH
```

Run a tool temporarily without installing:

```bash
uv tool run ruff check .
# shortcut
uvx ruff check .
```

`uv tool run` installs into a temporary environment, runs the tool, then cleans up. `uvx` is the shortcut.

List / uninstall / upgrade tools:

```bash
uv tool list
uv tool uninstall ruff
uv tool upgrade --all
```

---

## 10) Inspect dependencies & disk savings

* `uv tree` — visualizes dependency graph.
* UV caches packages globally and reuses them across projects to save download time and disk space (fast reinstalls).

---

## 11) Useful commands summary

```text
uv init <name>           # initialize project
uv add <pkg> [...]       # add deps (creates .venv and lockfile)
uv remove <pkg>          # remove dependency
uv run <script>          # run code using project environment
uv sync                  # recreate environment from uv.lock
uv tree                  # show dependency tree
uv pip <...>             # pip-compatible subcommands
uv tool install <tool>   # install global CLI tools
uv tool run <tool>       # run a tool transiently
uvx <tool> <args>        # shortcut for uv tool run
```

---

## 12) Troubleshooting & tips

* If UV can’t find the right Python interpreter: set `python-version` file or install the appropriate Python and re-run.
* Want consistent Python across machines? Commit `python-version` to VCS.
* Keep `uv.lock` committed for true reproducibility.
* To speed up rebuilds on CI, cache UV’s global package cache (CI docs will show cache path; check `uv cache` help).
* To inspect what changed in lockfile: use `git diff uv.lock`.

---

## 13) Recommended beginner workflow

1. `uv init myproject`
2. `uv add <packages>` while developing
3. `git add pyproject.toml uv.lock` and commit
4. Collaborator: `git clone ... && uv sync`
5. For global tools: `uv tool install <tool>` or `uvx` for one-off use

---

## 14) When to stick with `uv pip`

If you’re converting slowly or working in an environment tightly coupled to old tooling, `uv pip` lets you keep familiar pip commands while UV manages environments; but migrating to `uv add`/`uv sync` gives full reproducibility and speed gains.

---

## **💡 Dev Note: Resetting a Broken UV Environment / Kernel**

**Problem:**

* UV environment not working, kernel not initializing, `main.py` fails.

**Quick Fix Steps:**

1. **Delete the broken environment**

```bash
cd /path/to/project
rm -rf .venv
```

2. **Rebuild environment with UV**

```bash
uv sync
```

* Recreates `.venv` and installs all dependencies from `uv.lock` / `pyproject.toml`.

3. **Add missing dependencies (if needed)**

```bash
uv add <package_name>
```

* Updates both `pyproject.toml` and `uv.lock`.

4. **Run your script using UV**

```bash
uv run main.py
```

* Automatically uses the new `.venv`.

5. **Connect Jupyter notebook to the UV environment**

```bash
uv run python -m ipykernel install --user --name coffee-uv --display-name "Python (coffee-UV)"
```

* Select **Kernel → Python** in your notebook.

6. **Verify environment**

```bash
uv run python --version
uv run which python
```

* Confirms `.venv` is being used.

---

**Tip:**

* **Do not delete `pyproject.toml` or `uv.lock`** — these files define your project environment.
* Use `uv run <script>` instead of activating `.venv` manually; UV handles the environment automatically.

