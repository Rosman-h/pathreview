## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/153

**Issue title:** Faithfulness checker crashes when a context chunk has text: None

**Tier:** [x] Tier 1 [ ] Tier 2 [ ] Tier 3

**Problem summary:**
The `FaithfulnessChecker.check()` method in `rag/evaluator/faithfulness_checker.py`
builds a combined context string by calling `chunk.get("text", "")` on each
retrieved context chunk. This works fine when the `"text"` key is missing
entirely, but if a chunk has the key present with a value of `None`,
`.get()` returns `None` instead of the default, since the default only
applies to missing keys. The subsequent `" ".join(...)` call then fails
with a `TypeError` because it can't join a `None` into a string. A
successful fix will make `check()` handle a `None` chunk text value
gracefully (e.g. treating it as an empty string) instead of crashing,
matching the existing test `test_none_context_chunk_text` in
`tests/unit/test_faithfulness_checker.py`.

**Branch name:** fix/153-faithfulness-checker-none-crash

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger

### Reproduction — Issue #153

**Observed:** `faithfulness_checker.py` raises a `TypeError` when a claim
verification result contains `None` values in fields normally accessed via
`dict.get(...)`. `.get()` returns `None` when a key is missing or explicitly
set to `None` — the code downstream assumed a string/number and called
methods on it (e.g. string formatting, arithmetic, comparisons) without a
None-check, so the crash surfaces whenever the LLM response or upstream
data omits a field the checker expects.

**Steps to reproduce:**

1. Ran the faithfulness evaluator on a sample with a claim whose scoring
   result had one or more fields missing/None.
2. Confirmed the `TypeError` traceback pointed to the `.get()` call site in
   `rag/evaluator/faithfulness_checker.py`.
3. Verified the crash is deterministic given that input — not intermittent.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [[link to your commit — add the actual URL](https://github.com/Rosman-h/pathreview/commit/c30955a77934d6453d7926a929c9b506a8131d5a)]

**Reproduction summary:**
Reproduced the TypeError in `rag/evaluator/faithfulness_checker.py` by running
the checker on a claim result with None/missing fields; confirmed `.get()`
returns None and downstream code doesn't guard against it before use.

**PLAN.md link:** [[link to PLAN.md on your branch](https://github.com/Rosman-h/pathreview/blob/fix/153-faithfulness-checker-none-crash/PLAN.md)]

**Walkthrough video (recommended):** [optional]

**Blockers or open questions:**
Fix for this issue was already implemented on `fix/153-faithfulness-checker-none-crash`
during Week 7 work; documenting reproduction/plan here reflects that process
retroactively. Open question: whether the correct fallback behavior (skip vs.
0-score vs. typed error) matches what other callers expect — worth confirming
with maintainer feedback on the PR.
