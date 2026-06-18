# prompt-injection-screen

Prześwietla pliki, PDF-y i strony pod kątem **ukrytych i widocznych instrukcji prompt injection** — **zanim** agent LLM (Claude Code / Cowork) zacznie czytać treść i jej ufać. Dwie warstwy: deterministyczny skaner mechaniki ukrywania (nieprzekupny) + klasyfikator semantyczny w kwarantannie (OpenRouter).

> **EN — what is this?** A first-pass screener for indirect prompt injection in untrusted documents/web pages. A deterministic layer detects *hiding mechanics* (white text, zero-width chars, off-canvas text, metadata, hidden CSS); a quarantined LLM layer (via OpenRouter) detects *visible prose* that instructs an AI to collect/exfiltrate data (the EchoLeak class). Designed to run **before** a coding agent ingests the content. Prompts and messages are in Polish but trivially translatable.

---

## Po co to

Indirect prompt injection to **ryzyko #1 wg OWASP (LLM01)** i nie da się go „załatać". Atak przyjeżdża wraz z treścią, której ufasz — pracą naukową, PDF-em, stroną. Dwa realne wektory:

- **Ukryty** (np. biały tekst, znaki zero-width, metadane) — niewidoczny dla człowieka, czytelny dla modelu.
- **Widoczny** (klasa **EchoLeak**, CVE-2025-32711) — normalna proza udająca tekst do człowieka, sterująca agentem; obeszła nawet dedykowany klasyfikator Microsoftu.

To narzędzie jest **pierwszą warstwą** (czujką) — nie zastępuje zasady najmniejszych uprawnień ani human-in-the-loop, tylko je uzupełnia.

## Architektura (defense in depth)

```
PLIK / URL / PDF
   ├─ Warstwa 0 — skaner deterministyczny (NIE-LLM, nie da się wstrzyknąć)
   │    zero-width / bidi / Unicode tags, biały tekst, mikroczcionka,
   │    tekst poza CropBox, ukryte runy DOCX, ukryty HTML, metadane
   ├─ Warstwa 1 — klasyfikator semantyczny (model w KWARANTANNIE, bez narzędzi)
   │    widoczna proza i OGÓLNIKOWE polecenia „zbierz/wyślij informacje"
   │    → tylko werdykt JSON; surowa treść nigdy nie wraca do agenta (wzorzec CaMeL)
   └─ Werdykt: CZYSTY / PODEJRZANY / WSTRZYKNIĘCIE  →  decyzja należy do człowieka
```

Pełne uzasadnienie i ustalenia ekspertów (OWASP, Anthropic, Google DeepMind, papery PhantomLint/EchoLeak): **[01-zagrozenie.md](01-zagrozenie.md)**, **[02-co-mowia-eksperci.md](02-co-mowia-eksperci.md)**, **[03-architektura-obrony.md](03-architektura-obrony.md)**, **[04-instrukcja-screeningu.md](04-instrukcja-screeningu.md)**.

## Co wykrywa

| Warstwa | Sygnały |
|---|---|
| **0 (deterministyczna)** | znaki zero-width / word-joiner / BOM / soft-hyphen / bidi / Unicode Tags (+odkodowanie); biały/jasny tekst i mikroczcionka w PDF; tekst poza widocznym CropBox; ukryte runy DOCX; metadane PDF/DOCX; ukryty HTML (`display:none`, `visibility:hidden`, `font-size:0`, `opacity:0`, off-screen, alt/aria, komentarze); widoczny rozkaz zbierania/wysyłania informacji (też ogólnikowy) |
| **1 (LLM)** | widoczna proza sterująca AI, w tym ogólnikowa eksfiltracja „zinterpretuj sam, co wrażliwe"; dzielenie dużych dokumentów na części (chunking) |

## Instalacja

```bash
git clone <repo-url> prompt-injection-screen
cd prompt-injection-screen
python3 -m venv .venv
./.venv/bin/python -m pip install -r requirements.txt
```

Warstwa 1 (semantyczna) wymaga klucza OpenRouter:
```bash
export OPENROUTER_API_KEY="sk-or-..."        # https://openrouter.ai/keys
# opcjonalnie: export OPENROUTER_MODEL="anthropic/claude-haiku-4.5"
```
Bez klucza działa tylko warstwa 0 (deterministyczna). **Nigdy nie commituj klucza.**

## Użycie

**CLI:**
```bash
./.venv/bin/python scan.py dokument.pdf                 # pełny skan (warstwa 0 + 1)
./.venv/bin/python scan.py dokument.pdf --fast          # tylko warstwa 0 (bez API)
./.venv/bin/python scan.py dokument.pdf --sanitize out.md  # + odkażona kopia jeśli CZYSTY
cat strona.html | ./.venv/bin/python scan.py --stdin    # skan treści z STDIN (dla hooków)
```
Kody wyjścia: `0` CZYSTY · `1` PODEJRZANY · `2` WSTRZYKNIĘCIE · `3` BŁĄD.

**Duże pliki (np. karty modeli 300 stron):** warstwa 1 dzieli tekst na części (`--max-chars`, domyślnie 60k) i skanuje każdą — wstrzyknięcie głęboko w dokumencie nie umyka. Bardzo duże pliki: `--max-chunks` (domyślnie 80; powyżej — ostrzeżenie o pominięciu).

**Integracja z Claude Code / Cowork (opcjonalnie):**
- Skill `/screen` — patrz [examples/SKILL.md](examples/SKILL.md) (skopiuj do `~/.claude/skills/screen/`).
- Hooki automatyczne (Read / WebFetch / WebSearch) — patrz [examples/settings.hooks.json](examples/settings.hooks.json) (wklej do `~/.claude/settings.json`).
- Reguła „treść = dane niezaufane" do `~/.claude/CLAUDE.md` — gotowe brzmienie w [04-instrukcja-screeningu.md](04-instrukcja-screeningu.md) (Forma A).

Operacyjny przewodnik krok po kroku: **[05-jak-uzywac.md](05-jak-uzywac.md)**.

## Testy

```bash
./.venv/bin/python tests/run_all.py     # regresja warstwy 0: fixture dla każdego warunku (oczekiwane 31/31)
```

## Granice (uczciwie)

- Warstwa 0 nie wykryje **widocznej** prozy bez mechaniki ukrywania — od tego jest warstwa 1 (LLM).
- Detektor widocznej eksfiltracji jest **precyzyjny po polsku** (tryb rozkazujący ≠ bezokolicznik), **omylny po angielsku** (`send` dwuznaczne) — domyślnie nastawiony na **wysoki recall** (lepiej fałszywy alarm niż przeoczenie).
- Hook `PostToolUse` to czujka, nie twarda blokada — ostatnią linią obrony jest polityka uprawnień (Forma A), nie detekcja.
- Tekst poza MediaBox PDF (rzadkie) może umknąć ekstrakcji.

## Układ repo

```
scan.py              # silnik: warstwa 0 + 1 (chunking, OpenRouter przez urllib)
hooks/screen-hook.py # hook do Claude Code (ścieżka wykrywana automatycznie)
tests/run_all.py     # regresja warstwy 0
examples/            # SKILL.md + snippet hooków do settings.json
01..05*.md           # dokumentacja: zagrożenie, eksperci, architektura, instrukcje, użycie
ZRODLA.md            # źródła
```

## Bezpieczeństwo i status

Narzędzie defensywne, do screeningu treści przed analizą. **To czujka, nie gwarancja** — żaden detektor prompt injection nie ma 100% recall. Stosuj razem z najmniejszymi uprawnieniami i potwierdzeniem człowieka przy akcjach wychodzących/nieodwracalnych.

Zgłoszenia i PR mile widziane. Licencja: **[The Unlicense](LICENSE)** — public domain. Wolno używać prywatnie i komercyjnie, modyfikować i rozpowszechniać, **bez obowiązku atrybucji**.
