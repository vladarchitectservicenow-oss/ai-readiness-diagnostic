# ai-readiness-diagnostic Dependency Report

**Date:** 2026-06-01  
**Author:** Vladimir Kapustin

## Internal Dependencies (ServiceNow Platform)

| Dependency | Type | Required Version | Status | Notes |
|-----------|------|-----------------|--------|-------|
| Now Assist Plugin (com.glide.hub.ai) | Platform Plugin | Washington DC+ | CHECKED | Target must have this installed; scanner detects absence |
| Generative AI Plugin (com.snc.generative_ai) | Platform Plugin | Washington DC+ | CHECKED | Required for BYOK provider configs |
| AI Agent Studio (com.snc.ai_agent_studio) | Platform Plugin | Australia+ | CHECKED | Optional — scanner reports if available |
| GlideRecord API | Platform API | All versions | REQUIRED | For table queries on target instance |
| GlideEncrypter | Platform API | All versions | REQUIRED | For credential encryption in config records |
| Table API (REST) | REST API | All versions | REQUIRED | For remote instance scanning |
| Stats API | REST API | All versions | OPTIONAL | For instance metadata collection |
| sys_log | Platform Table | All versions | REQUIRED | For audit trail logging |

## External Dependencies (CLI Runner)

| Dependency | Type | Version | Purpose |
|-----------|------|---------|---------|
| Python | Runtime | 3.9+ | CLI runner host |
| requests | Python Package | 2.28+ | HTTP calls to target REST API |
| urllib3 | Python Package | 1.26+ | TLS/SSL transport (transitive via requests) |

## Cross-Scope Access Requirements

When scanning a target instance's scoped applications for AI readiness, the scanner needs read access to:

| Table | Scope | Purpose |
|-------|-------|---------|
| sn_now_assist_config | Global | Now Assist skill configuration |
| sn_generative_ai_cfg_provider | Global | BYOK provider configuration |
| sys_plugins | Global | Plugin activation status |
| sys_user_role | Global | Role assignments |
| sys_user_has_role | Global | User-to-role mapping |
| sys_scope_privilege | Global | Cross-scope access grants |
| sys_app | Global | Installed application list |
| sys_store_app | Global | Application store metadata |

## Plugin Dependency Resolution

The scanner does NOT require these plugins on its own instance. It checks for their presence on the target instance and reports:

- **MISSING** — Plugin not installed → CRITICAL finding (score hit: -30)
- **INACTIVE** — Plugin installed but disabled → HIGH finding (score hit: -15)
- **ACTIVE** — Plugin installed and active → PASS
- **NOT_CONFIGURED** — Plugin active but providers/roles missing → MEDIUM finding (score hit: -10)

## Update Compatibility

| Release | Now Assist API | AI Agent Studio | Notes |
|---------|---------------|-----------------|-------|
| Washington DC | Available (v1) | Not available | Minimum supported |
| Vancouver | Available (v1) | Not available | Compatible |
| Yokohama | Available (v2) | Preview | Full support |
| Zurich | Available (v2) | GA | Full AI readiness |
| Australia | Available (v2) | GA + new skills | Target release |

## Risk of Dependency Breakage

- **Low:** GlideRecord, GlideEncrypter, sys_log — core platform APIs, stable across releases
- **Medium:** sn_now_assist_config table schema — may change between releases, scanner uses dynamic field detection
- **Low-Medium:** Generative AI provider tables — Australia introduces new provider types (Bedrock, Vertex AI); scanner handles unknown provider types gracefully via enum-based detection
- **Low:** REST APIs — Table API and Stats API are version-stable

## Mitigation

1. Dynamic table schema detection — scanner reads field list at runtime, no hardcoded column names for volatile tables
2. Graceful degradation — if a plugin table is missing (expected on pre-Australia), scanner reports NOT_CONFIGURED/INFO instead of error
3. Version-aware scoring — adjusts weight thresholds based on detected instance version
