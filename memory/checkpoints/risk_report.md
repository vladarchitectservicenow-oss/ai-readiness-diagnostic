# ai-readiness-diagnostic Risk Report

**Date:** 2026-06-01  
**Author:** Vladimir Kapustin  
**Classification:** Internal — Security-Relevant

## Risk Register

### P0 — Critical (Must Fix Before Production Use)

| ID | Risk | Impact | Likelihood | Mitigation | Owner |
|----|------|--------|------------|------------|-------|
| R01 | Target instance credentials exposed in scan logs | PII/credential leak via sys_log | Medium | Credentials encrypted with GlideEncrypter before storage; scan logs reference scan_id only, never credentials | Dev |
| R02 | Scanner writes to target instance accidentally | Data corruption on customer instance | Low | All operations use read-only GET/query; no setValue()/insert()/update()/delete() calls | Dev |
| R03 | Cross-scope privilege escalation | Unauthorized access to scoped app data | Low | Scanner runs in its own scope; cross-scope reads require explicit privilege grants documented in installation guide | Dev |

### P1 — High

| ID | Risk | Impact | Likelihood | Mitigation | Owner |
|----|------|--------|------------|------------|-------|
| R04 | Rate limiting by target instance during large scans | Scan timeout, incomplete report | Medium | Configurable chunk size (default 500 records), pagination support, retry with exponential backoff | Dev |
| R05 | False negatives — scanner misses misconfigured provider due to table schema change | Customer believes instance is AI-ready when it is not | Medium | Dynamic field detection at runtime; version-aware checks; comprehensive test suite for each release | QA |
| R06 | Scan on hibernated PDI returns incorrect results | Wasted troubleshooting time | Medium | Pre-scan connectivity check; HTTP 503 detection with clear "instance may be hibernating" message | Dev |
| R07 | Plugin table missing on pre-Australia instances | Scanner crashes or reports false CRITICAL | Low | Graceful NOT_CONFIGURED/INFO fallback; table existence check before query | Dev |

### P2 — Medium

| ID | Risk | Impact | Likelihood | Mitigation | Owner |
|----|------|--------|------------|------------|-------|
| R08 | Score weighting becomes outdated for new releases | Readiness score misleads customers | Medium | Version-aware scoring weights; configurable via config record | Dev |
| R09 | Multiple concurrent scans on same target | Rate limiting, duplicate data | Low | Singleton scan per target; scan queue with "in_progress" status flag | Dev |
| R10 | Delta comparison fails when baseline schema changes | Inaccurate regression detection | Low | Schema version stamp in baseline; auto-invalidate on version mismatch | Dev |

### P3 — Low

| ID | Risk | Impact | Likelihood | Mitigation | Owner |
|----|------|--------|------------|------------|-------|
| R11 | CLI runner fails on Python < 3.9 | Cannot run from older CI environments | Low | Version check at startup; clear error message with upgrade instructions | Dev |
| R12 | Large instance (>100K records) causes memory pressure in CLI runner | CLI runner OOM | Low | Streaming JSON parser; `--chunk-size` flag; warn on instances >50K records | Dev |

## Residual Risk After Mitigation

After all P0 and P1 mitigations are applied:
- **Credential leak risk:** Acceptable (encrypted at rest, no plaintext in logs)
- **Data corruption risk:** Acceptable (read-only operations only)
- **False negative risk:** Tolerable (dynamic detection + comprehensive test suite)
- **Rate limit risk:** Tolerable (chunking + backoff)

## Security Review Checklist

- [x] No hardcoded credentials in source code
- [x] All REST calls use HTTPS only
- [x] Target credentials encrypted at rest (GlideEncrypter)
- [x] Read-only operations — no write/delete to target
- [x] Cross-scope privileges explicitly documented
- [x] Audit logging via sys_log for all scan operations
- [x] No PII collection — scan results contain only metadata
- [x] GDPR-compliant — configurable data retention policy for scan records

## Sign-off

| Role | Name | Date | Status |
|------|------|------|--------|
| Developer | Vladimir Kapustin | 2026-06-01 | Approved |
| Security Review | (Pending) | — | — |
| QA Lead | (Pending) | — | — |
