# sn_teamviewer_remote_connector — Execution Plan

**Product:** ServiceNow TeamViewer Remote Connector  
**Scope:** x_sn_teamviewer_remote_connector  
**Author:** Vladimir Kapustin  
**Date:** 2026-06-02  
**Version:** 1.0.0 → 2.0.0 Roadmap

## Phase 1: Foundation (v1.0.0 — COMPLETED)

### 1.1 Core Engine
- [x] `engine.py`: HTTP fetch, data processing, report generation
- [x] `cli.py`: Argument parsing, command-line entry point
- [x] Basic Auth authentication against ServiceNow REST API
- [x] JSON + Markdown dual-format report output

### 1.2 Testing
- [x] 7 unit tests with mocked dependencies
- [x] Test coverage: fetch, process, report (JSON/MD), empty data, error handling, CLI invocation
- [x] 100% pass rate verified

### 1.3 Documentation
- [x] README.md ≥2000 words with Mermaid diagrams
- [x] Architecture summary, dependency report, risk report, execution plan
- [x] Validation suite: SOP (12 scenarios), regression cases, edge cases, checklist
- [x] AGPL-3.0 LICENSE (full text)

### 1.4 Quality Gates
- [x] G0: test_suite_SOP.md ≥10 scenarios
- [x] G1: All tests pass (7/7)
- [x] G2: README ≥2000 words with Mermaid + ROI
- [x] G3: AGPL-3.0 copyright header on all src/ files
- [x] G5: No hardcoded credentials in source
- [x] G6: `.gitignore` created
- [x] G7: README license matches LICENSE file
- [x] G8: No duplicate README sections

## Phase 2: TeamViewer Integration (v1.1 — PLANNED)

### 2.1 TeamViewer API Client
- [ ] Implement TeamViewer token-based authentication
- [ ] Session initiation: POST /api/v1/sessions
- [ ] Session status polling: GET /api/v1/sessions/{id}
- [ ] Connection metadata extraction (session ID, start/end timestamps)

### 2.2 Enhanced CLI
- [ ] `--teamviewer-token` parameter (env-var: `TV_API_TOKEN`)
- [ ] `--initiate-session` flag: auto-create TeamViewer session from incident context
- [ ] `--write-back` flag: POST session metadata to ServiceNow custom table

### 2.3 Error Handling Improvements
- [ ] Retry logic for transient network errors (3 attempts, exponential backoff)
- [ ] Environment variable credential support (`SN_URL`, `SN_USER`, `SN_PASS`)
- [ ] URL format validation before API calls

## Phase 3: ServiceNow Scoped App (v1.2 — PLANNED)

### 3.1 ServiceNow Module
- [ ] Create scoped application `x_sn_teamviewer_remote_connector`
- [ ] Custom tables: `x_sn_teamviewer_remote_connector_config`, `x_sn_teamviewer_remote_connector_log`
- [ ] Script Includes: `TeamViewerConnector`, `TeamViewerSessionManager`
- [ ] UI Action: "Start TeamViewer Session" on incident form

### 3.2 REST Endpoints
- [ ] POST `/api/x_sn_teamviewer_remote_connector/sessions`: Initiate session
- [ ] GET `/api/x_sn_teamviewer_remote_connector/sessions/{id}`: Status check
- [ ] GET `/api/x_sn_teamviewer_remote_connector/audit`: Session history

### 3.3 PDI Smoke Test
- [ ] Deploy to PDI
- [ ] Background Script validation
- [ ] UI Action functional test

## Phase 4: Production Hardening (v2.0 — PLANNED)

### 4.1 Scalability
- [ ] Pagination for tables >100 records
- [ ] Concurrent request handling with file locking
- [ ] Connection pooling for repeated API calls

### 4.2 Observability
- [ ] Structured logging (JSON format)
- [ ] Metrics: request latency, error rate, session count
- [ ] Health check endpoint

### 4.3 Security
- [ ] OAuth 2.0 support (in addition to Basic Auth)
- [ ] API key rotation support
- [ ] Audit trail completeness verification

### 4.4 CI/CD
- [ ] GitHub Actions workflow for test execution
- [ ] Release automation (tag → changelog → GitHub Release)
- [ ] Docker container for headless/mid-server deployment

## Execution Tracking

| Phase | Status | Start | Target | Progress |
|---|---|---|---|---|
| Phase 1: Foundation | ✓ Complete | 2026-05 | 2026-05 | 100% |
| Phase 2: TeamViewer API | Planned | 2026-06 | 2026-07 | 0% |
| Phase 3: Scoped App | Planned | 2026-07 | 2026-08 | 0% |
| Phase 4: Hardening | Planned | 2026-08 | 2026-09 | 0% |

## Dependencies Between Phases

```
Phase 1 (Core Engine)
    ↓
Phase 2 (TeamViewer API) ── requires engine.fetch() for context
    ↓
Phase 3 (Scoped App) ── requires TeamViewer session management from Phase 2
    ↓
Phase 4 (Hardening) ── requires all three prior phases
```

## Risk to Schedule

| Risk | Impact | Phase Affected |
|---|---|---|
| TeamViewer API rate limits | Medium | Phase 2 |
| ServiceNow scoped app limit (2 SI max) | Medium | Phase 3 |
| Zurich/Australia API changes | Low | Phase 1-4 |
| Customer PII requirements | Low | Phase 4 |
