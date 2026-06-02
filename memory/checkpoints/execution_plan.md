# ai-readiness-diagnostic Execution Plan

**Date:** 2026-06-01  
**Author:** Vladimir Kapustin  
**Target Release:** Australia (May 2026)

## Phase Overview

| Phase | Name | Duration | Status |
|-------|------|----------|--------|
| 1 | Documentation & Architecture | 2 days | ✅ COMPLETE (2026-06-01) |
| 2 | Validation Suite | 2 days | ✅ COMPLETE (2026-06-01) |
| 3 | Core Engine Development | 5 days | Planned |
| 4 | REST API & Integration | 3 days | Planned |
| 5 | CLI Runner | 2 days | Planned |
| 6 | Testing & QA | 3 days | Planned |
| 7 | PDI Smoke Test | 1 day | Planned |
| 8 | Release Packaging | 1 day | Planned |

**Total estimated:** 19 days  
**Release window:** Australia GA + 2 weeks

## Phase 3 — Core Engine Development

### Task 3.1: Base Scanner (Script Include)
- Implement `AiReadinessScanner` class
- Methods: `initialize(config)`, `runAllChecks()`, `getScore()`, `getFindings()`
- Auto-detect target instance version from stats API

### Task 3.2: Plugin Detection Module
- Query `sys_plugins` on target via REST
- Required plugin list: `com.glide.hub.ai`, `com.snc.generative_ai`, `com.snc.ai_agent_studio`
- Map status values (0=inactive, 1=active, null=not installed)

### Task 3.3: Provider Configuration Module
- Query `sn_generative_ai_cfg_provider` on target
- Detect provider types: Azure OpenAI, AWS Bedrock, GCP Vertex AI, watsonx
- Validate required fields per provider type
- Handle TABLE_MISSING gracefully (pre-Australia instances)

### Task 3.4: Role Assignment Module
- Query `sys_user_has_role` → filter for `sn_now_assist.*` roles
- Report role assignment counts and coverage
- Flag: 0 users with admin role → CRITICAL

### Task 3.5: ACL Verification Module
- Query `sys_scope_privilege` for cross-scope access to AI tables
- Verify read access to `sn_now_assist_config`, `sn_generative_ai_cfg_provider`
- Flag missing grants

### Task 3.6: Scoring Engine
- Weight configuration: PluginCheck(30), ProviderCheck(25), RoleCheck(20), AclCheck(15), VersionCheck(10)
- Compute weighted score 0-100
- Thresholds: <50 = NOT READY, 50-74 = PARTIALLY READY, 75-89 = READY WITH GAPS, 90+ = FULLY READY

## Phase 4 — REST API & Integration

### Task 4.1: Scan Trigger Endpoint
- POST `/api/x_ai_readiness_diagnostic/scan`
- Body: `{"target_instance": "dev123456", "format": "json"}`  
- Returns: `{"scan_id": "...", "status": "queued"}`

### Task 4.2: Report Retrieval Endpoint
- GET `/api/x_ai_readiness_diagnostic/report/{scan_id}`  
- Returns full scan results in requested format
- Supports `?format=json|md|csv`

### Task 4.3: Baseline Management
- POST `/api/x_ai_readiness_diagnostic/baseline/{scan_id}`  
- Stores current scan as baseline for future delta comparisons
- GET `/api/x_ai_readiness_diagnostic/delta/{scan_id}` — compares against stored baseline

## Phase 5 — CLI Runner

### Task 5.1: Python CLI
- `src/cli.py` — argparse-based CLI
- Flags: `--sn-url`, `--sn-user`, `--sn-pass`, `--output`, `--format`, `--chunk-size`, `--timeout`
- Output: writes report to filesystem, prints score summary to stdout
- Exit codes: 0=PASS, 1=CRITICAL findings, 2=connection error

### Task 5.2: CI/CD Integration
- JSON output mode for machine parsing
- Non-zero exit codes for CI/CD pipeline gating
- Example GitHub Actions workflow

## Phase 6 — Testing & QA

### Task 6.1: Unit Tests
- Mock GlideRecord for all check modules
- Test each check with: PASS, FAIL, MISSING_TABLE, PARTIAL scenarios
- Target: 90%+ code coverage

### Task 6.2: Integration Tests
- Test against real ServiceNow PDI (dev362840)
- Test against simulated instance (Python mock server)
- Test multi-instance scanning

### Task 6.3: Edge Case Testing
- Empty instance (0 plugins, 0 roles)
- Maximum instance (all plugins active, all roles assigned)
- Pre-Australia instance (missing provider table)
- Hibernated instance (HTTP 503)
- Rate-limited instance (HTTP 429)
- Invalid credentials (HTTP 401)
- Network timeout
- Large instance (100K+ records)

## Phase 7 — PDI Smoke Test

1. Install application to dev362840 via Studio XML import
2. Configure target as dev362840 (self-scan)
3. Run scan via Background Script: `new AiReadinessScanner().run()`
4. Verify report generation
5. Test REST endpoint via curl
6. Verify no data modification on target

## Phase 8 — Release Packaging

1. Export sys_app.xml from Studio
2. Create GitHub release with version tag
3. Update README with version badge
4. Publish to ServiceNow Share (optional)
5. Marketing materials generation

## Dependencies & Blockers

| Blocker | Status | Resolution |
|---------|--------|------------|
| Australia PDI availability | ✅ Resolved | dev362840 active |
| Now Assist API documentation | ✅ Resolved | docs.servicenow.com |
| Generative AI provider schema | ✅ Resolved | Australia release notes |
| Cross-scope privilege testing | ⚠️ Pending | Requires admin on target |

## Rollback Plan

If scanner produces false results:
1. Identify root cause via sys_log audit trail
2. Fix check module logic
3. Re-scan and validate delta against known-good baseline
4. Mark affected scans as "invalidated" in database
5. Notify users who consumed invalid reports (if tracked)
