# ai-readiness-diagnostic Architecture Summary

**Product:** ai-readiness-diagnostic  
**Repo:** `vladarchitectservicenow-oss/ai-readiness-diagnostic`  
**Scope:** `x_ai_readiness_diagnostic`  
**Release Target:** Australia (Washington DC+)  
**Author:** Vladimir Kapustin  
**Date:** 2026-06-01

## 1. Product Purpose

ai-readiness-diagnostic is a ServiceNow scoped application that scans any ServiceNow instance to determine its readiness for AI features — including Now Assist skills, Generative AI Controller, AI Agent Studio, and BYOK provider configurations. It produces a structured diagnostic report (MD/JSON/CSV) identifying gaps, missing plugins, unconfigured providers, and ACL/role deficiencies that block AI feature enablement.

## 2. Architecture Layers

### Layer 1 — Data Collection (REST + GlideRecord)
- Connects to target ServiceNow instance via REST API (Basic Auth / OAuth)
- Scans `sn_now_assist_config`, `sn_generative_ai_cfg_provider`, `sys_plugins`, `sys_user_role`, `sys_scope_privilege`, `sys_app`
- Runs GlideRecord queries across tables in read-only mode
- Respects `sysparm_limit` and pagination for large instances

### Layer 2 — Analysis Engine (JS Script Include)
- `AiReadinessScanner` — main scan orchestrator
- `PluginCheck` — verifies required plugins are installed and active (com.glide.hub.ai, com.snc.generative_ai)
- `ProviderCheck` — validates BYOK provider configurations (Azure OpenAI, Bedrock, Vertex AI)
- `RoleCheck` — checks `sn_now_assist.admin`, `sn_now_assist.user` role assignments
- `AclCheck` — verifies cross-scope access grants for AI tables
- `ScoreEngine` — computes 0-100 readiness score from weighted sub-checks

### Layer 3 — Report Generation
- Produces multi-format output: Markdown, JSON, CSV
- Delta mode: compares current scan to previous baseline, reports regressions
- Risk-tiered findings: CRITICAL (blockers), HIGH (missing components), MEDIUM (partial config), LOW (recommendations), INFO (observations)

### Layer 4 — Integration Surface
- REST endpoint: `POST /api/x_ai_readiness_diagnostic/scan` — trigger scan programmatically
- REST endpoint: `GET /api/x_ai_readiness_diagnostic/report/{scan_id}` — retrieve report
- CI/CD compatible — JSON output parsable by GitHub Actions, Jenkins, Azure DevOps

## 3. Data Model

| Table | Purpose |
|-------|---------|
| `x_ai_readiness_diagnostic_config` | Scan configuration (target instance, credentials, check tiers) |
| `x_ai_readiness_diagnostic_scan` | Individual scan records with status, score, timestamp |
| `x_ai_readiness_diagnostic_finding` | Per-check findings with severity, description, remediation |
| `x_ai_readiness_diagnostic_baseline` | Stored baseline for delta comparisons |

## 4. Security Model

- All target instance credentials encrypted at rest via GlideEncrypter
- Read-only operations only — scanner never modifies target instance data
- Access controlled via `x_ai_readiness_diagnostic.scanner` and `.admin` roles
- No PII collected — scan results contain only plugin names, role counts, config status
- Audit trail: all scan executions logged to `sys_log`

## 5. Deployment

- Via ServiceNow Studio XML import (`sys_app.xml`)
- Requires Global scope installation with cross-scope privileges granted
- Minimum version: Washington DC (for Now Assist API availability)
- PDI-compatible with Australia release

## 6. Dependencies

- **Required Plugins:** None — scanner runs independently (it checks FOR plugins on target, does not require them locally)
- **External:** `requests` Python module (for CLI runner)
- **ServiceNow APIs:** Table API, Stats API, REST Message API

## 7. Component Diagram

```
┌─────────────────────────────────────────────────┐
│                  ai-readiness-diagnostic          │
│                                                   │
│  ┌──────────┐   ┌──────────┐   ┌──────────────┐ │
│  │  Scanner  │──▶│  Engine  │──▶│ Report Gen   │ │
│  │ (REST)   │   │ (JS SI)  │   │ (MD/JSON/CSV)│ │
│  └──────────┘   └──────────┘   └──────────────┘ │
│       │               │                  │        │
│       ▼               ▼                  ▼        │
│  ┌──────────────────────────────────────────┐    │
│  │         Target Instance (read-only)       │    │
│  │  plugins │ roles │ providers │ ACLs      │    │
│  └──────────────────────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

## 8. Key Design Decisions

1. **Read-only scanning** — scanner never modifies target. Eliminates risk of accidental changes.
2. **Weighted scoring** — Plugin availability (30%), Provider config (25%), Role assignment (20%), ACL grants (15%), version check (10%).
3. **Delta mode** — stores baseline after each scan, enabling regression detection between instances or over time.
4. **Credentials encryption** — target instance passwords never stored in plaintext in scan records.
5. **Multi-instance capable** — single installation can scan multiple target instances, each with its own config record.
