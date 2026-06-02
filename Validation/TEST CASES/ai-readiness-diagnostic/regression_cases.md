# ai-readiness-diagnostic — Regression Cases

**Date:** 2026-06-01  
**Author:** Vladimir Kapustin

Regression cases verify that new code does not break existing functionality. Run on every release.

## REG-01: Score Calculation Consistency
**Purpose:** Verify scoring algorithm produces identical results for identical input across versions  
**Input:** Fixed scan result set (cached baseline)  
**Expected:** Score matches known value ± 0.1 tolerance  
**Verification:** Compare `score` field against golden baseline of 87.5  
**Fail if:** Score deviates by > 1.0 or status category changes

## REG-02: Report Format Stability
**Purpose:** JSON report schema must not break downstream consumers  
**Input:** Any completed scan  
**Expected:** JSON report contains all documented fields: scan_id, timestamp, score, status, findings[], metadata{}  
**Verification:** Parse JSON, assert all required keys present  
**Fail if:** Missing required field, changed field type (e.g., score changed from number to string)

## REG-03: Plugin Name Matching (Case Sensitivity)
**Purpose:** Plugin detection must be case-insensitive to handle PDI vs production naming variations  
**Input:** Plugin name "com.glide.hub.AI" (mixed case) vs stored "com.glide.hub.ai"  
**Expected:** Both match the same check  
**Verification:** Run plugin check with mixed-case input, verify it is detected as active  
**Fail if:** Mixed-case plugin name not detected

## REG-04: Provider Type Enum Stability
**Purpose:** Adding new provider types (e.g., watsonx in Australia) must not break existing detection  
**Input:** Scan target with Azure OpenAI only (no new providers)  
**Expected:** Azure OpenAI detected, new provider added to "not configured" list with INFO severity  
**Verification:** Provider findings contain Azure OpenAI as PASS, new provider as INFO  
**Fail if:** New provider type causes error, Azure OpenAI not detected

## REG-05: Cross-Scope Permission Inheritance
**Purpose:** Scanner must detect missing cross-scope privileges even when admin role is present  
**Input:** Target where admin role exists but explicit cross-scope privilege is absent for scanner's scope  
**Expected:** ACL check reports missing privilege despite admin role presence  
**Verification:** Run ACL check with admin role available but no scope privilege, verify finding generated  
**Fail if:** Scanner incorrectly reports ACL as PASS due to admin role bypass

## REG-06: Baseline Schema Version Compatibility
**Purpose:** Delta comparison must detect schema version mismatch and warn, not crash  
**Input:** Old baseline with schema v1, new scan with schema v2  
**Expected:** Delta report warns "schema mismatch", still produces comparison results  
**Verification:** Force schema version stamp mismatch, verify warning present, comparison still runs  
**Fail if:** Crash, empty delta, or missing schema version warning

## REG-07: Concurrent Access to Config Records
**Purpose:** Updating config while scan is running must not corrupt scan data  
**Input:** Running scan, simultaneous config update via REST  
**Expected:** Scan uses config snapshot from start time, update does not affect running scan  
**Verification:** Start scan, update config mid-scan, verify scan results use original config  
**Fail if:** Running scan picks up mid-scan config change

## REG-08: Large Finding Count (100+ Findings)
**Purpose:** Report generation must handle large finding arrays without truncation or OOM  
**Input:** Simulated scan with 150 findings across all categories  
**Expected:** All 150 findings present in report, no truncation, response size < 5MB  
**Verification:** Count findings in JSON output, verify = 150, verify response HTTP 200  
**Fail if:** Truncated findings, HTTP 500, or response > 10MB

## REG-09: REST Endpoint Auth Requirements
**Purpose:** All REST endpoints must enforce authentication  
**Input:** Unauthenticated requests to scan and report endpoints  
**Expected:** HTTP 401 Unauthorized for all endpoints  
**Verification:** curl without auth header, verify 401 response  
**Fail if:** Any endpoint returns 200 without authentication

## REG-10: GlideEncrypter Round-Trip
**Purpose:** Encrypted credentials must decrypt correctly after instance restart  
**Input:** Stored encrypted password for target instance  
**Expected:** After simulated restart (re-read from DB), decrypted password matches original  
**Verification:** Store credential → read back → decrypt → compare  
**Fail if:** Decrypted value differs from original
