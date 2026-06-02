# ai-readiness-diagnostic — Test Suite SOP

**Date:** 2026-06-01  
**Author:** Vladimir Kapustin  
**Test Environment:** ServiceNow PDI (dev362840) + Python Mock CI

## Pre-Test Setup

1. Verify PDI is active: navigate to `https://dev362840.service-now.com`
2. Install application via Studio XML import
3. Grant cross-scope read privileges to `x_ai_readiness_diagnostic` scope for target tables
4. Configure at least one target instance record (self-scan: use dev362840)
5. Verify Python test environment: `pip install requests pytest`

## Test Scenarios

### SC-01: Full Scan — All Plugins Active, All Providers Configured (PASS)
**Precondition:** Target instance has Now Assist, Generative AI, AI Agent Studio plugins active. Azure OpenAI provider configured. Admin roles assigned.  
**Steps:**
1. Configure target as a fully-ready instance
2. Run scan: `POST /api/x_ai_readiness_diagnostic/scan`
3. Retrieve report: `GET /api/x_ai_readiness_diagnostic/report/{scan_id}?format=json`
**Expected:** Score ≥ 90, status = "FULLY READY", 0 CRITICAL/HIGH findings  
**Pass Criteria:** Score ≥ 90, all checks return PASS status

### SC-02: Full Scan — No AI Plugins Installed (CRITICAL)
**Precondition:** Target instance missing all AI plugins (`com.glide.hub.ai`, `com.snc.generative_ai`)  
**Steps:**
1. Configure target with no AI plugins
2. Run scan
3. Retrieve report
**Expected:** Score < 50, status = "NOT READY", findings include "Plugin missing" for each required plugin  
**Pass Criteria:** Score < 50, at least 2 CRITICAL findings for missing plugins

### SC-03: Partial — Plugins Active, No Providers Configured (MEDIUM)
**Precondition:** Plugins active but `sn_generative_ai_cfg_provider` has 0 records  
**Steps:**
1. Configure target with active plugins but no provider config
2. Run scan
3. Retrieve report
**Expected:** Score 50-74, status = "PARTIALLY READY", HIGH finding for missing providers  
**Pass Criteria:** Score in 50-74 range, finding severity = HIGH for provider check

### SC-04: Edge — Pre-Australia Instance (TABLE_MISSING)
**Precondition:** Target instance pre-Australia — `sn_generative_ai_cfg_provider` table does not exist  
**Steps:**
1. Configure target as pre-Australia (or simulate table absence)
2. Run scan
3. Retrieve report
**Expected:** Provider check returns NOT_CONFIGURED with severity INFO (not CRITICAL), score adjusts for version  
**Pass Criteria:** No CRITICAL finding for missing table, provider status = NOT_CONFIGURED, severity = INFO

### SC-05: Roles — No Users with Now Assist Roles (HIGH)
**Precondition:** Plugins active, providers configured, but no users have `sn_now_assist.admin` or `sn_now_assist.user` role  
**Steps:**
1. Configure target with 0 role assignments for sn_now_assist.*
2. Run scan
3. Retrieve report
**Expected:** Role check returns HIGH finding, score penalty applied  
**Pass Criteria:** Finding for role check severity = HIGH, score below 75

### SC-06: ACL — Missing Cross-Scope Privileges (HIGH)
**Precondition:** Scanner scope does not have read access to AI tables on target  
**Steps:**
1. Remove cross-scope read privilege for `sn_now_assist_config` from target
2. Run scan
3. Retrieve report
**Expected:** ACL check returns HIGH finding, specific privilege grant documented as missing  
**Pass Criteria:** Finding for ACL check includes table name and missing privilege type (read)

### SC-07: Delta — Regression Detection
**Precondition:** Baseline stored from SC-01 (fully ready)  
**Steps:**
1. Store baseline from a "ready" scan
2. Remove a plugin on target (simulate regression)
3. Run new scan
4. Request delta: `GET /api/x_ai_readiness_diagnostic/delta/{new_scan_id}`
**Expected:** Delta report shows "new finding: Plugin missing" compared to baseline, score decreased  
**Pass Criteria:** Delta report contains at least 1 regression finding, score lower than baseline

### SC-08: Multi-Format — JSON Output
**Steps:**
1. Run scan
2. Retrieve report as JSON
3. Validate JSON structure: `{"scan_id": "...", "score": N, "status": "...", "findings": [...]}`
**Expected:** Valid JSON, all required fields present, findings array non-empty  
**Pass Criteria:** JSON parse succeeds, scan_id is string, score is number 0-100, findings is array

### SC-09: Multi-Format — CSV Output
**Steps:**
1. Run scan
2. Retrieve report as CSV
3. Validate CSV structure: header row + data rows
**Expected:** CSV with columns: finding_id, category, severity, description, remediation  
**Pass Criteria:** CSV parse succeeds, at least 1 data row, all headers present

### SC-10: Error Handling — Invalid Credentials (401)
**Precondition:** Target instance URL is valid but credentials are wrong  
**Steps:**
1. Configure target with wrong password
2. Run scan
3. Retrieve report
**Expected:** Scan fails with clear error: "Authentication failed (HTTP 401)"  
**Pass Criteria:** Finding or error message contains "401" or "Authentication failed", no crash

### SC-11: Error Handling — Network Timeout
**Precondition:** Target instance is unreachable (wrong URL or network down)  
**Steps:**
1. Configure target with unreachable URL
2. Run scan with default timeout
3. Retrieve report
**Expected:** Scan fails with clear error: "Connection timeout" or "Host unreachable"  
**Pass Criteria:** Finding or error message contains "timeout" or "unreachable", no crash, scan marked as FAILED

### SC-12: Concurrent Scan Prevention
**Precondition:** Scan already running for target  
**Steps:**
1. Start scan for target (background)
2. Immediately start another scan for same target
**Expected:** Second scan rejected: "Scan already in progress for this target"  
**Pass Criteria:** Second scan returns HTTP 409 or error with "in progress" message

### SC-13: Large Instance — Chunked Scanning
**Precondition:** Target has 50K+ records in scanned tables  
**Steps:**
1. Configure chunk size = 500
2. Run scan
3. Verify pagination logs: each chunk logged separately
**Expected:** Scan completes without timeout, all chunks processed, total records = sum of chunks  
**Pass Criteria:** All records counted, no timeout, chunk count > 1 in audit log

### SC-14: Empty Instance — Zero Records
**Precondition:** Fresh instance with 0 plugins, 0 roles, 0 providers  
**Steps:**
1. Configure target as empty instance
2. Run scan
3. Retrieve report
**Expected:** Score = 0, all checks return FAIL or NOT_CONFIGURED, no crash  
**Pass Criteria:** Score = 0, scanner completes without error, findings generated for all categories

## Test Data Requirements

| Scenario | Target State | Data Setup |
|----------|-------------|------------|
| SC-01 | Fully ready | All plugins active, Azure OpenAI configured, admin roles assigned |
| SC-02 | No plugins | Empty instance or simulate plugin list |
| SC-04 | Pre-Australia | Mock table absence for sn_generative_ai_cfg_provider |
| SC-05 | No roles | Remove all sn_now_assist.* role assignments |
| SC-07 | Baseline comparison | Run SC-01 first, store baseline |

## Execution Order

1. SC-01 (baseline) → SC-07 (delta)
2. SC-02, SC-03, SC-04 (negative cases) 
3. SC-05, SC-06 (partial failures)
4. SC-08, SC-09 (format validation)
5. SC-10, SC-11 (error handling)
6. SC-12, SC-13, SC-14 (advanced)

## Pass/Fail Criteria Summary

- **PASS:** All expected findings present with correct severity, score in expected range, no crashes
- **FAIL:** Missing expected finding, wrong severity, score outside range, scanner crash
- **BLOCKED:** PDI hibernated or unavailable (document, defer, test via mock CI)
