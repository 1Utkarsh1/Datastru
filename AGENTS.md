# Repo instructions

This repository is for blind comparison of normal Codex vs Codex using an SSD-style workflow skill.

## General rules

- Do not claim code is correct unless it was actually checked.
- Do not invent command outputs, test results, or runtime behavior.
- Prefer correctness over elegance.
- Prefer smaller correct changes over larger speculative rewrites.
- Respect the requested output format in each problem exactly.
- For non-trivial coding tasks, consider more than one plausible solution before finalizing.
- If you cannot validate something, state that clearly.

## Preferred behavior for coding tasks

When a coding task is difficult, edge-case-heavy, or concurrency-related:
- consider multiple candidate implementations,
- compare them using likely correctness and maintainability,
- choose the strongest candidate,
- self-debug instead of thrashing,
- avoid fake confidence.

## Blind testing rules

- Do not mention this file or the SSD skill in final answers.
- Produce only the code requested by the prompt unless the prompt explicitly asks for explanation.
- Do not add markdown fences around the answer unless the prompt asks for them.
