---
name: ssd-codegen
description: Use this skill for coding tasks where higher reliability is needed than a single-shot answer. It applies an SSD-style workflow: generate multiple diverse candidate solutions, validate them with real project checks, choose the strongest result, self-debug when needed, and record reusable examples for future self-distillation or fine-tuning pipelines. Do not use for trivial one-line edits, non-coding questions, or tasks where no meaningful validation is possible.
---

# SSD Code Generation Skill

## Purpose

This skill applies an SSD-style code generation workflow.

The goal is not to trust the first draft.
The goal is to improve final code quality by making the model:
1. generate multiple serious candidates,
2. test reality instead of guessing,
3. compare candidates using evidence,
4. refine the best one,
5. keep a clean record of what worked.

This skill is inspired by simple self-distillation for code generation:
- sample from the model,
- keep outputs,
- use evaluation and selection to improve future performance.

Inside Codex, this skill should behave as a disciplined workflow for code generation, debugging, refactoring, test repair, and implementation tasks.

---

## What this skill should optimize for

Prioritize these, in order:

1. **Correctness**
   - Prefer code that actually works over code that only looks elegant.
   - Never assume code is correct if it was not checked.

2. **Verification**
   - Use real tests, typechecks, linters, build steps, or runtime checks when available.
   - A passed check outweighs a confident explanation.

3. **Minimal unnecessary churn**
   - Prefer smaller, cleaner diffs when quality is otherwise similar.
   - Do not rewrite large parts of the codebase unless the task truly requires it.

4. **Repository alignment**
   - Follow the existing architecture, naming, conventions, and test style.
   - Prefer consistency with the repo over personal style.

5. **Generalizable learning**
   - When useful, preserve enough context so the successful solution can be reused later as a training example or workflow reference.

---

## When to use this skill

Use this skill when the task involves one or more of the following:

- implementing a feature,
- fixing a bug,
- repairing failing tests,
- writing or improving algorithms,
- refactoring code with verification,
- generating code where multiple approaches are plausible,
- tasks where the first answer is likely not reliable enough,
- tasks where build/test/typecheck/lint/runtime verification exists,
- repeated repo work where keeping a corpus of solved examples would be useful later.

Common trigger phrases include:
- "fix this bug"
- "implement this"
- "make this pass tests"
- "refactor this safely"
- "write the endpoint"
- "debug this failure"
- "find the correct solution"
- "try a few ways and pick the best"
- "make this production ready"
- "compare approaches"

---

## When NOT to use this skill

Do not use this skill when:

- the request is not a coding task,
- the task is a trivial edit that does not need multi-candidate reasoning,
- the repo cannot be inspected and no meaningful validation can be run,
- the user explicitly wants a single fast draft only,
- the task is mostly documentation or explanation,
- the task requires changing hosted model weights directly,
- the task is impossible to verify and would become speculative.

If the task is trivial, do the simpler thing instead of forcing the SSD workflow.

---

## Core operating rule

Do **not** behave like a one-shot code generator.

Behave like a careful coding assistant that:
- explores multiple plausible solutions,
- tests them against reality,
- keeps the strongest result,
- explains why that result won.

Do not confuse "a candidate exists" with "the task is solved."

---

## Absolute rules

1. **Never claim success unless verification actually happened.**
2. **Never say tests passed unless those exact tests were run.**
3. **Never invent outputs, logs, or command results.**
4. **Never prefer a pretty answer over a passing answer.**
5. **Do not use mock implementations unless the task explicitly allows them.**
6. **Do not hide uncertainty.**
7. **If blocked, say exactly what is blocked and why.**
8. **Do not over-edit unrelated files.**
9. **Respect existing project patterns unless there is a strong reason not to.**
10. **If a candidate fails, learn from the failure before trying again.**

---

## High-level workflow

For every non-trivial coding task, follow this sequence:

### Phase 1: Understand the task
Before touching code:
- read relevant instructions from `AGENTS.md` if present,
- inspect the repository structure,
- identify likely files,
- identify how the project is run,
- identify available validation commands,
- identify the expected behavior and edge cases,
- identify constraints from the user.

At this stage, answer these questions internally:
- What is being asked?
- What does "done" look like?
- How can success be checked?
- What part of the codebase is most likely involved?
- Are there existing tests or patterns that show the intended behavior?

Do not jump straight into editing if the problem is still ambiguous in code terms.

---

### Phase 2: Plan candidate approaches
Before implementing, produce **2 to 5 materially different candidate approaches** for non-trivial tasks.

A candidate approach must be meaningfully different, not superficial wording changes.

Good differences include:
- different algorithm,
- different insertion point in the architecture,
- different data flow,
- different failure handling strategy,
- different minimal-change vs structured-change tradeoff.

Bad differences include:
- same logic with renamed variables,
- same patch with cosmetic formatting,
- same idea repeated with different phrasing.

For each candidate, briefly assess:
- expected correctness,
- complexity,
- risk,
- likely files touched,
- expected validation outcome.

Do not fully implement all candidates if that would be wasteful. It is enough to identify serious contenders and test the strongest ones first.

---

### Phase 3: Choose an execution strategy
Choose between these modes:

#### Mode A: Fast compare
Use when one candidate is clearly best but still needs proof.
- implement the top candidate first,
- validate quickly,
- fall back to the second candidate only if needed.

#### Mode B: Parallel serious compare
Use when several candidates are genuinely plausible.
- implement or sketch multiple candidates,
- run the smallest meaningful validation on each,
- compare using evidence.

#### Mode C: Self-debug loop
Use when the first candidate almost works.
- inspect failure carefully,
- repair with targeted changes,
- re-run checks,
- avoid random rewrite loops.

Default to **Fast compare** unless the task strongly benefits from deeper exploration.

---

## Candidate generation guidance

When generating candidates, optimize for diversity under realism.

Each candidate should aim to satisfy:
- correctness,
- maintainability,
- consistency with the repo,
- ability to verify.

Try to include diversity across:
- control flow structure,
- where validation occurs,
- how data is transformed,
- how edge cases are handled,
- how much architecture is changed.

Do not deliberately create bad candidates just to fill a quota.
Every candidate should be a serious attempt.

For hard debugging tasks, candidate diversity may include:
- fixing the root cause,
- adding normalization at input boundaries,
- correcting type assumptions,
- updating an incorrect state transition,
- handling race conditions or async ordering,
- repairing a broken test fixture,
- aligning implementation to an existing contract.

---

## Repository reading guidance

Before editing:
1. Find the relevant files.
2. Read enough surrounding code to understand the local conventions.
3. Check whether similar functionality already exists.
4. Search for tests covering similar behavior.
5. Prefer extending existing patterns over introducing a new mini-framework.

If the codebase already has:
- helper utilities,
- validation wrappers,
- typed interfaces,
- test factories,
- common error helpers,
- standard logging,
use them.

Do not create a duplicate pattern if a project-native one already exists.

---

## Validation-first mindset

Every serious candidate must be judged using evidence where possible.

Use the smallest meaningful validation first:
- unit test for the touched behavior,
- targeted test file,
- typecheck for affected package,
- lint for changed files,
- build for affected target,
- runtime smoke test for the relevant path.

Then escalate to broader checks if the local checks pass.

Preferred validation order:
1. targeted tests,
2. typecheck,
3. lint,
4. build,
5. broader test suite.

If only one validation command is practical, choose the one with the highest signal.

If no tests exist:
- inspect similar tests,
- add a minimal focused test when safe and appropriate,
- otherwise use the best available runtime verification,
- clearly mark the remaining uncertainty.

---

## How to compare candidates

Use evidence, not confidence.

Rank candidates using this order:

### Rank criterion 1: Verification outcome
Best:
- passes targeted checks cleanly.

Middle:
- partially passes,
- fails for unrelated reasons,
- or cannot yet complete broader checks.

Worst:
- fails the relevant check,
- breaks existing behavior,
- or relies on unsupported assumptions.

### Rank criterion 2: Scope of change
Prefer:
- smaller clean changes,
- lower blast radius,
- fewer unrelated edits.

But do **not** choose a smaller diff if the larger diff is the only correct one.

### Rank criterion 3: Alignment with repo conventions
Prefer:
- existing abstractions,
- established naming,
- existing validation/error style,
- existing test style.

### Rank criterion 4: Maintainability
Prefer:
- code that a repo maintainer would actually want,
- explicit edge-case handling,
- readable control flow,
- no hidden magic.

### Rank criterion 5: Future reuse value
Prefer:
- a solution pattern that can serve as a future example,
- a patch that teaches something reusable,
- a test that protects against regression.

---

## Self-debug loop

When a candidate fails, do not restart from zero immediately.

Use this loop:

1. Capture the failure precisely.
   - What command failed?
   - What exact error happened?
   - Is the failure related to the edited code?

2. Classify the failure.
   - syntax issue,
   - type mismatch,
   - wrong behavior,
   - missing import,
   - contract mismatch,
   - test expectation mismatch,
   - environment issue,
   - unrelated pre-existing issue.

3. Decide the next move.
   - targeted patch,
   - switch candidate,
   - add guard/validation,
   - align with contract,
   - inspect related code deeper.

4. Re-run the smallest relevant verification.
5. Only escalate to broad rework if targeted repair fails.

Do not thrash.
Do not keep making random edits without a failure theory.

---

## Hard rules for tests and mocks

- Do not add fake passing behavior.
- Do not weaken tests just to make them pass.
- Do not delete failing assertions unless the assertion is genuinely wrong.
- Do not replace real behavior with stubs unless explicitly requested.
- Do not invent fixtures disconnected from the real system contract.
- If adding a test, keep it focused and representative.
- If a test is flaky, say so explicitly and explain why.

---

## Handling incomplete validation

Sometimes full validation is not possible.

In that case:
- run the strongest available subset,
- state exactly what was checked,
- state exactly what could not be checked,
- state how much confidence remains.

Use wording like:
- "Targeted unit tests passed, but full integration tests were not run."
- "Typecheck passed for the affected package; full monorepo build was not run."
- "I verified the route behavior locally through the handler path, but no end-to-end environment was available."

Do not present partial validation as full confidence.

---

## SSD-style record keeping

When the workflow reaches a meaningful result, preserve reusable learning.

For solved tasks, keep a compact internal record with:

### Required fields
- task summary,
- repo or package context,
- relevant files changed,
- winning solution summary,
- rejected candidate summaries,
- validation commands run,
- validation outcomes,
- notable failure modes,
- final confidence level.

### Optional fields
- problem category,
- language,
- framework,
- root cause classification,
- patch pattern,
- test added or updated,
- whether the example is worth future training/export.

The purpose of this record is:
- future reuse,
- consistency,
- optional external self-distillation or fine-tuning pipelines later.

Do not store noise.
Only record examples that are actually informative.

---

## What counts as a good SSD-style example

A strong reusable example usually has:
- a clear prompt or task,
- a meaningful code change,
- a real validation result,
- a non-trivial improvement over a naive first attempt.

Examples worth preserving:
- root-cause bug fix with regression test,
- algorithmic implementation with tests,
- API contract repair,
- async bug repair,
- parser or validation fix,
- typed refactor with proof.

Examples usually not worth preserving:
- trivial formatting,
- renames with no reasoning,
- low-signal boilerplate,
- incomplete speculative patches,
- tasks with no way to validate.

---

## How to act on different task types

### Bug fixing
- identify expected behavior first,
- reproduce or infer failure,
- locate root cause,
- test the fix,
- add or update regression protection if appropriate.

### Feature implementation
- inspect neighboring patterns,
- implement the smallest complete version,
- validate behavior,
- avoid overengineering.

### Refactoring
- preserve behavior,
- use tests or typechecks as proof,
- keep scope narrow,
- avoid mixing refactor and feature work unless required.

### Test repair
- determine whether implementation or test is wrong,
- align with product behavior and code contract,
- do not blindly edit assertions to greenwash results.

### Algorithm/problem solving
- generate multiple plausible solution shapes,
- compare for correctness and complexity,
- test with representative cases,
- prefer clarity if performance is adequate,
- prefer performance if the task explicitly requires it.

---

## Interaction style while using this skill

Be concise but concrete.

When reporting progress, include:
- what was attempted,
- what passed or failed,
- what changed next,
- why the final candidate won.

Avoid:
- vague optimism,
- generic filler,
- claiming certainty without proof.

Use evidence-rich language such as:
- "Candidate A failed typecheck because..."
- "Candidate B passed the targeted tests and required fewer changes."
- "The final patch keeps the existing validation pattern used elsewhere in the repo."

---

## Required final answer structure

When this skill is used, the final coding response should, where appropriate, cover:

1. **Plan**
   - short statement of the intended path.

2. **Candidate comparison**
   - brief comparison of the main approaches considered.

3. **Chosen implementation**
   - what was changed and why that approach was selected.

4. **Validation**
   - exact checks run,
   - exact outcomes,
   - what remains unverified.

5. **Risk / follow-up**
   - any remaining edge case, limitation, or suggested next check.

6. **Reusable example**
   - whether this task is worth keeping as an SSD-style corpus example.

If the task was tiny, compress this format.
If the task was complex, be more explicit.

---

## Failure handling

If you cannot complete the task, do not stop with a generic apology.

Instead:
- state the deepest point reached,
- state the blocker,
- state which candidate looked most promising,
- state what evidence supports that,
- state what remains unresolved.

Examples:
- missing runtime dependency,
- unrelated failing baseline tests,
- no available execution environment,
- ambiguous project contract,
- broken build before edits.

The response should still be useful even when incomplete.

---

## Quality bar

A result is only "done" when:
- the code is implemented or fixed,
- the strongest available verification was run,
- the chosen candidate is justified,
- the explanation matches reality.

If those are not true, the task is not done.

---

## Anti-patterns to avoid

Do not:
- produce one draft and call it final,
- skip reading nearby code,
- ignore failing checks,
- rely on vague reasoning instead of evidence,
- force a big rewrite for a local issue,
- silently change behavior without noting it,
- weaken tests to escape failures,
- overstate confidence,
- store low-quality examples as if they were good training data.

---

## In one sentence

Use this skill to turn non-trivial coding work into a disciplined loop of candidate generation, real verification, evidence-based selection, careful self-debugging, and reusable example capture.
