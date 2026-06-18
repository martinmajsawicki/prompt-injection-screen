# SETUP — install & configure (for a human *or* an AI agent)

This is a step-by-step recipe. A non-technical user can follow it. An AI agent (e.g. Claude
Code) can **execute it autonomously** — and should, if a user says "set up this tool for me".

The only thing an agent **cannot** do for the user is obtain the API key (step 3). Everything
else — clone, virtualenv, dependencies, the optional skill/hooks, verification — an agent can do.

---

## 0. Prerequisites

- **Python 3.9+** and **git**. Check: `python3 --version` and `git --version`.
- A **free OpenRouter API key** for the smart (Layer 1) checks — get one at
  <https://openrouter.ai/keys>. *(Optional: without it, the tool still runs the deterministic
  Layer 0; you just don't get the semantic layer.)*

## 1. Get the code

```bash
git clone https://github.com/martinmajsawicki/prompt-injection-screen.git
cd prompt-injection-screen
```

## 2. Create the environment and install dependencies

```bash
python3 -m venv .venv
./.venv/bin/python -m pip install -r requirements.txt
```

## 3. Configure the API key — pick the row that matches how you run Claude

> macOS note: desktop apps launched from the Dock do **not** read `~/.zshrc`. Choose accordingly.

| How you use it | Where to put `OPENROUTER_API_KEY` |
|---|---|
| **Claude desktop app (Cowork / Code)** | In-app environment editor (prompt box → environment dropdown → **Local** → gear) **or** the `"env"` block of `~/.claude/settings.json`. |
| **Claude Code CLI (from a terminal)** | `export OPENROUTER_API_KEY="sk-or-..."` in `~/.zshrc` (or `~/.bashrc`), then open a new terminal. |
| **Plain command-line use of `scan.py`** | `export OPENROUTER_API_KEY="sk-or-..."` in your shell before running. |

Example `~/.claude/settings.json` snippet (works for CLI **and** desktop):
```json
{ "env": { "OPENROUTER_API_KEY": "sk-or-...", "OPENROUTER_MODEL": "anthropic/claude-haiku-4.5" } }
```
**Never commit the key.** It is git-ignored here, but keep it out of any file you push.

## 4. (Optional) Wire it into Claude Code

- **Skill** `/screen`: copy `examples/SKILL.md` → `~/.claude/skills/screen/SKILL.md`, then edit the
  two paths inside to point at this folder.
- **Hooks** (auto-screen `Read`, `WebFetch`, `WebSearch`): copy the blocks from
  `examples/settings.hooks.json` into `~/.claude/settings.json` and set the absolute paths to this
  folder's `.venv/bin/python` and `hooks/screen-hook.py`. Restart the app/session to load them.

## 5. Verify it works

```bash
./.venv/bin/python tests/run_all.py        # expect: WYNIK: 31/31 przeszło
```
Then a real scan (if you set a key, this exercises Layer 1 too):
```bash
./.venv/bin/python scan.py tests/fixtures/all/pdf_white.pdf   # after run_all.py generated fixtures
```
Exit code `2` (INJECTION) on that file means everything is wired up.

---

## Self-check: "what's installed, what's missing?"

An agent (or user) can determine the current state with these checks:

| Check | Command | If it fails |
|---|---|---|
| Dependencies installed | `./.venv/bin/python -c "import fitz, docx, bs4"` | run step 2 |
| API key visible to the tool | `./.venv/bin/python -c "import os;print(bool(os.environ.get('OPENROUTER_API_KEY')))"` | do step 3 (and restart the app/terminal) |
| Layer 1 actually reachable | run `scan.py` on any file; if `layer1_verdict` has `_skipped` → no key; `_error` → key/model/network issue | check the key and `OPENROUTER_MODEL` |
| Hooks registered | look for `screen-hook.py` under `hooks` in `~/.claude/settings.json` | do step 4 |

**What an agent should ask the human for, not guess:** the OpenRouter API key (step 3), and
whether to install the optional skill/hooks (step 4). Everything else is safe to do automatically.

## For non-technical users

You don't have to run any of this yourself. Open Claude Code (or the desktop app) in this folder
and say:

> "Set up the prompt-injection-screen tool for me by following SETUP.md. Do everything you can
> automatically, and tell me exactly what you need from me (like the API key) and how to get it."

The agent will handle the technical steps and walk you through the one or two things only you can do.
