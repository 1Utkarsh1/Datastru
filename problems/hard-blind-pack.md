# Hard Blind Pack for Codex vs SSD-Skill Codex

These problems are intentionally difficult.
They are designed to expose differences in edge-case handling, concurrency safety, robustness, and maintainability.

## Instructions

For each problem:
- run plain Codex on the prompt,
- run Codex with the SSD skill on the same prompt,
- do not tell the judge which version used the skill,
- keep the final code only unless the prompt asks for explanation.

For stronger signal:
- do 3 runs per setup,
- keep the best final answer from each setup,
- compare the best normal answer vs the best skill answer.

---

## H1 — Async fair limiter with cancellation safety

```text
You are given a Python module that implements an async concurrency limiter, but it has two production bugs:

1. tasks can starve under heavy contention,
2. if a waiting task is cancelled at the wrong time, capacity can be leaked or FIFO order can break.

Implement a production-ready limiter with this API:

class FairLimiter:
    def __init__(self, limit: int): ...
    async def acquire(self) -> None: ...
    def release(self) -> None: ...
    @property
    def in_use(self) -> int: ...
    @property
    def queued(self) -> int: ...

Requirements:
- Strict FIFO fairness for waiters.
- If a waiter is cancelled before acquiring, it must be removed cleanly.
- If cancellation happens right as the waiter is being woken, capacity must not leak.
- release() must wake exactly one next valid waiter if any exist.
- No busy waiting.
- No polling loop.
- Must work under Python asyncio.
- Use time.monotonic() only if timing is needed.
- Raise ValueError if limit <= 0.
- Releasing when nothing is acquired must raise RuntimeError.

Edge cases:
- many waiters cancelled in the middle of the queue,
- repeated acquire/release bursts,
- cancellation storms,
- alternating fast and slow tasks,
- double release bug.

What a weak solution usually does wrong:
- uses a plain semaphore and claims fairness,
- forgets cancellation cleanup,
- leaks permits,
- wakes more than one waiter,
- mutates the queue unsafely.

Output:
Write the full implementation only.
```

---

## H2 — Incremental JSON stream parser with UTF-8 chunk boundaries

```text
Implement a Python class that incrementally parses a stream of newline-delimited JSON objects from arbitrary byte chunks.

Public API:

class NDJSONStreamParser:
    def feed(self, chunk: bytes) -> list[object]:
        """Consumes bytes and returns all complete parsed JSON objects now available."""
    def close(self) -> list[object]:
        """Flushes the stream. Raise ValueError on incomplete trailing JSON record."""

Requirements:
- Input arrives as arbitrary byte chunks.
- Chunks may split:
  - UTF-8 multibyte characters,
  - escaped sequences,
  - the newline delimiter itself.
- Each record is a single JSON value separated by \n.
- Blank lines should be ignored.
- Leading/trailing spaces around each JSON value are allowed.
- Invalid JSON must raise ValueError and preserve no corrupted internal state.
- After close(), future feed() calls must raise RuntimeError.

Edge cases:
- multibyte unicode split across chunks,
- \\n inside a JSON string must not act as a record delimiter,
- last record incomplete at EOF,
- chunk contains multiple records,
- chunk contains half of one record.

What a weak solution usually does wrong:
- decodes each chunk independently,
- breaks UTF-8 at chunk boundaries,
- splits on byte newline without record logic,
- mishandles escaped newlines inside strings.

Output:
Write the full implementation only.
```

---

## H3 — SQL query builder with precedence, injection safety, and stable parameters

```text
Write a Python function that builds a parameterized SQL query for a tickets table.

Function:

def build_ticket_query(
    status: list[str] | None = None,
    assignee_ids: list[int] | None = None,
    search: str | None = None,
    created_from: str | None = None,
    created_to: str | None = None,
    include_deleted: bool = False,
    sort_by: str = "created_at",
    descending: bool = True,
    limit: int = 50,
    offset: int = 0,
) -> tuple[str, list[object]]:
    ...

Table columns:
- id
- title
- body
- status
- assignee_id
- created_at
- deleted_at

Requirements:
- Return SQL plus parameters.
- SQL must be safe for parameterized execution.
- status and assignee_ids filters use IN (...).
- search should search case-insensitively in title and body.
- include_deleted=False means only rows where deleted_at IS NULL.
- created_from is inclusive.
- created_to is exclusive.
- Allowed sort_by values: created_at, status, id.
- Any other sort_by must raise ValueError.
- Negative limit or offset must raise ValueError.
- Preserve deterministic parameter order.

Important logic rule:
The search clause must be grouped so that it does not break the meaning of the other filters.

What a weak solution usually does wrong:
- interpolates raw values into SQL,
- forgets to validate sort_by,
- gets AND/OR precedence wrong,
- generates unstable parameter ordering,
- mishandles empty lists.

Output:
Write the full function only.
```

---

## H4 — In-flight request coalescing cache with TTL and stale-while-revalidate

```text
Implement an async Python cache that supports:
- TTL,
- stale-while-revalidate,
- in-flight request coalescing.

API:

class AsyncSWRCache:
    def __init__(self, ttl_seconds: float, stale_seconds: float): ...
    async def get(self, key: str, loader) -> object:
        """
        loader is an async callable with no args.
        """
    def invalidate(self, key: str) -> None: ...
    def clear(self) -> None: ...

Semantics:
- If key is fresh, return cached value.
- If key is expired but still within stale window:
  - return stale value immediately,
  - trigger exactly one background refresh for that key.
- If key is fully stale or absent:
  - callers should await a loader result,
  - concurrent callers for the same key must share one in-flight loader call.
- Exceptions from loader must not poison the cache forever.
- If refresh fails and stale data exists, stale data may remain until stale window ends.
- No global lock that serializes unrelated keys.

Edge cases:
- 50 concurrent callers for same key,
- invalidate during refresh,
- loader exception,
- refresh race,
- clear while refresh task exists.

What a weak solution usually does wrong:
- starts multiple refreshes,
- returns stale forever,
- uses one global lock,
- loses cache consistency on exceptions,
- cannot reason about key-local concurrency.

Output:
Write the full implementation only.
```

---

## H5 — Gitignore-style matcher with **, negation, escapes, and normalized paths

```text
Implement a Python matcher that behaves like a simplified .gitignore.

API:

class IgnoreMatcher:
    def __init__(self, patterns: list[str]): ...
    def matches(self, path: str, is_dir: bool = False) -> bool: ...

Pattern rules:
- # starts a comment unless escaped.
- blank lines ignored.
- leading ! negates a previous ignore rule.
- trailing / means directory-only match.
- * matches within a path segment.
- ** may cross directory boundaries.
- paths should be normalized to forward slashes internally.
- duplicate slashes should not matter.
- \ can escape special prefix characters in patterns.

Semantics:
- Later rules override earlier ones.
- The final matching rule decides the result.
- Matching should work for nested directories and files.
- Rooted vs non-rooted behavior should be handled consistently.

Edge cases:
- foo/**/bar.py
- build/
- !build/keep.txt
- escaped \#literal
- Windows-style input paths
- repeated separators

What a weak solution usually does wrong:
- treats ** same as *,
- ignores rule ordering,
- breaks negation,
- mishandles normalization,
- incorrectly applies directory-only rules.

Output:
Write the full implementation only.
```

---

## H6 — Rule engine with dependency ordering and cycle detection

```text
Implement a small rule engine in Python.

Each rule has:
- a unique string name,
- a list of dependency rule names,
- a callable fn(context, resolved_results) -> object.

API:

from dataclasses import dataclass
from typing import Callable, Any

@dataclass
class Rule:
    name: str
    deps: list[str]
    fn: Callable[[dict, dict], Any]

class RuleEngine:
    def __init__(self, rules: list[Rule]): ...
    def run(self, context: dict) -> dict[str, object]: ...

Requirements:
- Evaluate rules in dependency-safe order.
- resolved_results passed to a rule should include only its already-resolved dependencies.
- Missing dependency must raise KeyError.
- Duplicate rule names must raise ValueError.
- Cycles must raise ValueError with a useful message.
- Execution order should be deterministic.
- Do not recurse deeply if an iterative topological approach is cleaner.

Edge cases:
- diamond dependency graphs,
- self-dependency,
- disconnected subgraphs,
- cycle of length > 2,
- rules list in arbitrary order.

What a weak solution usually does wrong:
- forgets duplicate detection,
- gives bad cycle handling,
- produces nondeterministic order,
- passes too much state into each rule,
- uses recursion carelessly.

Output:
Write the full implementation only.
```

---

## H7 — Versioned document store with optimistic concurrency

```text
Implement an in-memory Python document store with optimistic concurrency control.

API:

class ConflictError(Exception): ...
class NotFoundError(Exception): ...

class DocumentStore:
    def create(self, doc_id: str, data: dict) -> int: ...
    def get(self, doc_id: str) -> tuple[dict, int]: ...
    def update(self, doc_id: str, expected_version: int, patch: dict) -> int: ...
    def delete(self, doc_id: str, expected_version: int) -> None: ...

Requirements:
- create stores a deep copy and returns version 1.
- get returns a deep copy and current version.
- update applies a shallow merge of patch into the current document only if expected_version matches.
- successful update increments version by 1.
- stale writes raise ConflictError.
- missing docs raise NotFoundError.
- external mutation of returned objects must not mutate the store.

Edge cases:
- repeated stale updates,
- delete with wrong version,
- nested mutable values,
- update after delete,
- caller mutates returned dict after get.

What a weak solution usually does wrong:
- forgets deep copy isolation,
- mishandles version checks,
- mutates stored state through references,
- confuses missing vs conflict.

Output:
Write the full implementation only.
```

---

## H8 — Retry policy with jitter, classification, and deadline budget

```text
Implement a Python retry helper for async callables.

API:

import asyncio
from typing import Callable, Awaitable, Any

async def retry_async(
    fn: Callable[[], Awaitable[Any]],
    *,
    attempts: int = 5,
    base_delay: float = 0.1,
    max_delay: float = 2.0,
    jitter: float = 0.1,
    retry_if=None,
    deadline: float | None = None,
) -> Any:
    ...

Requirements:
- Retry up to attempts.
- Use exponential backoff capped by max_delay.
- Add bounded jitter.
- retry_if(exc) decides whether an exception is retryable. If retry_if is None, retry everything except asyncio.CancelledError.
- Never swallow CancelledError.
- Respect total deadline budget if provided.
- If remaining deadline budget is too small for another delay+attempt, stop and raise the last exception.
- Validate arguments and raise ValueError on invalid settings.

Edge cases:
- attempts=1,
- non-retryable exception,
- cancellation during sleep,
- deadline exhausted,
- jitter would exceed bounds,
- huge backoff values.

What a weak solution usually does wrong:
- retries cancellation,
- ignores total deadline,
- computes jitter badly,
- oversleeps beyond deadline,
- off-by-one on attempts.

Output:
Write the full implementation only.
```
