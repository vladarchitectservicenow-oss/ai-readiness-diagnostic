# ai-readiness-diagnostic — Edge Cases

**Date:** 2026-06-01  
**Author:** Vladimir Kapustin

## EC-01: Target Instance Has 0 Users
**Condition:** `sys_user` table empty on target instance  
**Expected Behavior:** Role check reports 0 users with AI roles, finding severity = HIGH, no crash  
**Risk:** Null division or empty-set iteration errors  
**Test:** Mock `sys_user_has_role` returning 0 rows

## EC-02: Target Instance Has 0 Applications (sys_app Empty)
**Condition:** No scoped applications installed on target  
**Expected Behavior:** Application scan returns empty list, not an error  
**Risk:** Null pointer on empty GlideRecord  
**Test:** Mock `sys_app` returning 0 rows

## EC-03: Plugin Partially Installed (Plugin Record Exists but State is NULL)
**Condition:** `sys_plugins` record exists but `active` field is NULL  
**Expected Behavior:** Plugin reported as UNKNOWN, severity INFO, score penalty 5 (not full -30)  
**Risk:** Crash on NULL comparison  
**Test:** Mock plugin record with active=NULL

## EC-04: Provider Configured with Empty Required Field
**Condition:** Azure OpenAI provider record has empty `api_key` (required field)  
**Expected Behavior:** Provider check reports WARNING: "Provider configured but required field 'api_key' is empty"  
**Risk:** False PASS on incomplete configuration  
**Test:** Mock provider with empty api_key

## EC-05: Multiple Providers of Same Type
**Condition:** Two Azure OpenAI provider records exist  
**Expected Behavior:** Both evaluated independently, aggregate status = worst of the two  
**Risk:** Only first record evaluated, second silently skipped  
**Test:** Mock 2 Azure OpenAI providers, one valid, one invalid

## EC-06: Instance Version String Unparseable
**Condition:** Target returns version string "CustomBuild-v12" instead of standard "Washington"  
**Expected Behavior:** Version check returns UNKNOWN, severity INFO, no score penalty  
**Risk:** Crash on version parsing  
**Test:** Mock version endpoint returning non-standard string

## EC-07: REST Response Truncated Mid-JSON
**Condition:** Target REST API returns truncated JSON (network interruption)  
**Expected Behavior:** Scanner detects JSON parse error, retries once, reports "Connection interrupted" if retry fails  
**Risk:** Silent empty report or partial data treated as complete  
**Test:** Simulate truncated response with incomplete JSON

## EC-08: All Checks Return INFO (No Significant Findings)
**Condition:** Target instance is perfectly configured — all plugins active, providers configured, roles assigned, ACLs granted  
**Expected Behavior:** Score = 100, status = "FULLY READY", findings array contains only INFO-level items  
**Risk:** Score calculation fails when all weights sum to 100 without penalty  
**Test:** Mock all checks returning PASS

## EC-09: Concurrent Scan from Different Scopes
**Condition:** Two different scoped apps try to scan the same target simultaneously  
**Expected Behavior:** Second scan rejected with "Scan already in progress" regardless of originating scope  
**Risk:** Scope-based locking instead of target-based (allows concurrent scans from different scopes)  
**Test:** Start scan from scope A, attempt scan from scope B

## EC-10: Credential Contains Special Characters
**Condition:** Target password contains `"`, `\`, `%`, `@`, `#` characters  
**Expected Behavior:** Password properly URL-encoded for REST transport, stored/retrieved correctly via GlideEncrypter  
**Risk:** REST auth fails due to unencoded special chars, or encryption truncates special chars  
**Test:** Use password `P@ss"w0rd#with\%special` — verify round-trip

## EC-11: Very Large Instance — 1M+ Records
**Condition:** Target has 1,000,000+ records across scanned tables  
**Expected Behavior:** Chunked scanning handles all records, progress logged per chunk, no OOM  
**Risk:** Memory exhaustion, timeout after default 120s  
**Test:** Increase timeout to 600s, chunk size to 1000, verify all chunks processed

## EC-12: Plugin Name Collision
**Condition:** Two plugins with similar names: `com.snc.generative_ai` and `com.snc.generative_ai_beta`  
**Expected Behavior:** Exact match only — `generative_ai_beta` does NOT satisfy `generative_ai` check  
**Risk:** Substring match causes false PASS for incomplete configuration  
**Test:** Mock both plugins, verify beta plugin does not trigger PASS for required plugin check
