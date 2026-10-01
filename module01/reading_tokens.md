# Pre-read for Session 9b — ten minutes

**What the model actually sees.** A language model never sees letters or words. It sees *tokens*: chunks of text, roughly three-quarters of an English word each, chosen by a fixed dictionary called a *tokenizer*. "EMI calculation" might be three tokens; a Telugu sentence of the same meaning might be ten, because the dictionary was built mostly from English and code.

**Why you care.** Three reasons, and they run through the whole course:
1. **Cost** — every provider charges per token, in and out. Your `hello.py` printed `prompt=96 completion=105 total=201`. That is the bill.
2. **The context window** — a model can only see a fixed number of tokens at once (tens of thousands to a million, depending on the model). Everything — your instructions, the conversation so far, retrieved documents, tool results — must fit. When it doesn't, something gets cut.
3. **Speed** — the model produces one token at a time. More tokens out, more waiting.

**Tokenizers differ by model.** The tokenizer belongs to the model, not to the text. Our default model (`openai/gpt-oss-20b`) uses an encoding called `o200k_harmony`; Llama models use a different one. So the same sentence can be 40 tokens for one model and 46 for another. In class we count with `tiktoken` (exact for gpt-oss) and then check against what each model reports in its `usage` field.

**Embeddings — the other thing a model can produce.** An *embedding model* does not answer; it turns text into a list of numbers (a *vector*) such that texts with similar meaning end up close together. Two words are "near" if their vectors point the same way. That is the machinery behind semantic search and RAG (Module 5). In class we embed ten banking words with a local model and draw them on a plane.

**Three questions to arrive with**
- If Telugu costs three times as many tokens as English, what does that mean for a bank whose customers write in Telugu?

  -> Every message from a Telugu-speaking customer costs roughly 3× more to process and eats 3× more of the context window than the same request in English — purely because of the tokenizer's vocabulary bias toward English and code, not because the request is any more complex. That's a real cost and capacity penalty tied to language, which matters for a bank whose customers don't all write in English.

- Your chatbot has a 30-turn conversation. What is in the context window at turn 30?

  -> Whatever still fits, in chronological order: the system prompt (normally kept throughout, since the model needs it to behave correctly) plus as many of the most recent turns as the token budget allows. If the full 30-turn history doesn't fit, the oldest turns are the ones dropped to make room — and that eviction is a decision the application's memory strategy makes (sliding window, summarization, retrieval), not something the model does on its own.

- What can an embedding model *not* do?

  -> It can't answer a question or generate text — it only outputs a vector, so there's nothing to read on its own. It can't explain *why* two things are similar, only that they're statistically close in its training data. And it can struggle with narrow domain jargon or acronyms that are rare in its training corpus — e.g. a bank's own term like "EMI" might not land sensibly near "loan" in embedding space even though the two are obviously related, because the model has only seen patterns, not real understanding of the concept.
