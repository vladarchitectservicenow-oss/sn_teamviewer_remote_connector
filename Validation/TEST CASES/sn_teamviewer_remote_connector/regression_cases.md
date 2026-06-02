# Regression Cases: sn_teamviewer_remote_connector

**Author:** Vladimir Kapustin  
**License:** AGPL-3.0  
**Date:** 2026-06-02

## Regression Test Cases

### R01 — Idempotent Execution
**Category:** Data Integrity  
**Description:** Running the same report twice with identical parameters produces bit-identical output.  
**Steps:**
1. Run `engine.fetch("incident")` with mocked 3 records → get result A
2. Clear any cache/temp state
3. Run `engine.fetch("incident")` again with same mock → get result B
4. Assert `A == B` (deep equality)
**Expected:** Identical output for identical input.  
**Failure Signal:** Non-deterministic output, timestamp drift, or ordering changes.

### R02 — Format Consistency Across Runs
**Category:** Output Stability  
**Description:** Report format (JSON keys, MD structure) does not drift between versions.  
**Steps:**
1. Generate report with 3 records → record structure S
2. Compare S against schema: `{"total": int, "items": [{"sys_id": str, ...}]}`
3. Assert all keys present, no unexpected fields
**Expected:** Schema stable — no renamed keys, no dropped fields.

### R03 — Backward Compatibility of CLI Interface
**Category:** API Stability  
**Description:** Existing CLI scripts continue to work after minor version bumps.  
**Steps:**
1. Run `python3 src/cli.py --sn-url X --sn-user Y --sn-pass Z --table incident`
2. Assert exit code == 0
3. Assert no new required arguments added
4. Assert all existing flags still accepted
**Expected:** CLI interface additive only — no breaking changes.

### R04 — Empty Table Regression
**Category:** Edge Stability  
**Description:** Empty table responses that worked in previous version still work.  
**Steps:**
1. Mock ServiceNow API returning `{"result": []}`
2. Call `engine.fetch("incident")`
3. Assert returns `[]`, not `None`, not crash
**Expected:** Same behavior as v1.0.0 — silent empty list.

### R05 — Error Handling Path Stability
**Category:** Resilience  
**Description:** Error handling behavior does not regress — exceptions caught, not propagated.  
**Steps:**
1. Induce each error type from v1.0.0 test suite: ConnectionError, HTTPError
2. Assert all return `[]` (not exception propagation)
3. Assert no new error types introduced
**Expected:** All caught — no unhandled exceptions reach caller.

### R06 — Test Suite Self-Consistency
**Category:** Quality Gate  
**Description:** Existing test suite still passes with 0 failures after any code change.  
**Steps:**
1. Run `python3 -m pytest tests/ -v`
2. Assert exit code 0
3. Assert 7 tests collected, 7 passed
**Expected:** 7/7 PASS, <1s execution.

### R07 — File Artifact Cleanup
**Category:** Side Effects  
**Description:** Report generation does not leave stale files from prior runs.  
**Steps:**
1. Generate report to `/tmp/test_r`
2. Verify `test_r.json` and `test_r.md` created
3. Delete files, regenerate
4. Verify new files created (not appending to old)
**Expected:** Clean overwrite — no stale data concatenation.

### R08 — Python Version Compatibility
**Category:** Environment  
**Description:** Code runs on Python 3.10 through 3.12 without syntax errors.  
**Steps:**
1. Check for Python 3.10+ syntax features (match/case, type hints)
2. Verify no deprecated APIs (e.g., `distutils`, `imp`)
3. Run tests on target Python version
**Expected:** No `SyntaxError` or `DeprecationWarning` on Python 3.10-3.12.

## Regression Run Configuration

```bash
# Run full regression suite
cd sn_teamviewer_remote_connector
python3 -m pytest tests/ -v --tb=short --durations=5

# With coverage
python3 -m pytest tests/ -v --cov=src --cov-report=term-missing

# Expected: 7 passed, 100% coverage on engine.py
```
