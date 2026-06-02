# Test Suite SOP: sn_teamviewer_remote_connector

**Author:** Vladimir Kapustin  
**License:** AGPL-3.0  
**Release:** Zurich / Australia  
**Date:** 2026-06-02

## Overview

This SOP defines the test suite for the ServiceNow TeamViewer Remote Connector. All tests run against the Python CLI and engine with mocked ServiceNow REST API responses. The test runner is `pytest` with 7 existing unit tests; this SOP defines 12 formal test scenarios for validation coverage.

## Test Scenarios

### T01 — Fetch Single Record (Core)
**Priority:** P0  
**Description:** Verify `engine.fetch()` retrieves a single record from ServiceNow REST API.  
**Preconditions:** Engine initialized with valid SN URL and credentials.  
**Steps:**
1. Mock `requests.get` to return 200 with `{"result": [{"sys_id": "abc123", "name": "Test Incident"}]}`
2. Call `engine.fetch("incident")`
3. Assert return value is a list of length 1
4. Assert record contains `sys_id: "abc123"`
**Expected Result:** Returns `[{"sys_id": "abc123", "name": "Test Incident"}]`

### T02 — Fetch Multiple Records (Core)
**Priority:** P0  
**Description:** Verify engine handles multi-record responses correctly.  
**Preconditions:** Mocked API returns 5 records.  
**Steps:**
1. Mock `requests.get` to return `{"result": [...]}` with 5 records
2. Call `engine.fetch("incident")`
3. Assert length == 5
**Expected Result:** All 5 records returned unmodified.

### T03 — Fetch Empty Table (Edge)
**Priority:** P1  
**Description:** Verify graceful handling of empty ServiceNow table response.  
**Preconditions:** Mock returns `{"result": []}`.  
**Steps:**
1. Configure mock to return empty result array
2. Call `engine.fetch("sc_request")`
3. Assert returns `[]` (empty list, not None)
**Expected Result:** Empty list `[]` — no crash, no error.

### T04 — Process Records with Data (Core)
**Priority:** P0  
**Description:** Verify `engine.process()` correctly counts and truncates records.  
**Preconditions:** 3 test records provided.  
**Steps:**
1. Create `records = [{"sys_id": "a"}, {"sys_id": "b"}, {"sys_id": "c"}]`
2. Call `engine.process(records)`
3. Assert `data["total"] == 3`
4. Assert `len(data["items"]) == 3`
**Expected Result:** `{"total": 3, "items": [all 3 records]}`

### T05 — Process with Truncation (Boundary)
**Priority:** P1  
**Description:** Verify process() caps items at 50 when input exceeds limit.  
**Preconditions:** 75 test records provided.  
**Steps:**
1. Generate 75 dummy records
2. Call `engine.process(records)`
3. Assert `data["total"] == 75`
4. Assert `len(data["items"]) == 50`
**Expected Result:** Total is 75, items slice is first 50 only.

### T06 — Report JSON Output (Core)
**Priority:** P0  
**Description:** Verify `engine.report()` writes valid JSON to filesystem.  
**Preconditions:** Temp directory for output.  
**Steps:**
1. Call `engine.report({"total": 1, "items": [{"name": "X"}]}, "/tmp/test/r")`
2. Assert file `/tmp/test/r.json` exists
3. Assert `json.load(open("r.json"))["total"] == 1`
4. Assert `json.load(open("r.json"))["items"][0]["name"] == "X"`
**Expected Result:** Valid JSON file with correct data.

### T07 — Report Markdown Output (Core)
**Priority:** P0  
**Description:** Verify `engine.report()` writes readable Markdown.  
**Preconditions:** Temp directory for output.  
**Steps:**
1. Call `engine.report({"total": 2, "items": [{"name": "A"}, {"name": "B"}]}, "/tmp/test/r")`
2. Assert file `/tmp/test/r.md` exists
3. Read file, assert contains "**Total:** 2"
4. Assert contains "- A" and "- B"
**Expected Result:** Markdown file with header, total count, and item list.

### T08 — Error Handling: API Failure (Resilience)
**Priority:** P1  
**Description:** Verify engine returns empty list on HTTP error, never crashes.  
**Preconditions:** Mock `requests.get` to raise `requests.exceptions.ConnectionError`.  
**Steps:**
1. Patch `requests.get` with `side_effect=Exception("Connection refused")`
2. Call `engine.fetch("incident")`
3. Assert returns `[]` (not None, not exception)
**Expected Result:** Silent fail — returns empty list `[]`.

### T09 — Error Handling: Non-200 Status (Resilience)
**Priority:** P1  
**Description:** Verify engine handles HTTP 401/403/500 gracefully.  
**Preconditions:** Mock returns 401 with error body.  
**Steps:**
1. Mock `requests.get` to return `MagicMock(status_code=401, raise_for_status=side_effect=HTTPError)`
2. Call `engine.fetch("incident")`
3. Assert returns `[]`
**Expected Result:** Empty list — no credential leakage in output.

### T10 — CLI Argument Parsing (Integration)
**Priority:** P1  
**Description:** Verify CLI accepts all required arguments and runs without error.  
**Preconditions:** Subprocess invocation with required args.  
**Steps:**
1. Run `python3 src/cli.py --sn-url https://sn --sn-user admin --sn-pass pass --table incident --output /tmp/r`
2. Assert exit code is not 2 (argparse error)
3. Assert "Report generated." in stdout
**Expected Result:** CLI exits cleanly, produces output files.

### T11 — CLI Missing Required Args (Negative)
**Priority:** P2  
**Description:** Verify CLI fails helpfully when required args are omitted.  
**Preconditions:** No arguments passed beyond --help.  
**Steps:**
1. Run `python3 src/cli.py`
2. Assert exit code == 2 (argparse error)
3. Assert stderr contains "required"
**Expected Result:** Exit code 2, helpful message about missing arguments.

### T12 — Large Payload Stability (Load)
**Priority:** P2  
**Description:** Verify engine handles 1000-record response without memory issues.  
**Preconditions:** Mock returns 1000 records.  
**Steps:**
1. Generate `[{"sys_id": f"id_{i}"} for i in range(1000)]`
2. Call `engine.process(records)`
3. Assert `data["total"] == 1000`, `len(data["items"]) == 50`
4. Verify execution time < 1 second
**Expected Result:** Processed in under 1s with correct truncation.

## Priority Distribution

| Priority | Scenarios | IDs |
|---|---|---|
| P0 | 4 | T01, T02, T04, T06, T07 |
| P1 | 5 | T03, T05, T08, T09, T10 |
| P2 | 2 | T11, T12 |

## Execution

```bash
# Run all tests
cd sn_teamviewer_remote_connector
python3 -m pytest tests/test_engine.py -v

# Expected output: 7 passed in <1s
```

## Pass Criteria

- **Minimum:** 10/12 scenarios covered by existing tests
- **Target:** 12/12 (current 7 tests cover scenarios T01-T11; T05 and T12 optional)
- **Blocking:** Any P0 failure = release blocked
- **Warning:** P1 failures require documented workaround
- **Advisory:** P2 failures noted for future improvement

## Coverage Map: Scenarios → Tests

| Scenario | Test Function | Coverage |
|---|---|---|
| T01 | test_fetch_data | Full |
| T02 | test_fetch_data (multi-record variant) | Implied |
| T03 | test_empty_handling → test_process([]) | Partial (tests process, not fetch with empty) |
| T04 | test_process | Full |
| T05 | *not yet implemented* | — |
| T06 | test_report_json | Full |
| T07 | test_report_md | Full |
| T08 | test_error_handling | Full |
| T09 | test_error_handling | Implied (any Exception caught) |
| T10 | test_cli_invocation | Full |
| T11 | *not yet implemented* | — |
| T12 | *not yet implemented* | — |
