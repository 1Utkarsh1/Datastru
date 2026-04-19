# SSD Codex Blind Test Repo

This repo is a small test harness for comparing plain Codex vs Codex using an SSD-style skill.

It contains:
- a detailed Codex skill in `.agents/skills/ssd-codegen/SKILL.md`
- a hard blind test pack in `problems/hard-blind-pack.md`
- a copy-paste template for blind judging in `templates/blind-test-template.md`
- a repo-level `AGENTS.md` that pushes Codex toward disciplined validation

## Goal

The goal is to test whether the SSD-style workflow produces better code than normal one-shot code generation.

The skill does **not** retrain OpenAI's hosted Codex weights.
Instead, it changes how Codex works on coding tasks:
- consider multiple real candidates
- validate instead of guessing
- compare approaches using evidence
- self-debug carefully
- avoid fake confidence

## Repo structure

```text
.agents/
  skills/
    ssd-codegen/
      SKILL.md
AGENTS.md
problems/
  hard-blind-pack.md
templates/
  blind-test-template.md
README.md
```

## How to use this repo

### 1. Clone the repo

```bash
git clone https://github.com/1Utkarsh1/Datastru.git
cd Datastru
```

### 2. Open the repo in Codex

Open this repo in the Codex environment you want to test.

Because the skill lives under `.agents/skills/ssd-codegen/SKILL.md`, Codex can discover it when working inside the repo.

### 3. Run a fair comparison

For each problem in `problems/hard-blind-pack.md`:

- run **plain Codex** on the exact problem prompt
- run **Codex with the SSD skill active** on the exact same prompt
- do this in a clean state if possible
- keep the final answer from each side
- label them only as **Version A** and **Version B**

Do not tell the judge which version used the skill.

### 4. Use multiple runs

For stronger signal:
- run each problem 3 times per setup
- keep the strongest final answer from each setup
- compare the best normal run vs the best skill run

### 5. Judge blind

Paste both answers into the template in `templates/blind-test-template.md` and compare them blindly.

## Recommended first problems

Use these first because they usually expose the biggest difference:
- H4 async cache with TTL plus stale-while-revalidate plus request coalescing
- H1 async fair limiter with cancellation safety
- H5 gitignore-style matcher
- H2 incremental NDJSON parser with chunked UTF-8 boundaries
- H8 async retry helper with deadline budget

## What usually separates the better version

The better version usually has:
- fewer hidden edge-case bugs
- better failure semantics
- cleaner concurrency control
- less fake confidence
- smaller unnecessary churn
- clearer reasoning in code structure

## Notes

This is a custom private-style blind pack.
It is intentionally shaped like repo and systems work rather than toy interview problems.
That makes it better for comparing real coding workflows.
