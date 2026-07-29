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

## Week 9 — Mid-week progress

**Status:** Core fix for #153 implemented and tested. Added `_safe_chunk_text`
helper to `FaithfulnessChecker` to guard against three failure modes:
non-dict chunks, None text values, and non-string text values (e.g. int).
7 new regression tests added and passing.

**Pre-existing issue found (out of scope):** While testing, found that
`_is_supported()` has a stricter overlap threshold than 3 existing tests
assume — `test_partial_support_returns_middle_score`,
`test_multiple_context_chunks`, and `test_multiple_claims_varying_support`
fail even on the original, unmodified code (confirmed via `git stash`).
This appears unrelated to #153 and is not something I'm fixing in this PR
to keep scope contained. Flagging here and will mention in the PR
description in case a maintainer wants a separate issue filed.

**Remaining for this week:** Final documentation pass, full test suite run,
and PR submission.

## Week 9 — Submission

**PR link:** [[paste your actual PR URL once opened](https://github.com/ascherj/pathreview/pull/369)]

**Summary:** Implemented `_safe_chunk_text()` in `FaithfulnessChecker` to
guard against non-dict chunks, None text, and non-string text — closing
the remaining gaps from the original #153 crash. Added 7 regression tests
covering these cases, all passing.

**Verification:** Ran the full test suite before and after this change
(via checkout of the prior commit) — confirmed identical 52 pre-existing
failures unrelated to this work in both cases, and this PR adds 7 new
passing tests with no regressions.

**Known out-of-scope issues found and documented (not fixed):**

- 3 pre-existing test failures in this file caused by `_is_supported()`'s
  overlap threshold, unrelated to #153
- Pre-existing missing type annotations / unused variable in this test
  file, flagged to reviewers rather than silently fixed

**Reflection:** The core fix was small, but confirming scope (what's mine
to fix vs. pre-existing) took the most time this week. Worth it — avoided
scope creep into unrelated code.
