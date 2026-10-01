# GenAI · Agentic AI · AI Agents — my course work

Right now I'm on **Module 1**: tokens, cost and context (Session 9b), and picking the right model for the job (Session 10).

## Module 1 — what's actually in here

### Session 9b — tokens, cost, context, embeddings

I spent this session figuring out what a model *actually* sees before it answers anything — not words, tokens.

- [s09_tokens.ipynb](module01/s09_tokens.ipynb) — counts tokens with `tiktoken`, checks that count against what the provider's `usage` field actually reports, converts tokens into rupees, and plots ten banking-related words in 2-D using PCA on their embeddings.
- [tokens_utils.py](module01/tokens_utils.py) — the helper functions behind the notebook: `count_tokens`, `show_tokens`, `cost_inr`, `usage_for`, `embed_via_ollama`, `pca_2d`.
- [reading_tokens.md](module01/reading_tokens.md) and [terms_s09.md](module01/terms_s09.md) — the pre-read and glossary for this session.

**The thing that actually surprised me:** `tiktoken` only counts the text you write. But the provider's real `usage` number is bigger, because it also counts the chat-template wrapper added behind the scenes. For Groq's `openai/gpt-oss-20b`, that wrapper turned out to be a fixed 71 tokens every single request — strip that out and it matches `tiktoken` exactly. And token counts genuinely don't transfer across tokenizers: the exact same Telugu sentence cost roughly 4x more tokens on an Ollama model than it did under `o200k_harmony`.

| text | tiktoken | Groq usage | Groq minus wrapper | Ollama usage |
|---|---|---|---|---|
| English | 16 | 87 | 16 | 41 |
| Telugu | 25 | 96 | 25 | 132 |
| Python | 35 | 106 | 35 | 61 |
| JSON | 25 | 96 | 25 | 49 |

### Session 10 — picking a model without guessing

This one was about not just reaching for whatever model is trendy. The method I'm using throughout the course is: **eliminate** anything that fails a hard constraint (data residency, latency, context size, call volume) → **score** whatever survives on cost, latency and quality, weighted by what the business actually cares about → **verify** the winner against real examples before trusting it.

- [s10_model_matrix.ipynb](module01/s10_model_matrix.ipynb) — runs that whole method against a model catalog across a few realistic scenarios, and actually times real models answering the same prompt.
- [landscape_utils.py](module01/landscape_utils.py) — the logic behind it: `rank`, `eligible`, `monthly_cost_usd`/`monthly_cost_inr`, `time_stream`, plus the quality-checking bits (`emi`, `check_answer`, `RUBRIC`).
- [models_catalog.json](module01/models_catalog.json) — the model catalog itself. Fair warning: the numbers in it are **indicative** (as of Sep 2026), and quality/latency start out as guesses I'm meant to replace with my own measurements.
- [reading_landscape.md](module01/reading_landscape.md) and [terms_s10.md](module01/terms_s10.md) — pre-read (with my own answers filled in) and glossary.

**The test case I used:** someone earning Rs 60,000/month, already paying Rs 8,000/month in EMIs, asking for a Rs 8,00,000 loan at 11.5% over 60 months. The correct numbers are EMI = **Rs 17,594.09**, ratio = **42.7%** — so yes, that's under the 50% cutoff. `check_answer()` counts it as correct if the EMI is within 1% and the ratio within 0.6 points; `RUBRIC` is the 1–5 scale I use to grade it by hand on top of that.

## Getting it running

You'll need [uv](https://docs.astral.sh/uv/) and Python 3.12 (it's pinned in `.python-version`). Everything runs through `uv run` so you don't have to think about activating a venv yourself.

```powershell
uv sync                           # install pinned dependencies
copy .env.example .env            # then paste your own Groq key as API_KEY
uv run python hello.py            # first LLM call, prints the token count
uv run pytest                     # all checkpoints
```

`.env` is git-ignored on purpose — never commit a real key. If one ever leaks, revoke it in the Groq console right away.

**Want to switch provider?** In `.env`, comment out the three Groq lines and uncomment the three Ollama lines — no code changes needed. For local models you'll need `ollama pull llama3.2:3b` and `ollama pull nomic-embed-text` (if Ollama isn't running, the embeddings section just falls back to a cached copy instead of failing).

### Running Module 1 specifically

```powershell
uv run jupyter lab module01/s09_tokens.ipynb
uv run jupyter lab module01/s10_model_matrix.ipynb
uv run pytest tests/test_s09.py tests/test_s10.py
```

`tests/test_s10.py` runs fully offline. `tiktoken` will download its encoding file the first time you use it. The pinned model lives in `.env.example` under `MODEL` — if it ever 404s on you, run `uv run python list_models.py` to see what's actually available.

## How this repo is laid out

```
module01/            notebooks, helpers, catalog, pre-reads and glossaries
tests/               test_setup.py, test_s09.py, test_s10.py
cheatsheet/          python-for-agents.md — the six patterns
hello.py             first LLM call
list_models.py       models your key can use
.env.example         copy to .env
pyproject.toml       pinned dependencies (uv sync)
```

More modules get added here as the course moves forward.

## Troubleshooting (things that actually went wrong for me)

- **`uv` isn't recognized** — close every open terminal, including the one inside VS Code, and reopen it.
- **401 invalid API key** — double check you're editing `.env`, not `.env.example`. No quotes around the key, no trailing space, and make sure it's not still the placeholder text.
- **Ollama connection refused** — Ollama just isn't running. Check the tray icon, or start it manually with `ollama serve`.
- **404 model not found** — the pinned model probably got retired on the provider's end. Run `list_models.py` to see current options, and flag it to the instructor.
