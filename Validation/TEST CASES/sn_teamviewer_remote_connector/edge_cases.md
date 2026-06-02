# Edge Cases: sn_teamviewer_remote_connector

**Author:** Vladimir Kapustin  
**License:** AGPL-3.0  
**Date:** 2026-06-02

## E01 — Empty Table Response
**Severity:** P1  
**Scenario:** ServiceNow table contains zero records at query time.  
**Input:** `fetch("incident")` → API returns `{"result": []}`  
**Expected Behavior:** Returns `[]` (empty list). `process([])` returns `{"total": 0, "items": []}`. `report()` writes files with zero items.  
**Actual (v1.0):** ✓ Handled correctly — tests verify empty list path.  
**Risk if unhandled:** `None`-type errors, report generation crash.

## E02 — Record Exceeds Truncation Limit (50+ records)
**Severity:** P2  
**Scenario:** ServiceNow table returns >50 records — `process()` truncates items to first 50.  
**Input:** 75 records returned from API.  
**Expected Behavior:** `{"total": 75, "items": [first 50]}` — total reflects reality, items are head slice.  
**Actual (v1.0):** ✓ Truncation via `records[:50]`. Total count preserved.  
**Risk if unhandled:** Memory pressure on very large tables (10k+ records).

## E03 — Malformed ServiceNow URL
**Severity:** P2  
**Scenario:** User passes invalid URL (`--sn-url not-a-url`) or URL without scheme.  
**Input:** `Engine("not_a_url", "user", "pass")` → `fetch()` constructs `not_a_url/api/now/table/incident`.  
**Expected Behavior:** `requests.get()` raises `requests.exceptions.MissingSchema` → caught by try/except → returns `[]`.  
**Actual (v1.0):** ✓ Silent fail via generic `except Exception`. Report file still created but empty.  
**Risk if unhandled:** Uncaught exception crashes CLI mid-run.

## E04 — Special Characters in Record Data
**Severity:** P3  
**Scenario:** ServiceNow record contains Unicode, newlines, or JSON-breaking characters in field values.  
**Input:** `{"sys_id": "x", "name": "Incident: é—🔥\nnewline"}`  
**Expected Behavior:** JSON serialized correctly with `ensure_ascii=False`. Markdown may have newline issues in list items.  
**Actual (v1.0):** JSON handles unicode correctly. Markdown may break on embedded newlines.  
**Risk if unhandled:** Garbled report output, JSON parse errors on re-read.

## E05 — Network Timeout During Fetch
**Severity:** P1  
**Scenario:** ServiceNow instance is slow or unreachable — request hangs for 30s.  
**Input:** `requests.get(..., timeout=30)` with API that never responds.  
**Expected Behavior:** `requests.exceptions.Timeout` → caught by except → returns `[]`.  
**Actual (v1.0):** ✓ Timeout parameter set to 30s. Exception caught.  
**Risk if unhandled:** Indefinite hang, cron job stall.

## E06 — Concurrent Report Generation
**Severity:** P3  
**Scenario:** Two CLI instances write to same output prefix simultaneously.  
**Input:** Process A writes `report.json`, Process B writes `report.json` at same time.  
**Expected Behavior:** One process wins (last writer); other data lost. No crash.  
**Actual (v1.0):** ✓ Python file writes are atomic at OS level. No locking — last writer wins silently.  
**Risk if unhandled:** Stale data if one process finishes before other starts — acceptable for cron use.

## E07 — None/Missing Field Values in Records
**Severity:** P2  
**Scenario:** ServiceNow record has `null` values for fields like `name`, `assigned_to`.  
**Input:** `{"sys_id": "abc", "name": None}`  
**Expected Behavior:** `report()` uses `.get('name', '')` → outputs empty string. No crash.  
**Actual (v1.0):** ✓ `.get('name', i.get('sys_id', ''))` — falls back to sys_id when name is None.  
**Risk if unhandled:** `NoneType` string formatting error in report line generation.

## E08 — CLI --output Path with No Write Permission
**Severity:** P2  
**Scenario:** User specifies output path in read-only directory.  
**Input:** `--output /root/report` (user has no write permission)  
**Expected Behavior:** `PermissionError` on `open()` → uncaught → CLI crashes with traceback.  
**Actual (v1.0):** ✗ Not handled — exception propagates.  
**Mitigation:** Document in Troubleshooting; add permission check in v1.1.

## Summary

| ID | Severity | Status | v1.1 Target |
|---|---|---|---|
| E01 | P1 | Handled | — |
| E02 | P2 | Handled | Add pagination for >50 limit |
| E03 | P2 | Handled | Add URL validation |
| E04 | P3 | Handled | Add newline sanitization in MD |
| E05 | P1 | Handled | Add configurable timeout |
| E06 | P3 | Acceptable | Add file locking |
| E07 | P2 | Handled | — |
| E08 | P2 | Open | Add permission check + helpful error |
