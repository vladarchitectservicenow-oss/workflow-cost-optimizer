# Risk Report — Workflow Cost Optimizer (WCO)

## Risk Assessment Summary

| Risk ID | Category | Severity | Description | Impact | Mitigation |
|---------|----------|----------|-------------|--------|------------|
| R01 | Performance | P1 | `sc_req_item` full table scan without limits on large instances (>1M rows) | Profiler timeout, instance load spike | `setLimit(200)` already applied on `sc_cat_item`; add row count pre-check with early exit if >100K rows |
| R02 | Data Accuracy | P0 | Channel affinity derived from `request_source` field on `sc_req_item` — may be null/unpopulated on older instances | Incorrect classification → suboptimal routing | Fallback to `sys_created_by` domain or `opened_by` department mapping |
| R03 | Platform Config | P1 | Pricing models manually configured — stale data leads to inaccurate cost comparisons | Routing recommendations may be $10K+ off | Weekly pricing freshness alerts; enforce 90-day max staleness |
| R04 | Security | P1 | REST API endpoints expose workflow volume data without authentication granularity | Information disclosure to unauthorized users | Require `x_snc_wco_viewer` role on all `/v1/*` endpoints |
| R05 | Migration | P2 | `GlideAggregate.getAggregate()` return type varies by platform version (string vs number on pre-Utah) | `parseInt()` may fail on string representation | Explicit `parseInt()` coercion already in place; add try/catch wrapper |
| R06 | Architecture | P0 | `RoutingEngine._saveRouting()` does not clear stale routing records before insert — leads to duplicate recommendations | Dashboard shows conflicting routing for same workflow | Add `deleteObsolete()` pre-step: delete all `x_snc_wco_routing` records before insert batch |
| R07 | User Experience | P2 | Hardcoded `MEDIUM` complexity for all incident categories — no differentiation between password reset (simple) and security incident (complex) | Routing may not prioritize correctly for high-stakes incidents | Extend `_inferComplexity()` to check incident priority, assignment group, and resolution SLA |
| R08 | Compliance | P1 | `_passesCompliance()` checks certs as JSON array but doesn't validate JSON.parse success | Malformed `compliance_certs` crashes routing engine | Wrap `JSON.parse()` in try/catch; fallback to `[]` on parse failure |
| R09 | Data Integrity | P2 | `_getTotalVolume()` defaults to 1 on empty table — causes division errors if single workflow | Fixed allocation skewed for single-workflow instances | Check row count >0 before aggregate; return 0 with guard in caller |
| R10 | Performance | P2 | `RoutingEngine.roiProjection()` does per-profile GlideRecord queries in a loop (N+1 problem) | Slow on >500 workflow profiles | Batch load all routing records into a hash map before iteration |
| R11 | Operational | P1 | No transaction rollback if scheduled job fails mid-execution | Partial profile writes → inconsistent state | Wrap scheduled job in GlideTransaction; add `x_snc_wco_scan_run` status tracking with FAILED state |
| R12 | Dependency | P3 | `Array.prototype.find()` (ES6) not available on pre-Utah Rhino engine | `roiProjection()` crashes | Replace with `for` loop + `break` pattern; add ES5 polyfill comment |
| R13 | Config | P1 | `gs.monthsAgo(6)` returns hardcoded 6-month window — not configurable | Cannot adjust analysis window for seasonal businesses | Expose `lookback_months` as System Property `x_snc_wco.lookback_months` |
| R14 | Error Handling | P0 | No catch blocks around `GlideRecord.get()` or `GlideAggregate` calls | Uncaught exceptions crash REST API with 500 | Add try/catch in all public methods (`profileAll`, `calculateForWorkflow`, `generateOptimalRouting`, `roiProjection`) |
| R15 | Documentation | P3 | REST API contract in SPEC.md references `x_snc_wco` but files use different namespace references | Integration confusion for external consumers | Standardize all docs and code to `x_snc_wco` |

## Risk Matrix

```
Impact
  ^
H │  R02  R06  R14
I │  R04  R08  R11
G │  R01  R03  R13
H │
  │
M │  R05  R09  R10
E │
D │
  │
L │  R07  R12  R15
O │
W │
  └──────────────────────────→ Likelihood
     Low      Medium      High
```

## Critical Path Risks (Must Fix Before Production)

| Priority | Risk IDs | Combined Impact |
|----------|----------|-----------------|
| P0 — Blockers | R02, R06, R14 | Data accuracy + duplicate routing + no error handling = unusable in production |
| P1 — High | R01, R03, R04, R08, R11, R13 | Performance degrades, stale data, auth gaps, compliance crashes, no rollback |
| P2 — Important | R05, R09, R10 | Edge case data failures, N+1 perf |
| P3 — Nice-to-have | R07, R12, R15 | UX polish, polyfill, doc consistency |

## Recommended Remediation Sequence

1. **Sprint 1:** R02 (channel fallback) → R06 (routing dedup) → R14 (error handling)
2. **Sprint 2:** R01 (row limits) → R03 (freshness alerts) → R04 (auth enforcement)
3. **Sprint 3:** R08 (compliance parse guard) → R11 (transaction rollback) → R13 (configurable lookback)
4. **Backlog:** R05 (type coercion), R09 (division guard), R10 (N+1 batch), R07 (complexity refinement), R12 (ES5 polyfill), R15 (doc consistency)
