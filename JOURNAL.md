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

**PLAN.md link:** https://github.com/Vyou13/pathreview/blob/test/156-readme-scorer-fixture-word-count/plan.md

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
