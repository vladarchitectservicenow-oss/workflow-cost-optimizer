# Test Suite SOP — Workflow Cost Optimizer (WCO)

## Purpose
This document defines the standard operating procedure for validating Workflow Cost Optimizer. All tests must pass before any release.

## Test Environment
- ServiceNow instance: Utah+ (Development PDI)
- Scope: `x_snc_wco`
- Pre-populated catalog items: ≥10 (mix of portal, email, API-originated requests)
- Pre-populated incidents: ≥50 across 5 categories
- Required roles: `x_snc_wco_admin`

---

## Test Scenarios

### T01: Profile Catalog Items — Basic Classification
**Type:** Functional / Positive  
**Precondition:** 5+ `sc_cat_item` records exist with `sc_req_item` history from last 6 months  
**Steps:**
1. Navigate to `x_snc_wco_profile` table
2. Execute `new x_snc_wco.WorkflowProfiler().profileAll()`
3. Query `x_snc_wco_profile` for records where `source_table = 'sc_cat_item'`  
**Expected:**
- ≥1 profile record created per catalog item
- `channel_affinity` is one of: `PORTAL`, `SLACK`, `EMAIL`, `MIXED`
- `complexity` is `SIMPLE` or `MEDIUM`
- `monthly_volume` > 0
- `data_sensitivity` is `LOW`, `MEDIUM`, or `HIGH`

### T02: Profile Incident Categories
**Type:** Functional / Positive  
**Precondition:** `sys_choice` has incident categories with incident records from last 6 months  
**Steps:**
1. Execute `profileAll()`
2. Query profiles where `source_table = 'incident'`  
**Expected:**
- Profile records created for each active incident category
- `name` format: "Incident — {category_label}"
- `complexity = 'MEDIUM'`
- `avg_resolution_min = 30`

### T03: Channel Affinity — Majority PORTAL
**Type:** Functional / Edge  
**Precondition:** Catalog item with 80%+ requests via `service_portal` source  
**Steps:**
1. Profile the item
2. Check `channel_affinity`  
**Expected:** `channel_affinity = 'PORTAL'`

### T04: Channel Affinity — Unknown Source Falls to MIXED
**Type:** Functional / Negative  
**Precondition:** Catalog item with requests from unrecognized source values (empty or custom)  
**Steps:**
1. Profile the item
2. Check `channel_affinity`  
**Expected:** `channel_affinity = 'MIXED'` (not `null`, not error)

### T05: CostCalculator — Single Workflow Single Platform
**Type:** Functional / Positive  
**Precondition:** `x_snc_wco_pricing` has one platform with `fixed_monthly_cost=1000`, `cost_per_transaction=2.50`  
**Steps:**
1. Create profile with `monthly_volume=200`
2. Call `new x_snc_wco.CostCalculator().calculateForWorkflow(profileId)`  
**Expected:**
- `monthly_cost = 1500` (1000 × 200/200 + 200 × 2.50)
- `annual_cost = 18000`
- `cost_per_ticket = 7.50`
- `recommended.platform` matches the single platform

### T06: CostCalculator — Multiple Platforms Comparison
**Type:** Functional / Positive  
**Precondition:** 3 pricing records: NOW_ASSIST ($5K fixed, $4/txn), MOVEWORKS ($3K fixed, $2/txn), STANDALONE ($1K fixed, $0.50/txn)  
**Steps:**
1. Profile with `monthly_volume=500`
2. Call `calculateForWorkflow(profileId)`  
**Expected:**
- Results sorted by `monthly_cost` ascending
- STANDALONE cheapest, MOVEWORKS second, NOW_ASSIST most expensive
- Each result has `platform`, `monthly_cost`, `annual_cost`, `cost_per_ticket`

### T07: RoutingEngine — Compliance Hard Constraint
**Type:** Functional / Positive  
**Precondition:** Profile with `data_sensitivity = 'HIGH'`, Platform A has `compliance_certs = '["GDPR","HIPAA"]'`, Platform B has `compliance_certs = '[]'`
**Steps:**
1. Call `generateOptimalRouting()`  
**Expected:** Platform B is excluded from candidates; routing recommends Platform A (or no routing if Platform A doesn't exist)

### T08: RoutingEngine — Budget Constraint Greedy Removal
**Type:** Functional / Positive  
**Precondition:** 5 profiles with monthly costs [500, 400, 300, 200, 100], budget = 600  
**Steps:**
1. Call `generateOptimalRouting(600)`  
**Expected:**
- Total `total_monthly_cost` ≤ 600
- Workflows removed are highest cost-per-volume (500, 400 removed if total > 600)
- `coverage_pct` = 60% (3 of 5 routed)

### T09: ROI Projection — Savings Positive
**Type:** Functional / Positive  
**Precondition:** 3 profiles, NOW_ASSIST pricing costs $5000/mo total, optimized routing saves $1500/mo  
**Steps:**
1. Call `roiProjection()`  
**Expected:**
- `baseline_annual > optimized_annual`
- `annual_savings > 0`
- `savings_pct > 0`

### T10: REST API — GET /profiles Returns JSON
**Type:** Integration / Positive  
**Precondition:** 3 profiles exist  
**Steps:**
1. `GET /api/x_snc_wco/v1/profiles`  
**Expected:**
- HTTP 200
- Response body: `{"profiles": [{"id":"...","name":"...","volume":N,"affinity":"..."}]}`
- Array length = 3

### T11: REST API — POST /optimize With Budget
**Type:** Integration / Positive  
**Precondition:** Profiles + pricing exist  
**Steps:**
1. `POST /api/x_snc_wco/v1/optimize` with body `{"budget": 50000}`  
**Expected:**
- HTTP 200
- Response includes `routing_map` array, `total_monthly_cost`, `annual_cost`, `coverage_pct`
- `annual_cost ≤ budget`

### T12: Empty Instance — No Data
**Type:** Functional / Negative  
**Precondition:** Zero catalog items, zero incident categories  
**Steps:**
1. Execute `profileAll()`  
**Expected:** Returns `{profiled: 0}`, no profiles created, no error thrown

### T13: Monthly Cost Scan Scheduled Job
**Type:** Functional / Positive  
**Precondition:** Scheduled Job `monthly_cost_scan.js` is active  
**Steps:**
1. Trigger job via `sys_trigger`
2. Monitor `x_snc_wco_scan_run` table  
**Expected:**
- New scan run record created
- `status = 'COMPLETED'`
- `profiles_scanned ≥ 0`

### T14: Pricing Freshness — Stale Data Flag
**Type:** Functional / Negative  
**Precondition:** `x_snc_wco_pricing` records with `sys_updated_on > 90 days ago`  
**Steps:**
1. Run pricing freshness alert job  
**Expected:** Alert generated, notifying admin of stale pricing data

### T15: Concurrent Profile — Duplicate Prevention
**Type:** Functional / Negative  
**Precondition:** Profile already exists for `sc_cat_item` with `sys_id = X`  
**Steps:**
1. Run `profileAll()` twice consecutively  
**Expected:** No duplicate `x_snc_wco_profile` records for same `source_table + source_sys_id` pair

---

## Test Execution Log Format

```
[PASS/FAIL] TXX — Scenario Name — Actual Behavior
Timestamp: YYYY-MM-DD HH:MM:SS
Instance: devXXXXX.service-now.com
Executor: x_snc_wco_admin
```

## Pass/Fail Threshold

- **10/10 PASS** — Minimum for release
- **≥13/15 PASS** — Production-ready (T03, T04, T07, T08, T10 are non-negotiable)

## Negative Test Cases

| ID | Scenario | Expected Behavior |
|----|----------|-------------------|
| N01 | Call `calculateForWorkflow` with non-existent profile ID | Returns `{error: "Profile not found"}` — no uncaught exception |
| N02 | Call `generateOptimalRouting` with budget=0 | Returns routing with `total_monthly_cost = 0` and `coverage_pct = 0` |
| N03 | `x_snc_wco_pricing` table empty | `calculateForWorkflow` returns empty platforms array, routing returns empty map |
| N04 | Malformed `compliance_certs` field (not valid JSON) | `_passesCompliance` treats as empty `[]` — no crash |
| N05 | `sc_req_item` has null `request_source` on all requests | Channel affinity falls to `MIXED` — no null classification |
