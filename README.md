# FunSearch from Scratch

Building [FunSearch](https://www.nature.com/articles/s41586-023-06924-6) (DeepMind, 2023) from zero — an LLM-guided evolutionary search over programs.

## Setup

- **Model provider:** NVIDIA NIM (free tier)
- **Model:** `openai/gpt-oss-20b`
- **LLM library:** [litellm](https://github.com/BerriAI/litellm)

```bash
pip install litellm
```

Set your API key in `.env`:
```
NVIDIA_NIM_API_KEY=your-key-here
```

## Notebooks

### `1.ipynb` — Foundations
Builds FunSearch piece by piece:
- **Sampler** (`sample_code`) — calls LLM, returns clean code
- **Evaluator** (`evaluate`) — runs candidate code against test assertions, returns pass/fail scores
- **Programs Database** (`DB`, `db_add`, `best`) — sorted list of (code, score) pairs
- **Prompt Builder** (`build_prompt`) — stitches task spec + best programs into an LLM prompt
- **The Loop** — wires everything together: seed → prompt → sample → evaluate → store → repeat
- **Problem:** `add(a, b)` with error handling (easy warm-up)

### `2.ipynb` — Harder Problem + Seed Experiments
Applies FunSearch to **string compression** — a problem the LLM can't one-shot:
- **Evaluator** scores by compression ratio (shorter output = higher score, must decompress correctly)
- **Seed importance** — empirically compares seeded vs no-seed runs
- **Debug logging** — shows why candidates fail (wrong decompress, crashes, missing functions)

### `3.ipynb` — Islands + Full System
Adds the remaining FunSearch components:
- **Islands** (`IslandDB`) — multiple separate populations to prevent premature convergence
- **Migration** — copies best program from one island to another every N iterations
- **Island culling** — wipes worst-performing island and re-seeds it
- **Population cap** — limits each island to top-N programs
- **Parallel sampling** (`sample_multiple`) — calls LLM multiple times concurrently via `ThreadPoolExecutor`
- **Per-island temperature** — conservative (0.5), balanced (0.8), wild (1.2) for exploration vs exploitation

## Architecture

```
        Seed
          ↓ init
DB → Best(k) → Build Prompt → LLM → Evaluator
↑________________________________________________↙ store + repeat
```

## Key Insights

1. FunSearch searches **program space**, not solution space
2. The evaluator is the **only source of truth** — no reward model, just code execution
3. Seed quality: **correct but mediocre** is the sweet spot
4. The LLM **one-shots known problems** (knapsack) — FunSearch only helps on unsolved ones
5. **Islands** prevent premature convergence
6. **Temperature per island** = exploration vs exploitation tradeoff
