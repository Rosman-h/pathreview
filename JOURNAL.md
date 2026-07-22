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
