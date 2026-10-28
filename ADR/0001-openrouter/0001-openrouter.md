# ADR-0001: OpenRouter as the gateway to third-party LLMs

- Status: Validated
- Date: 2026-10-28
- Decision maker: Mehdi B.
- Confidence: Medium (to be revisited when the LiteLLM gateway is set up)

***

## Context
- The core infrastructure I host at home is evolving fast: OpenClaw (orchestrator + agents) needs access to several models (Anthropic, Kimi, GLM, DeepSeek…) without managing one account and one key per provider.
- Models change fast: I need to be able to switch models without changing code.
- My needs and projects evolve as fast as the technology.
- Speed of execution is critical while both my needs and the various tech stacks stabilize.
- However, I have a real concern about how these "unknown" AI providers consume and reuse my own data.
- No budget cap for now: no tasks running in loops, only actions I trigger myself.

***

## Decision
Use **OpenRouter as the single access point** (OpenAI-compatible API) for all remote models I integrate into my apps.
- **Provider routing set globally** (`order` + `allow_fallbacks`) at the `agents.defaults.models` level.
- **Zero Data Retention (ZDR)** enabled at account level for all models:
    - Anthropic / OpenAI / SpaceXAI: ZDR (direct endpoints disabled → Bedrock / Vertex / Azure)
    - Google: safety retention (enforced since 2026-10-18), direct endpoint disabled → Vertex only; no sensitive data sent to Gemini
    - All other models: zero retention only
- **Health and other sensitive data flows** never go through OpenRouter, only through local Ollama. Enforcement will be handled by LiteLLM (upcoming ADR). Until then, no OpenRouter key is configured in any service that processes health data.
- Create **one API key per project** to track each project's usage.

***

## Rejected alternatives
1. Direct provider APIs: no intermediary fees, but N accounts, N keys, N invoices.
2. LiteLLM alone (self-hosted): no platform fees, but still requires an account with each provider. The two remain compatible (LiteLLM in front, OpenRouter behind).
3. Local models only (Ollama): insufficient quality and context size for orchestration.

***

## Consequences

**Positive**
- A single key; switching models on the fly is just a config change.
- A single invoice: administrative simplicity.
- Automatic failover to another provider if one goes down.

**Negative**
- ~5.5% fee on credit purchases.
- One more intermediary to depend on (May 2026 incident: 500 errors with Kimi via NovitaAI).
- Quality varies depending on the provider behind the same model, so routing must be actively managed.
- Enabling ZDR:
    - Disables the direct endpoints of Anthropic, Google, OpenAI and SpaceXAI.
    - Removes some of the cheapest providers for the other models (beyond those listed above).
    - May make precise provider targeting harder (e.g. SiliconFlow / Baseten).
    - Mechanically shrinks the provider pool, which can reduce my applications' resilience.
    - Prompts sent to Gemini may be retained (mandatory safety retention).
    - ZDR does not guarantee that data is processed in Europe.

***

## When to reassess
- Spending exceeds €100/month: reassess not OpenRouter itself, but moving some flows to a local LLM.
- Provider incidents recur and actually impact my applications.
- The LiteLLM gateway is deployed (knowledge base project).
- RAM / GPU prices make local hardware pay for itself in under a year compared to API calls. This depends on several factors: my API call volume, increases in per-million-token pricing, and hardware cost.

***

## Open questions / next steps
- Study LiteLLM for securing and routing my AI flows.
- Study local LLMs to delegate very specific tasks to right-sized models, cut costs and, above all, keep control of my data and secure it.
