# Źródła

Research wykonany 2026-06-18 (WebSearch + WebFetch).

## Standardy i dokumentacja oficjalna

- **OWASP — LLM01:2025 Prompt Injection** — https://genai.owasp.org/llmrisk/llm01-prompt-injection/
- **OWASP Top 10 for LLM Applications 2025 (PDF)** — https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-v2025.pdf
- **Anthropic — Mitigate jailbreaks and prompt injections** (kluczowe: harmlessness screen, screening wyników narzędzi, untrusted_content_policy, JSON-encode, structured output) — https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks
- **Anthropic — Mitigating the risk of prompt injections in browser use** — https://www.anthropic.com/research/prompt-injection-defenses
- **Google DeepMind — Lessons from Defending Gemini Against Indirect Prompt Injections (2025, PDF)** — https://storage.googleapis.com/deepmind-media/Security%20and%20Privacy/Gemini_Security_Paper.pdf

## Papery naukowe

- **PhantomLint: Principled Detection of Hidden LLM Prompts in Structured Documents** (2508.17884) — https://arxiv.org/pdf/2508.17884 — *najbliższe „prześwietlaczowi dokumentów": techniki ukrywania + wielowarstwowa detekcja, niski FP.*
- **Mitigating Indirect Prompt Injection via Instruction-Following Intent Analysis (IntentGuard)** (2512.00966) — https://arxiv.org/abs/2512.00966
- **MELON: Provable Defense Against Indirect Prompt Injection Attacks in AI Agents** (2502.05174) — https://arxiv.org/pdf/2502.05174
- **Defense Against Indirect Prompt Injection via Tool Result Parsing** (2601.04795) — https://arxiv.org/pdf/2601.04795
- **Are AI-assisted Development Tools Immune to Prompt Injection?** (2603.21642) — https://arxiv.org/pdf/2603.21642

## Analizy branżowe / przypadki

- **Lakera — Indirect Prompt Injection: The Hidden Threat** (m.in. przypadek Perplexity Comet / Reddit / wyciek OTP) — https://www.lakera.ai/blog/indirect-prompt-injection
- **Prompt Injection & AI Agent Security: Claude Code Guide for Enterprise (TrueFoundry)** — https://www.truefoundry.com/blog/claude-code-prompt-injection
- **MintMCP — Claude Cowork File Exfiltration / Prompt Injection** — https://www.mintmcp.com/blog/claude-cowork-file-exfiltration · https://www.mintmcp.com/blog/claude-cowork-promt-injection
- **Wikipedia — Prompt injection** — https://en.wikipedia.org/wiki/Prompt_injection

## Postać kontekstowa

- **Johann Rehberger (embracethered.com)** — badacz, który publicznie demonstrował ukryte instrukcje w dokumentach i wektory eksfiltracji (wzmiankowany w źródłach powyżej).

---

*Uwaga: część dat arXiv (26xx) to identyfikatory z bieżącego okna czasowego researchu. Przy cytowaniu zweryfikuj ostateczny status (preprint vs. recenzowane).*
