# Blind Test Template

Use this template when sending two solutions for blind judging.
Do not reveal which version used the skill.

```text
Problem: H1

Original prompt:
[paste the full prompt exactly]

Version A:
[paste the full code only]

Version B:
[paste the full code only]

Optional validation for A:
[paste tests / output / lint / runtime result]

Optional validation for B:
[paste tests / output / lint / runtime result]
```

## Suggested judging criteria

Score each version by:
- correctness against the prompt,
- edge-case handling,
- robustness,
- maintainability,
- unnecessary complexity,
- safety under failure conditions,
- likelihood of surviving real repo tests.

## Good testing protocol

- Use the exact same problem prompt for both runs.
- Use a clean environment if possible.
- Run 3 attempts per setup if you want stronger evidence.
- Keep only the strongest final answer from each setup.
- Ask for blind judging only after labels are removed.
