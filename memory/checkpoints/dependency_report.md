# sn_teamviewer_remote_connector — Dependency Report

**Product:** ServiceNow TeamViewer Remote Connector  
**Scope:** x_sn_teamviewer_remote_connector  
**Author:** Vladimir Kapustin  
**Date:** 2026-06-02

## Internal Dependencies (ServiceNow Platform)

| Dependency | Type | Required | Notes |
|---|---|---|---|
| REST API Access (table_api) | Plugin | Yes | com.glide.rest — all table read operations |
| Authentication (Basic Auth) | Platform | Yes | ServiceNow username/password or OAuth token |
| sys_log table | Table | Optional | Audit logging target (not yet implemented) |
| System Properties | Config | Optional | Could store default SN URL, timeout values |

## External Dependencies (Python Runtime)

| Package | Version | Required | Purpose |
|---|---|---|---|
| requests | ≥2.28.0 | Yes | HTTP client for ServiceNow REST API calls |
| argparse | stdlib | Yes | CLI argument parsing |
| json | stdlib | Yes | JSON serialization for reports |
| os | stdlib | Yes | File path operations |
| sys | stdlib | Yes | Python path manipulation |
| tempfile | stdlib | Test | Temporary directory creation in test suite |
| unittest.mock | stdlib | Test | Mocking requests.get in unit tests |
| pytest | ≥7.0 | Test | Test framework |
| subprocess | stdlib | Test | CLI invocation test |

## Integration Points

| External System | Protocol | Authentication | Direction |
|---|---|---|---|
| ServiceNow Instance | HTTPS REST (JSON) | Basic Auth | Read (GET /api/now/table/*) |
| ServiceNow Instance | HTTPS REST (JSON) | Basic Auth | Write (POST /api/now/table/*) *planned* |
| TeamViewer Remote API | HTTPS REST (JSON) | API Token | Read/Write *planned* |
| Local Filesystem | OS I/O | Filesystem ACLs | Write (report files) |

## ServiceNow Specific Dependencies

### Required Plugins
- **REST API Plugin (com.glide.rest):** Enables REST endpoints on the instance. Activated by default on all modern instances.

### Table Access Requirements
| Table | Access Level | Purpose |
|---|---|---|
| incident | Read | Fetch incident records for context |
| sc_request | Read | Fetch service catalog request records |
| x_sn_teamviewer_remote_connector_config | Read/Write | Configuration store *planned* |
| x_sn_teamviewer_remote_connector_log | Write | Session audit log *planned* |

### Role Requirements
| Role | Purpose | Scope |
|---|---|---|
| snc_platform_rest_api_access | REST API usage | Global |
| itil | Incident read access | Global |
| x_sn_teamviewer_remote_connector.admin | Admin operations | Scoped |

## Test Dependencies

| Component | Version | Notes |
|---|---|---|
| Python | ≥3.10 | Required for match/case, type hints |
| pytest | ≥7.0 | Test runner |
| pytest-asyncio | — | Not used (sync-only tests) |
| Coverage (coverage.py) | ≥7.0 | Optional test coverage reporting |

## Version Compatibility Matrix

| ServiceNow Release | API Version | Connector Status |
|---|---|---|
| Utah | v2 | Supported |
| Vancouver | v2 | Supported |
| Washington DC | v2 | Supported |
| Australia | v2 | Target, to be validated |
| Zurich | v2 | Target, to be validated |

## Risk Assessment

| Risk | Severity | Mitigation |
|---|---|---|
| `requests` library not available | P2 | Could migrate to `urllib` (stdlib) |
| SN instance version API changes | P1 | Pin API version, add version check |
| Network connectivity loss | P1 | Retry logic (not yet implemented) |
| Large table response (>100 records) | P3 | Pagination support *planned* |
