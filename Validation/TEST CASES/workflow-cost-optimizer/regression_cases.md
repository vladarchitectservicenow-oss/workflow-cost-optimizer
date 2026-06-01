# Regression Cases — Workflow Cost Optimizer (WCO)

## Purpose
Regression cases verify that existing functionality is not broken by new changes. These cover the complete lifecycle: profiling → costing → routing → ROI → REST.

---

## Regression Scenarios

### R01: Profiler Output Stability Across Runs
**Category:** Profiling  
**Trigger:** Any change to `WorkflowProfiler` Script Include  
**Steps:**
1. Run `profileAll()` and capture `profiled` count + first 5 profile names
2. Make code change
3. Run `profileAll()` again  
**Expected:** Same count, same profile names (immutable source data assumed)

### R02: CostCalculator Arithmetic Consistency
**Category:** Costing  
**Trigger:** Any change to `CostCalculator` or `x_snc_wco_pricing` table schema  
**Steps:**
1. Load fixed pricing: `fixed_monthly_cost=2000`, `cost_per_transaction=5`
2. Create profile with `monthly_volume=100`
3. Calculate cost, record `monthly_cost`
4. Manually compute: 2000 × (100/100) + 100 × 5 = 2500  
**Expected:** `monthly_cost = 2500.00` — zero deviation from manual calculation

### R03: RoutingEngine Does Not Crash on Empty Profiles
**Category:** Robustness  
**Trigger:** Any change to `RoutingEngine._getAllProfiles()`  
**Steps:**
1. Delete all `x_snc_wco_profile` records
2. Call `generateOptimalRouting()`  
**Expected:** Returns `{routing_map: [], total_monthly_cost: 0, coverage_pct: 0}` — no exception

### R04: RoutingEngine Consistent Recommendation for Same Data
**Category:** Determinism  
**Trigger:** Any change to `RoutingEngine` algorithm or `_calcCost()`  
**Steps:**
1. Run `generateOptimalRouting()` with 3 profiles + 2 platforms, record routing map
2. Run again with identical data  
**Expected:** Same `recommended_platform` for each workflow across both runs

### R05: ROI Projection Returns Non-Negative Savings
**Category:** ROI  
**Trigger:** Any change to `roiProjection()` logic  
**Steps:**
1. Configure NOW_ASSIST as most expensive platform
2. Run `generateOptimalRouting()` then `roiProjection()`  
**Expected:** `annual_savings ≥ 0` (optimized cannot cost more than all-Now-Assist baseline)

### R06: REST API /profiles Returns Consistent Count
**Category:** Integration  
**Trigger:** Any change to REST endpoint or `_getAllProfiles()`  
**Steps:**
1. Create N profile records
2. `GET /api/x_snc_wco/v1/profiles`  
**Expected:** Response array length = N

### R07: Scheduled Job Creates Scan Run Record
**Category:** Scheduling  
**Trigger:** Any change to `monthly_cost_scan.js`  
**Steps:**
1. Trigger scheduled job
2. Query `x_snc_wco_scan_run` sorted by `sys_created_on DESC`  
**Expected:** Newest record has `start_time` within last 60 seconds

### R08: All Script Includes Load Without Syntax Errors
**Category:** Build  
**Trigger:** Any change to any `.js` file in `src/script_includes/`  
**Steps:**
1. Load all Script Includes in ServiceNow Studio
2. Verify no "Syntax Error" or "Reference Error" at load time  
**Expected:** All three files load clean

---

## Regression Execution Log

```
Run ID: REG-YYYYMMDD-NNN
Date:
Instance:
Trigger:
Results:
  R01: [PASS/FAIL]
  R02: [PASS/FAIL]
  R03: [PASS/FAIL]
  R04: [PASS/FAIL]
  R05: [PASS/FAIL]
  R06: [PASS/FAIL]
  R07: [PASS/FAIL]
  R08: [PASS/FAIL]
```

## Regression Threshold
- **8/8 PASS** — Required for any release
- Any FAIL must be root-caused and fixed before proceeding
