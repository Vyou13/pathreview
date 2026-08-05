# Module 3 Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/156

**Issue title:** README scorer test fixture is too short for its own word-count assertion

**Tier:** [x] Tier 1 [ ] Tier 2 [ ] Tier 3

**Problem summary:**
The `test_readme_with_all_quality_signals` unit test in the README scorer test
suite checks that a scored README reports `word_count > 100` and lands in the
`"comprehensive"` word-count category. The fixture README it feeds the scorer,
however, is only about 51 words long, so the assertion fails even though the
scorer itself is behaving correctly — the test is simply lying to itself about
its input. A successful fix makes the test validate its stated intent, either by
extending the fixture README past the 100-word threshold so it genuinely reaches
the comprehensive category, or by correcting the assertion to match a realistic
fixture. This lives in the agent scoring test layer (`tests/unit/test_readme_scorer.py`)
and touches only test fixtures/assertions, not the scorer logic itself.

**Branch name:** test/156-readme-scorer-fixture-word-count

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

### "Is this right for me?" checklist reasoning

- **Scope:** Tightly bounded — the change is confined to a single test fixture and
  its assertions. No production scorer code needs to change, which keeps the blast
  radius small and the success criterion unambiguous (`pytest tests/unit/test_readme_scorer.py`
  goes green).
- **Skills fit:** Requires reading Python tests and understanding a word-count
  assertion; no new subsystems to learn. Good match for a first Module 3 issue.
- **Clear done state:** The issue gives an exact reproduction command and the exact
  failing assertion (`assert 51 > 100`), so I can verify the fix objectively.
- **Tier 1 / good first issue:** Labeled appropriately for an onboarding-scoped
  contribution, matching where I am in the module.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/Vyou13/pathreview/commit/9cabf41ec7a901d637737c94414c5469f02eb2ff

**Reproduction summary:**
Installed the project dependencies (a missing `structlog` dependency was
initially blocking test collection) and ran `pytest tests/unit/test_readme_scorer.py -q`.
`TestReadmeScorer.test_readme_with_all_quality_signals` fails with `assert 51 > 100`
at `tests/unit/test_readme_scorer.py:56` (1 failed, 22 passed). The captured log
shows the scorer correctly returns `category=minimal word_count=51` for the
fixture — so the scorer is behaving correctly and the fixture README (~51 words)
is simply too short to reach the `word_count > 100` / `"comprehensive"` threshold
the test asserts. The test's own input doesn't match its stated intent.

**PLAN.md link:** https://github.com/Vyou13/pathreview/blob/test/156-readme-scorer-fixture-word-count/PLAN.md

**Walkthrough picture:** ![alt text](image.png)

**Blockers or open questions:**
The scorer lives in `agent/tools/readme_scorer.py` (`ReadmeScorer`) and the
fixture is an inline string in the test method, so the edit is self-contained.
Before writing the extended fixture in Week 9 I still need to confirm from the
scorer source how `word_count` is computed (raw whitespace split vs.
markdown/code stripped — the current 51-word count over a many-line fixture
suggests only prose words count) and the exact category boundary, so I can size
the fixture past 100 words without overshooting the "comprehensive" band or
breaking the other assertions in the same test.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Read `agent/tools/readme_scorer.py` to confirm the two Week 8 unknowns (PLAN.md
steps 1–2). `word_count` is a plain `len(content.split())`, and the category
boundaries are `<100` minimal, `<500` adequate, `>=500` comprehensive — so
reaching `"comprehensive"` requires **500+ words**, not the ~150 I had estimated
in PLAN.md. That's the key correction this week: to satisfy both
`word_count > 100` **and** `word_count_category == "comprehensive"` the fixture
must clear 500 words.

**Next steps:**
Rewrite the inline `readme` fixture to ~500+ words of realistic prose while
preserving every quality signal the test checks (heading, install/usage code
blocks, features + tech-stack lists, two badges, live-demo link), then run the
file and the full unit suite, and open the PR.

**Blockers:**
None.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/844

**Branch:** `test/156-readme-scorer-fixture-word-count`

**What you built:**
Extended the inline fixture in `test_readme_with_all_quality_signals` from ~51 to
~526 words of realistic README prose, keeping every quality signal the test
asserts. The scorer now returns `word_count=526` and
`word_count_category="comprehensive"`, so the test validates its stated intent.
No production code changed — the scorer was already correct; the test's input was
the bug.

**Tests added or updated:**
`tests/unit/test_readme_scorer.py` — updated the fixture in
`test_readme_with_all_quality_signals`. It now genuinely exercises the
`comprehensive` word-count branch and its `overall_score > 0.7` assertion. Suite
goes from `1 failed / 22 passed` to `23 passed`.

**Self-review confirmation:** [ ] make check passes  [ ] make test-unit passes

> Note: repo-wide `make check` and `make test-unit` do **not** pass, but the
> failures are pre-existing and unrelated to this change (other open seeded
> issues such as #149 and #150). For the touched file:
> `pytest tests/unit/test_readme_scorer.py` → 23 passed, and
> `ruff check tests/unit/test_readme_scorer.py` → clean. Verified the same
> unrelated tests fail with my change stashed, and only the readme-scorer test
> flips from red to green.

**Draft PR feedback received from:** none
