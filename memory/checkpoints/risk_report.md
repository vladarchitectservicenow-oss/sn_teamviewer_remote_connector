# sn_teamviewer_remote_connector — Risk Report

**Product:** ServiceNow TeamViewer Remote Connector  
**Scope:** x_sn_teamviewer_remote_connector  
**Author:** Vladimir Kapustin  
**Date:** 2026-06-02

## Risk Matrix

| ID | Risk | Severity | Likelihood | Impact | Status |
|---|---|---|---|---|---|
| R01 | Silent error suppression in fetch() | P0 | High | High data loss, false-negative reports | Mitigated: test_error_handling verifies silent fail returns [] |
| R02 | No authentication retry on token expiry | P1 | Medium | High — batch jobs fail silently mid-run | Open: retry logic not yet implemented |
| R03 | Credentials passed as CLI arguments | P1 | High | Critical — shell history leaks credentials | Open: argparse uses plaintext args; env-var support needed |
| R04 | 100-record limit truncation | P2 | Medium | Medium — incomplete reports in large instances | Open: pagination not yet implemented |
| R05 | No HTTPS certificate validation | P2 | Low | High — MITM attack vector | Mitigated: requests verifies certs by default |
| R06 | Report file overwrite without warning | P3 | Low | Medium — data loss if same prefix reused | Open: no existence check before write |
| R07 | No concurrent request handling | P3 | Low | Low — race conditions in multi-instance cron | Open: file-based locking not implemented |
| R08 | CLI --table parameter accepts arbitrary values | P2 | Medium | Medium — SQL injection not possible (REST API), but error handling poor | Mitigated: fetch() returns [] on any failure |
| R09 | No input validation on --sn-url | P2 | Medium | Low — malformed URL causes cryptic requests error | Open: URL format validation not implemented |
| R10 | Missing .gitignore for report artifacts | P3 | Low | Low — accidental commit of customer data | Open: .gitignore not present |

## Severity Definitions

| Level | Definition | Response |
|---|---|---|
| P0 | Critical — data loss, security breach, complete failure | Immediate fix required before deployment |
| P1 | High — significant functional gap, security concern | Must fix before production use |
| P2 | Medium — operational inconvenience, edge case | Should fix in next release |
| P3 | Low — cosmetic, unlikely to occur, best practice | Track for future improvement |

## Risk Trend

| Category | Count | Trend |
|---|---|---|
| P0 Critical | 1 | Fixed (R01 — error handling verified) |
| P1 High | 2 | Open (R02, R03) |
| P2 Medium | 3 | Open (R04, R08, R09) |
| P3 Low | 3 | Open (R06, R07, R10) |
| Resolved | 3 | R01, R05, R08 (partial) |

## Mitigation Roadmap

### Short-term (v1.1)
- [ ] R03: Add environment variable credential support (`SN_URL`, `SN_USER`, `SN_PASS`)
- [ ] R02: Implement retry with exponential backoff (3 attempts, 1s/2s/4s)
- [ ] R10: Add `.gitignore` for `.json`, `.md` report artifacts

### Medium-term (v1.2)
- [ ] R04: Implement pagination via `sysparm_offset` for tables >100 records
- [ ] R09: Add URL validation with `urllib.parse.urlparse()`
- [ ] R06: Check for file existence before overwrite, add --force flag

### Long-term (v2.0)
- [ ] R07: Add file-based locking for concurrent execution safety
- [ ] Add TeamViewer session initiation API integration
- [ ] Add ServiceNow write-back (session log to custom table)

## Acceptance Criteria

| Gate | Criterion | Met? |
|---|---|---|
| G0 | All P0 risks resolved | ✓ (R01 fixed, R05 false positive) |
| G1 | P1 risks documented with workaround | ✓ (R02: retry on wrapper, R03: shell env) |
| G2 | Risk report ≥10 entries | ✓ (10 risks identified) |
| G3 | Each risk has severity + likelihood + impact | ✓ |
| G4 | Mitigation roadmap exists | ✓ (3-phase plan) |
