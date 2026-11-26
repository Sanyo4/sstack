# Models: why each sits where it does

The `routing` table says which model runs each role. This file holds the reasons, the prices and the quirks. When a model ships, edit both together.

## Claude

- Opus 5.5 at medium matches or beats Opus 5 at high on coding, in fewer steps. It is strongest on multistep work in a real repo, catches more bugs in review with fewer false alarms, states wrong figures less often, and reads charts and screenshots better than any earlier Opus. It also holds long unattended runs with parallel subagents.
- Sonnet 5.5 is the speed and cost pick. Anthropic says an Opus model is the better choice for the hardest long-horizon work. Sonnet needs steering (see the worker role) at low and medium effort.
- Haiku 5.5 (released 2026-10-07) is the cheapest and fastest, about 20x cheaper than Sonnet on prompts up to 100K tokens. It is the first Haiku with effort (default medium), and has a 1M context, 128K output and a June 2026 knowledge cutoff. It is far ahead of Haiku 4.5 and close to Sonnet on computer and browser use (OSWorld 72.4% against Sonnet 83.9% and Haiku 4.5 15.7%) and on knowledge work (Humanity's Last Exam with tools 57.4% against Sonnet 64.5%). It is weak at agentic coding (Terminal-Bench 39.2% against Sonnet 70.6%), and Anthropic says Sonnet and Opus stay the better choice there. So Haiku reads, runs, classifies and reports; it does not write code. Its safety classifiers can refuse, including on benign security code, and there is no fallback model: send a refused task to a Sonnet role.
- Fable 5.2 (released 2026-11-09) is the top tier: 2x the Opus price, up to 30% faster than Fable 5.1, 1M context, 128K output. Anthropic reports it roughly matches Opus 5.5 on coding at high effort for about half the cost per task, and beats it at xhigh (FrontierCode 2.0: 49.1% against 45.5%). Anthropic still says to start most coding with Opus 5.5 and move up to Fable for the most open-ended technical work; Fable 5.2 also works well as a planner or orchestrator over cheaper models. It carries extra safeguards for biology, cybersecurity and LLM R&D (refusal categories `cyber`, `bio`, `frontier_llm`, `reasoning_extraction`, `general_harms`). If it refuses, drop back to Opus at high.

### Fable 5.2 compared with 5.1

- It stops early more often at low and medium effort. The architect section of `roles.md` carries a "Finishing work" paragraph that separates ordinary obstacles (work through them) from deliberate blockers (leave them, report them).
- It follows notes and "never" rules closely, so cautions that pile up in memory, briefs and reviewer findings can make it hold back routine work or ask for approvals it does not need. Name the exact actions that need the user's yes, and say which routine steps (retries, re-reads, test runs) need none. Prune stale cautions from memory and briefs.
- Lines that ask it to think less do not work at medium and above. Lower effort instead.
- It responds to what the prompt says about who owns the code. Saying "this codebase is yours" makes it merge repeated code and fix nearby problems, at about 1.5x the output and with changes nobody asked for. Leave the sentence out of briefs by default; add it only to a builder brief whose job is a restructure.
- At xhigh and max it may start subagents to review its own work. The architect role says to start one only when the brief asks.
- It follows style rules closely; the reply paragraph in the architect role already follows a plain technical style, so no extra length limits are needed.

### Prices

Per million tokens (input / output): Fable 5.2 $8 / $40 (Fable 5.1 was $10 / $50), Opus $4 / $20, Sonnet $2 / $10, Haiku 5.5 $0.10 / $0.50 on prompts up to 100K tokens and $0.50 / $2.50 above that. Keep Haiku briefs and the files they read small enough to stay under 100K where it is easy.

### Effort data

Source: `slopalytics.com` (Artificial Analysis), read 2026-11-25. Tokens per task are output tokens; cost per task is USD.

| Model | Effort | Intelligence index | Code index | Tokens per task | Cost per task |
|---|---|---:|---:|---:|---:|
| Fable 5.2 | low | 53.9 | 44.9 | 35.9K | $3.10 |
| Fable 5.2 | medium | 56.4 | 46.8 | 43.1K | $3.80 |
| Fable 5.2 | high | 61.8 | 53.3 | 97.8K | $9.08 |
| Fable 5.2 | xhigh | 62.2 | 53.2 | 161.9K | $14.83 |
| Fable 5.2 | max | 64.6 | 53.1 | 273.8K | $26.54 |
| Opus 5.5 | low | 34.8 | 22.0 | 19.1K | $0.98 |
| Opus 5.5 | medium | 48.2 | 40.9 | 43.8K | $2.45 |
| Opus 5.5 | high | 51.8 | 44.2 | 60.1K | $3.46 |
| Opus 5.5 | xhigh | 57.4 | 50.1 | 103.4K | $6.05 |
| Opus 5.5 | max | 59.1 | 51.7 | 173.3K | $10.04 |
| Sonnet 5.5 | low | 25.6 | 12.6 | 20.9K | $0.51 |
| Sonnet 5.5 | medium | 30.8 | 18.5 | 30.7K | $0.78 |
| Sonnet 5.5 | high | 41.5 | 33.0 | 58.4K | $1.64 |
| Sonnet 5.5 | xhigh | 49.9 | 44.2 | 122.2K | $4.09 |
| Sonnet 5.5 | max | 54.5 | 50.5 | 285.4K | $10.02 |
| Haiku 5.5 | low | 18.6 | 9.9 | 28.2K | $0.04 |
| Haiku 5.5 | medium | 24.6 | 15.7 | 53.3K | $0.09 |
| Haiku 5.5 | high | 29.9 | 22.6 | 88.1K | $0.16 |
| Haiku 5.5 | xhigh | 34.6 | 29.7 | 136.2K | $0.25 |
| Haiku 5.5 | max | 36.5 | 34.1 | 226.0K | $0.41 |

Compared with Gemini 4 Neon high (90.2K tokens per task), Opus 5.5 high uses fewer tokens per task (60.1K); Neon high uses about 1.5 times the tokens of Opus 5.5 high per task, which matters on the smaller Antigravity plan.

### Effort notes

1. Opus roles run at high: the owner set this on 2026-11-10. The lead runs at high too: the owner set this on 2026-11-23. On 2026-11-25 the owner raised worker to Sonnet 5.5 high, where the coding score jumps from 18.5 to 33.0. Haiku roles stay at medium. Do not drop Haiku to low to save money: its guide says low makes it skip searches and checks and stop early in long prompts, and medium already costs very little.
2. Fable 5.2 defaults to high on the API, and Anthropic's guide puts most of its gain at high for hard coding, research and long unattended runs (code index peaks at 53.3 at high). On 2026-11-09 the owner set the architect to high.

### Checking a new model

A model alias (`opus`, `sonnet`) follows the newest model. Pin a full ID (for example `claude-opus-5-5`) only to hold a version on purpose. Check which model a subagent really ran from its transcript's `message.model`, not from the agent's own answer. On 2026-11-09 the `fable` alias still ran Fable 5.1 on the day 5.2 shipped, and an old Claude Code ran a subagent pinned to `claude-fable-5-2` as Fable 5.1 because it did not know the ID.

## Gemini

- Gemini 4 Neon is the one Gemini model T3 offers for Antigravity (`gemini-4-neon`). Its thinking level is `thought_level`: `low`, `medium`, `high` or `xhigh`. There is no `max`, so `xhigh` is the top of the ladder.
- In the routing table Neon runs at high. With Antigravity as the only vendor, every role runs on Neon: scout and runner at medium, the rest at high.

### Effort data

Source: `slopalytics.com` (Artificial Analysis), read 2026-11-25. Tokens per task are output tokens; cost per task is USD (at $3 / $15 per million input / output tokens).

| Model | Effort | Intelligence index | Code index | Tokens per task | Cost per task |
|---|---|---:|---:|---:|---:|
| Gemini 4 Neon | low | 44.1 | 29.3 | 21.7K | $0.75 |
| Gemini 4 Neon | medium | 54.9 | 44.0 | 50.0K | $1.90 |
| Gemini 4 Neon | high | 59.6 | 50.6 | 90.2K | $3.74 |
| Gemini 4 Neon | xhigh | 61.4 | 52.1 | 141.6K | $6.44 |

Neon high uses about 1.5 times the tokens of Opus 5.5 high per task (90.2K against 60.1K), which matters on the smaller Antigravity plan. High gives most of xhigh's intelligence and code score at roughly 64% of the tokens.

### Where a level is real

- The native subagent call has no level input. So a role gets its own level only through T3's `delegate_task`, with `target.options.thought_level`. In plain Antigravity every role runs at the session's level, and the report says so.

### What is sent

Antigravity's terms let Google keep, train on and read what is sent. Keep `antigravity` in the Vendors setting only when your code may go there. A repo whose owner does not allow it says so in its instruction file, and the dispatch rule in the `routing` skill keeps its code on Claude.
