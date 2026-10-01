# GenAI · Agentic AI · AI Agents — course work

My working repo for the GenAI / Agentic AI / AI Agents course (instructor: Ajit Byru, [byruajit/genai-agents-course](https://github.com/byruajit/genai-agents-course)). It holds the course scaffolding plus my notebooks, notes and experiments, one `moduleNN/` folder at a time.

**Currently covered:** Module 1 — tokens, cost and context (Session 9b) and the model landscape (Session 10).

## What's in Module 1

### Session 9b — tokens, cost, context, embeddings
- [s09_tokens.ipynb](module01/s09_tokens.ipynb) — count tokens with `tiktoken`, compare against the `usage` a provider reports, turn tokens into rupees, and plot ten word embeddings in 2-D with PCA.
- [tokens_utils.py](module01/tokens_utils.py) — helpers: `count_tokens`, `show_tokens`, `cost_inr`, `usage_for`, `embed_via_ollama`, `pca_2d`.
- [reading_tokens.md](module01/reading_tokens.md) · [terms_s09.md](module01/terms_s09.md) — pre-read and glossary.

**Key finding.** `tiktoken` counts only your text. A provider's `usage` also counts the chat-template wrapper the model receives. On Groq `openai/gpt-oss-20b` that wrapper is a fixed 71 tokens per request, and subtracting it reproduces the `tiktoken` count exactly. Token counts are only comparable within one tokenizer: on an Ollama model, the same Telugu text cost about 4× more tokens than in `o200k_harmony`.

| text | tiktoken | Groq usage | Groq minus wrapper | Ollama usage |
|---|---|---|---|---|
| English | 16 | 87 | 16 | 41 |
| Telugu | 25 | 96 | 25 | 132 |
| Python | 35 | 106 | 35 | 61 |
| JSON | 25 | 96 | 25 | 49 |

### Session 10 — the model landscape
Method: **eliminate** with hard constraints (data residency, latency, context, volume) → **score** survivors on cost, latency and quality with business-chosen weights → **verify** on your own examples.

- [s10_model_matrix.ipynb](module01/s10_model_matrix.ipynb) — runs the method over the catalog for several scenarios and times real models on the same prompt.
- [landscape_utils.py](module01/landscape_utils.py) — `rank`, `eligible`, `monthly_cost_usd/inr`, `time_stream`, and the quality checker (`emi`, `check_answer`, `RUBRIC`).
- [models_catalog.json](module01/models_catalog.json) — model catalog. Numbers are **indicative** (Sep 2026); quality and latency are starting guesses to be replaced with measurements.
- [reading_landscape.md](module01/reading_landscape.md) · [terms_s10.md](module01/terms_s10.md) — pre-read (with my answers) and glossary.

**The test prompt.** Rs 60,000/month income, Rs 8,000 existing EMIs, Rs 8,00,000 loan at 11.5% for 60 months. Correct answer: EMI **Rs 17,594.09**, ratio **42.7%**, so **yes**, under the 50% limit. `check_answer` accepts an EMI within 1% and a ratio within 0.6 points; `RUBRIC` is the 1–5 manual score.

## Setup

Requires [uv](https://docs.astral.sh/uv/) and Python 3.12 (pinned in `.python-version`). Every Python command goes through `uv run`.

```powershell
uv sync                           # install pinned dependencies
copy .env.example .env            # then paste your own Groq key as API_KEY
uv run python hello.py            # first LLM call, prints the token count
uv run pytest                     # all checkpoints
```

`.env` is git-ignored. Never commit a key; if one leaks, revoke it in the Groq console.

**Switching provider.** In `.env`, comment the three Groq lines and uncomment the three Ollama lines. No code changes. For local models: `ollama pull llama3.2:3b` and `ollama pull nomic-embed-text` (the embeddings section falls back to a cached copy if Ollama is absent).

### Running Module 1

```powershell
uv run jupyter lab module01/s09_tokens.ipynb
uv run jupyter lab module01/s10_model_matrix.ipynb
uv run pytest tests/test_s09.py tests/test_s10.py
```

`tests/test_s10.py` works offline. `tiktoken` downloads its encoding file once on first use. Pinned model `MODEL` is set in `.env.example`; if it returns 404, list current models with `uv run python list_models.py`.

## Layout

```
module01/            notebooks, helpers, catalog, pre-reads and glossaries
tests/               test_setup.py, test_s09.py, test_s10.py
cheatsheet/          python-for-agents.md — the six patterns
hello.py             first LLM call
list_models.py       models your key can use
.env.example         copy to .env
pyproject.toml       pinned dependencies (uv sync)
```

Further modules are added as the course progresses.

## Troubleshooting

- **`uv` is not recognized** — close every terminal (including VS Code's) and reopen.
- **401 invalid API key** — check `.env`, not `.env.example`: no quotes, no trailing space, not the placeholder.
- **Ollama connection refused** — Ollama isn't running; check the tray icon or run `ollama serve`.
- **404 model not found** — the pinned model was retired; run `list_models.py` and tell the instructor.
