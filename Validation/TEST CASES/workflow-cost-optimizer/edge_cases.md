# Edge Cases — Workflow Cost Optimizer (WCO)

## Purpose
Document data boundary conditions, unexpected inputs, and platform-specific edge cases that could cause runtime failures or data corruption.

---

## Edge Cases

### E01: Zero Total Volume — Division by Zero
**Location:** `CostCalculator._getTotalVolume()`  
**Condition:** `x_snc_wco_profile` table is empty  
**Risk:** Division by zero in fixed allocation calculation  
**Current Behavior:** Defaults to `1` via `Math.max(totalVol, 1)`  
**Expected:** Fixed allocation = 0 when no profiles exist  
**Test:** Delete all profiles, call `calculateForWorkflow()` — must not throw

### E02: Single Profile — Fixed Cost Allocated Entirely
**Location:** `CostCalculator.calculateForWorkflow()`  
**Condition:** Exactly 1 profile in table, `monthly_volume > 0`  
**Risk:** Fixed cost allocation = 100% of platform fixed cost (correct behavior, but should be verified)  
**Expected:** `fixedAllocation = platform.fixed_monthly_cost`  
**Test:** One profile with volume 100, platform fixed cost 5000 → `monthly_cost = 5000 + (100 × cost_per_transaction)`

### E03: Null `request_source` on All Requests
**Location:** `WorkflowProfiler._inferChannel()`  
**Condition:** Every `sc_req_item` for a catalog item has `request_source = null` or empty  
**Risk:** `max` key is `"unknown"` → `channel_affinity = 'MIXED'`  
**Expected:** Returns `'MIXED'` — no crash, no null classification  
**Test:** Catalog item with 100 requests all with null source → profile has `channel_affinity = 'MIXED'`

### E04: 10,000+ Catalog Items — GlideRecord Row Limit
**Location:** `WorkflowProfiler._profileCatalogItems()`  
**Condition:** Instance has >10,000 active catalog items  
**Risk:** Profiler times out or exceeds GlideRecord batch memory  
**Current Mitigation:** `setLimit(200)` on `sc_cat_item` query  
**Expected:** Only 200 items profiled per run; scheduled job handles remainder  
**Test:** Instance with 5,000 items; verify profiler completes <3 minutes

### E05: Malformed JSON in compliance_certs
**Location:** `RoutingEngine._passesCompliance()`  
**Condition:** `x_snc_wco_pricing.compliance_certs` contains `"GDPR"` (not an array — invalid JSON format)  
**Risk:** `JSON.parse("GDPR")` throws `SyntaxError` → routing engine crashes  
**Current Code:** `JSON.parse(platform.compliance_certs || "[]")` — no try/catch  
**Expected:** Should catch parse error, treat as empty `[]`, log warning  
**Test:** Insert pricing record with `compliance_certs = 'GDPR'`, run `generateOptimalRouting()` — must not crash

### E06: Budget Constraint Below Any Single Workflow Cost
**Location:** `RoutingEngine.generateOptimalRouting()`  
**Condition:** Budget < cheapest workflow monthly cost  
**Risk:** Greedy removal empties entire routing array → `coverage_pct = 0`  
**Expected:** Returns empty routing with `total_monthly_cost = 0` and `coverage_pct = 0`  
**Test:** Budget=1 with min workflow cost >1 → routing_map=[]

### E07: Very Large monthly_volume (Integer Overflow)
**Location:** `CostCalculator`, `RoutingEngine`  
**Condition:** Profile has `monthly_volume = 2147483647` (max 32-bit signed int)  
**Risk:** `parseInt()` overflow in GlideRecord read; arithmetic overflow in cost calc  
**Expected:** JavaScript Number (64-bit float) handles this; verify no `NaN` in results  
**Test:** Profile with max int volume → cost calculation produces finite numbers

### E08: Concurrent Profile Execution — Race Condition
**Location:** `WorkflowProfiler.profileAll()`  
**Condition:** Two users trigger profileAll() simultaneously  
**Risk:** Duplicate profile records (no unique constraint on `source_table + source_sys_id`)  
**Expected:** Duplicate records allowed but detectable; scheduled job deduplication in v1.1  
**Test:** Run two profileAll() via REST API simultaneously; check for duplicates

### E09: UTF-8/Non-ASCII Workflow Names
**Location:** `WorkflowProfiler._classifyAndSave()`  
**Condition:** Catalog item name contains Chinese, Arabic, or emoji characters  
**Risk:** GlideRecord `insert()` fails silently or corrupts name  
**Expected:** UTF-8 preserved correctly in `x_snc_wco_profile.name`  
**Test:** Create catalog item "パスワードリセット 🔒", profile → name stored correctly

### E10: Negative monthly_volume (Data Corruption)
**Location:** `CostCalculator.calculateForWorkflow()`  
**Condition:** Manual DB edit sets `monthly_volume = -50`  
**Risk:** Negative costs, negative cost_per_ticket  
**Expected:** `Math.max(volume, 0)` guard prevents negative; cost should be 0 or positive  
**Test:** Profile with volume=-50 → `monthly_cost ≥ 0` and `cost_per_ticket ≥ 0`

### E11: All Platforms Incompatible (Compliance)
**Location:** `RoutingEngine.generateOptimalRouting()`  
**Condition:** Profile with `data_sensitivity = 'HIGH'` and zero platforms with compliance certs  
**Risk:** No routing for high-sensitivity workflow → silently excluded  
**Expected:** Workflow excluded from routing map, logged as "UNROUTABLE"  
**Test:** HIGH sensitivity profile, no GDPR/HIPAA cert on any platform → workflow not in routing map

### E12: REST API Rate Limiting / Concurrent Requests
**Location:** REST endpoints  
**Condition:** 50 concurrent GET /profiles requests  
**Risk:** GlideRecord connection pool exhaustion, 503 errors  
**Expected:** All requests complete within 5 seconds, no 500 errors  
**Test:** 50 concurrent requests → all return 200 or 429 (rate limited, not 500)
