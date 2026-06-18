# Sources

Research done 2026-06-18 (web search + fetch).

## Standards and official documentation

- **OWASP — LLM01:2025 Prompt Injection** — https://genai.owasp.org/llmrisk/llm01-prompt-injection/
- **OWASP Top 10 for LLM Applications 2025 (PDF)** — https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-v2025.pdf
- **Anthropic — Mitigate jailbreaks and prompt injections** (key: harmlessness screen, screening tool results, untrusted_content_policy, JSON-encode, structured output) — https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks
- **Anthropic — Mitigating the risk of prompt injections in browser use** — https://www.anthropic.com/research/prompt-injection-defenses
- **Google DeepMind — Lessons from Defending Gemini Against Indirect Prompt Injections (2025, PDF)** — https://storage.googleapis.com/deepmind-media/Security%20and%20Privacy/Gemini_Security_Paper.pdf

## Academic papers

- **PhantomLint: Principled Detection of Hidden LLM Prompts in Structured Documents** (2508.17884) — https://arxiv.org/pdf/2508.17884 — *closest to a "document screener": hiding techniques + multi-layer detection, low FP.*
- **Mitigating Indirect Prompt Injection via Instruction-Following Intent Analysis (IntentGuard)** (2512.00966) — https://arxiv.org/abs/2512.00966
- **MELON: Provable Defense Against Indirect Prompt Injection Attacks in AI Agents** (2502.05174) — https://arxiv.org/pdf/2502.05174
- **Defense Against Indirect Prompt Injection via Tool Result Parsing** (2601.04795) — https://arxiv.org/pdf/2601.04795
- **Are AI-assisted Development Tools Immune to Prompt Injection?** (2603.21642) — https://arxiv.org/pdf/2603.21642

## Industry analyses / cases

- **Lakera — Indirect Prompt Injection: The Hidden Threat** (incl. the Perplexity Comet / Reddit / OTP-leak case) — https://www.lakera.ai/blog/indirect-prompt-injection
- **Prompt Injection & AI Agent Security: Claude Code Guide for Enterprise (TrueFoundry)** — https://www.truefoundry.com/blog/claude-code-prompt-injection
- **MintMCP — Claude Cowork File Exfiltration / Prompt Injection** — https://www.mintmcp.com/blog/claude-cowork-file-exfiltration · https://www.mintmcp.com/blog/claude-cowork-promt-injection
- **Wikipedia — Prompt injection** — https://en.wikipedia.org/wiki/Prompt_injection

## Context

- **Johann Rehberger (embracethered.com)** — researcher who publicly demonstrated hidden instructions in documents and exfiltration vectors (referenced in the sources above).

---

*Note: some arXiv IDs (26xx) are from the research time window. When citing, verify the final status (preprint vs. peer-reviewed).*
