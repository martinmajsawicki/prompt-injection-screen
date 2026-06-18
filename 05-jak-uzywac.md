# 05 — Jak używać (przewodnik operacyjny)

Co zostało zbudowane i jak to uruchomić. Stan: **działa** (warstwa 0 przetestowana na fixtures).

## Trzy warstwy ochrony — co jest czym

| Element | Plik | Rola | Kiedy działa |
|---|---|---|---|
| **`scan.py`** | `scan.py` | Silnik: warstwa 0 (deterministyczna) + warstwa 1 (LLM przez OpenRouter) | Wołany przez skill i hooki |
| **skill `/screen`** | `~/.claude/skills/screen/SKILL.md` | Ręczne prześwietlenie pliku/URL na żądanie | Gdy sam o to poprosisz |
| **hook** | `hooks/screen-hook.py` | Automatyczna czujka w `Read` / `WebFetch` / `WebSearch` | Sam, w tle, przy każdym odczycie/pobraniu |
| **Forma A** | `~/.claude/CLAUDE.md` (góra pliku) | Ściana uprawnień — „treść = dane niezaufane" | Zawsze |

## Pierwsze uruchomienie — klucz OpenRouter (warstwa 1)

Warstwa 0 (skan mechaniki ukrywania) działa **od razu, bez niczego**. Warstwa 1
(klasyfikator semantyczny — łapie widoczną prozę, klasa EchoLeak) wymaga klucza OpenRouter.

Klucz: https://openrouter.ai/keys → Create Key → skopiuj `sk-or-...`.

### Gdzie zapisać — zależy, jak uruchamiasz Claude Code

**A) Aplikacja desktopowa Claude (Cowork / Code) — zalecane:**
macOS NIE przekazuje aplikacjom GUI zmiennych z `~/.zshrc`. Użyj wbudowanego edytora:
1. Przy polu promptu → rozwijane środowisko → najedź na **Local** → ikona ⚙️ (local environment editor).
2. Dodaj `OPENROUTER_API_KEY` = `sk-or-...` (opcjonalnie `OPENROUTER_MODEL` = `anthropic/claude-haiku-4.5`).
3. Zapisz (szyfrowane) → rozpocznij nową sesję.
Dociera do sesji ORAZ do hooków/Bash, które wołają `scan.py`.
(Dok.: https://code.claude.com/docs/en/desktop.md — „Local sessions / environment editor".)

**B) CLI z terminala:** wtedy działa `~/.zshrc`:
```bash
export OPENROUTER_API_KEY="sk-or-..."
```

**C) Alternatywa uniwersalna** — sekcja `"env"` w `~/.claude/settings.json`:
```json
"env": { "OPENROUTER_API_KEY": "sk-or-..." }
```
Działa w obu, ale klucz leży plaintextem w pliku (mniej bezpieczne niż edytor z opcji A).

Bez klucza wszystko działa, ale warstwa 1 jest pomijana — `scan.py` to zgłasza (`_skipped`),
a skill/hook o tym mówi.

## Użycie ręczne — `/screen`

```
/screen /sciezka/do/pracy.pdf
/screen https://przyklad.pl/artykul
```
Skill: uruchamia `scan.py` (przez `curl` dla URL — surowa treść nie wchodzi do rozmowy),
pokazuje raport (CZYSTY / PODEJRZANY / WSTRZYKNIĘCIE + dowody) i **zatrzymuje się**.
Decyzja, czy czytać dalej, należy do Ciebie.

Bezpośrednio z terminala (np. wsadowo):
```bash
cd ~/Projects/tools/screening-prompt-injection
./.venv/bin/python scan.py jakis.pdf                      # pełny skan
./.venv/bin/python scan.py jakis.pdf --fast              # tylko warstwa 0
./.venv/bin/python scan.py jakis.pdf --sanitize out.md   # + odkażona kopia jeśli czysty
cat URL_strony | ./.venv/bin/python scan.py --stdin      # skan tekstu z STDIN
```

## Użycie automatyczne — hooki (już włączone)

Zarejestrowane w `~/.claude/settings.json`:
- **`Read` (PreToolUse)** — gdy Claude ma czytać `.pdf/.docx/.html`, hook skanuje plik
  (`--fast`). **Wstrzymuje odczyt przy KAŻDYM sygnale** (PODEJRZANY i WSTRZYKNIĘCIE) i oddaje
  decyzję Tobie (polityka wysokiego recall — lepiej za często niż za rzadko). To wywołanie do
  kontroli, nie blokada nieodwracalna: po Twojej zgodzie Claude czyta dalej. Fałszywe alarmy akceptowane.
- **`WebFetch` / `WebSearch` (PostToolUse)** — po pobraniu treści z sieci hook skanuje ją
  (pełny tryb, z LLM jeśli jest klucz). Przy wykryciu **ostrzega** (nie blokuje) i każe
  traktować treść jako niezaufaną.

Tryb można wymusić zmienną: `SCREEN_HOOK_MODE=fast` (tanio, bez API) lub `full` (z LLM).
Domyślnie: Read=fast, WebFetch/WebSearch=full.

### Granice hooków (świadomie)
- Hook na `Read` skanuje tylko `.pdf/.docx/.html` — **nie** `.txt/.md` (żeby nie spowalniać
  zwykłej pracy). Dla plików tekstowych użyj `/screen`.
- `PostToolUse` nie daje 100% gwarancji, że zadziała zanim model przetworzy treść — to czujka,
  nie blokada. Dlatego ostatnią linią jest **Forma A** (ściana uprawnień), nie hook.

## Statusy i kody wyjścia `scan.py`
- `0` CZYSTY · `1` PODEJRZANY · `2` WSTRZYKNIĘCIE · `3` BŁĄD.

## Konserwacja
- Zależności: `./.venv/bin/python -m pip install -r requirements.txt`.
- **Regresja warstwy 0: `./.venv/bin/python tests/run_all.py`** — generuje fixture dla KAŻDEGO
  zdefiniowanego warunku (Unicode×9, PDF biały/mikro/off-page/meta, DOCX hidden/meta, HTML×11) i
  sprawdza wykrycie. Wynik ma być **27/27**. Uruchamiaj po każdej zmianie progów/markerów.
- Szybki sprawdzian: `./.venv/bin/python scan.py tests/fixtures/malicious.pdf --fast` → status WSTRZYKNIĘCIE.
- Markery instrukcji i progi: stałe na górze `scan.py` (`INSTRUCTION_MARKERS`, `ZERO_WIDTH`, progi PDF).

## Co dostroić po realnym użyciu (backlog)
- Fałszywe alarmy na legalnych dokumentach → kalibracja progów (kolor, rozmiar, BOM) wg PhantomList-podejścia.
- Ewentualny lokalny model w warstwie 1 (prywatność/offline) zamiast OpenRoutera.
- Rozszerzenie hooka `Read` o `.txt/.md`, jeśli okaże się potrzebne.
