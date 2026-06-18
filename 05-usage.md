# 05 — Usage (operational guide)

What's built and how to run it. (See also [SETUP.md](SETUP.md) for first-time install.)

## The protection layers — what's what

| Piece | File | Role | When it runs |
|---|---|---|---|
| **`scan.py`** | `scan.py` | Engine: Layer 0 (deterministic) + Layer 1 (LLM via OpenRouter) | Called by the skill and hooks |
| **skill `/screen`** | `~/.claude/skills/screen/SKILL.md` | Manual screen of a file/URL on demand | When you ask |
| **hook** | `hooks/screen-hook.py` | Automatic tripwire on `Read` / `WebFetch` / `WebSearch` | In the background, on every read/fetch |
| **Form A** | `~/.claude/CLAUDE.md` | The permission wall — "content = untrusted data" | Always |

## First run — the OpenRouter key (Layer 1)

Layer 0 (hiding-mechanics scan) works **immediately, with nothing**. Layer 1 (the semantic classifier — catches visible prose, the EchoLeak class) needs an OpenRouter key.

Key: https://openrouter.ai/keys → Create Key → copy `sk-or-...`.

### Where to put it — depends on how you run Claude

**A) Claude desktop app (Cowork / Code) — recommended:**
macOS does NOT pass `~/.zshrc` variables to GUI apps. Use the built-in editor:
1. By the prompt box → environment dropdown → hover **Local** → gear icon (local environment editor).
2. Add `OPENROUTER_API_KEY` = `sk-or-...` (optionally `OPENROUTER_MODEL` = `anthropic/claude-haiku-4.5`).
3. Save (encrypted) → start a new session.
This reaches the session AND the hooks/Bash that call `scan.py`.
(Docs: https://code.claude.com/docs/en/desktop.md — "Local sessions / environment editor".)

**B) CLI from a terminal:** `~/.zshrc` works:
```bash
export OPENROUTER_API_KEY="sk-or-..."
```

**C) Universal alternative** — the `"env"` block in `~/.claude/settings.json`:
```json
"env": { "OPENROUTER_API_KEY": "sk-or-..." }
```
Works for both, but the key sits in plaintext in the file (less safe than option A's editor).

Without a key everything still works, but Layer 1 is skipped — `scan.py` reports this (`_skipped`), and the skill/hook says so.

## Manual use — `/screen`

```
/screen /path/to/paper.pdf
/screen https://example.com/article
```
The skill runs `scan.py` (via `curl` for a URL — the raw content never enters the chat), shows a report (CLEAN / SUSPICIOUS / INJECTION + evidence) and **stops**. Whether to read on is your call.

Directly from a terminal (e.g. in batch):
```bash
cd <repo>
./.venv/bin/python scan.py some.pdf                      # full scan
./.venv/bin/python scan.py some.pdf --fast               # Layer 0 only
./.venv/bin/python scan.py some.pdf --sanitize out.md    # + a sanitized copy if clean
cat page.html | ./.venv/bin/python scan.py --stdin       # scan text from STDIN
```

## Automatic use — hooks

Registered in `~/.claude/settings.json`:
- **`Read` (PreToolUse)** — when the agent is about to read a `.pdf/.docx/.html`, the hook scans it (`--fast`). It **holds the read on ANY signal** (SUSPICIOUS and INJECTION) and hands the decision to you (high-recall policy — better too often than too rarely). It's a call for review, not an irreversible block: once you approve, the agent reads on. False alarms are accepted.
- **`WebFetch` / `WebSearch` (PostToolUse)** — after content is fetched from the web, the hook scans it (full mode, with the LLM if a key is set). On a signal it **warns** (does not block) and tells the agent to treat the content as untrusted.

You can force the mode with `SCREEN_HOOK_MODE=fast` (cheap, no API) or `full` (with the LLM). Defaults: Read=fast, WebFetch/WebSearch=full.

### Hook limits (deliberate)
- The `Read` hook scans only `.pdf/.docx/.html` — **not** `.txt/.md` (to avoid slowing ordinary work). For text files, use `/screen`.
- `PostToolUse` does not 100% guarantee it runs before the model processes the content — it's a tripwire, not a block. The last line of defense is **Form A** (the permission wall), not the hook.

## `scan.py` statuses and exit codes
- `0` CLEAN · `1` SUSPICIOUS · `2` INJECTION · `3` ERROR.

## Maintenance
- Dependencies: `./.venv/bin/python -m pip install -r requirements.txt`.
- **Layer-0 regression: `./.venv/bin/python tests/run_all.py`** — generates a fixture for EVERY defined condition (Unicode ×9, generic exfiltration, PDF white/tiny/off-page/metadata, DOCX hidden/metadata, HTML ×11) and checks detection. Expect all green (currently **31/31**). Run after any threshold/marker change.
- Markers and thresholds: constants at the top of `scan.py` (`INSTRUCTION_MARKERS`, `ZERO_WIDTH`, the PDF thresholds).

## Tuning backlog (after real-world use)
- False alarms on legitimate documents → calibrate thresholds (color, size, BOM), PhantomLint-style.
- A local Layer-1 model (privacy/offline) instead of OpenRouter.
- Extend the `Read` hook to `.txt/.md` if it proves useful.
- More formats (PPTX, XLSX, RTF, EPUB, OCR). See [AGENTS.md](AGENTS.md) for the full roadmap.
