# Pre-read for Session 10 — ten minutes

**The question.** Yesterday you learned what a model sees. Today: *which* model goes in the box? There are hundreds. The skill is not knowing them all — it is having a method that works when a new one appears next month.

## Three ways to get a model
1. **Vendor API (closed, "frontier").** A vendor (OpenAI, Anthropic, Google…) runs its own model; you send text over the internet and pay per token. You cannot download the model.
2. **Hosted open-weight.** The model's weights are published; a provider (Groq, and many others) runs them for you. You still send your text to someone else's machine, but the price is usually much lower.
3. **Self-hosted / local.** You run the same open weights yourself — on your laptop with Ollama, or on a GPU server in your own data centre. The text never leaves; you pay for hardware and for the people who run it, not per token.

**Open-weight** means the trained numbers (the *weights*) can be downloaded. It does not automatically mean *open source*: licences differ (some restrict commercial use), and training data and code are often not released.

## What differs between models
- **Tier.** Most vendors sell a family: a flagship, a mid-tier, an economy model. Same brand, very different price and speed. Rule: start with the smallest tier and move up only when *your own test* says you must.
- **Size.** Measured in *parameters* (the learned numbers): 3B, 8B, 20B, 120B. Bigger usually means more capable, slower, and hungrier for memory. Some models are *mixture-of-experts*: many parameters in total but only a fraction active for each token, which makes them fast for their size (the gpt-oss models are like this).
- **Context length.** How many tokens fit in the window. Hundreds of thousands to a million now — but you pay for every input token on every call, and a full window is not a free window. Output has its own, smaller limit.
- **Price.** Quoted per *million* tokens, separately for input and output. Output usually costs 3–6× input. Reasoning models bill their hidden thinking as output.
- **Speed.** Three different numbers: time to first token, tokens per second, total time. A reasoning model can be slow to the first *visible* word because it thinks first.
- **Quality.** The one you cannot read off a page. Leaderboards help you shortlist; only your own test on your own task decides.

## The constraint nobody can negotiate: data residency
Some data may not leave your building, or your country, however good and cheap the model is — customer identity documents, payment data, health records. Rules and contracts decide, not engineers. So residency is a **hard constraint**: it *removes* options before you compare anything else. Regulations differ by country and sector; in class we treat it as "ask compliance", and Session 57 returns to it.

## The method we will use (and reuse all course)
1. **Eliminate** with hard constraints (data must stay in-house; must answer in 3 seconds; must fit the context; must handle the volume).
2. **Score** the survivors on cost, speed and quality with weights the *business* chooses.
3. **Verify** the winner on your own examples. (Session 17 gives you the tools; today you do a tiny version.)

## Three questions to arrive with
- A bank wants a chatbot for public product questions *and* a system that reads customers' salary slips. Should they use the same model? Why or why not?

  -> No. The two jobs have completely different risk profiles and requirements. The FAQ bot answers general, public questions — no customer data, mistakes cost little beyond a mildly unhelpful reply, and it needs to be cheap at high volume. The salary-slip reader processes real customer PII (income, deductions, employer) that often has a hard data-residency constraint, and a mistake there (misreading a number) can feed directly into a loan decision, which is a much higher-stakes failure mode. Applying the course's method: the two jobs would get eliminated down to different survivor sets (the hosted FAQ model is excluded from the document job by the data-residency hard constraint) and scored with different weights (cost-heavy for FAQ, quality-heavy for documents) — so they land on different models almost by construction, not by preference.

- Your laptop can run a small model for free. Why can't it be the FAQ bot for 20,000 calls a day?

  -> Three things break down at that volume: **concurrency** (your laptop serves one conversation at a time; 20,000 calls/day means many requests can arrive at once, and a laptop has no way to serve dozens of simultaneous users without queuing or crashing), **uptime** (a laptop sleeps, loses wifi, reboots, and isn't built to stay reachable 24/7 the way a production service needs to be), and **capacity vs. free** ("free" just means no per-token bill — you're still spending finite CPU/GPU, RAM, and electricity, and a laptop's throughput tops out far below what 20,000 calls/day requires, as the course's own notebook measured: ~5,000 calls/day before this exact kind of local setup can't keep up).

- A leaderboard says Model A beats Model B by 4 points. What would you still want to check before switching?

  -> First, whether the benchmark actually measures the task I care about — a 4-point gain on a coding or trivia benchmark says nothing about quality on, say, loan-document extraction. Second, whether 4 points is a real difference or just noise — benchmark scores can vary run-to-run or depend on a small test set, so I'd want to know the variance, not just the headline number. Third, what the leaderboard doesn't show at all: cost, latency, context window, rate limits, and — most importantly — how the model actually performs on *my own* prompts and data, since that's the only test the course's method treats as final ("verify the winner on your own examples").

