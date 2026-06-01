# Dependency Report — Workflow Cost Optimizer (WCO)

## Platform Dependencies

### ServiceNow Platform Version

| Dependency | Minimum | Recommended | Notes |
|-----------|---------|-------------|-------|
| ServiceNow Instance | Utah | Australia | Uses GlideAggregate, GlideRecord, Class.create() — all stable across releases |
| Scope Type | Scoped Application | Scoped Application | `x_snc_wco` namespace |
| Required Plugin: REST API | com.glide.rest | Any version | For REST endpoints |
| Required Plugin: Scheduled Jobs | com.snc.scheduled_jobs | Any version | For monthly scan + pricing freshness alert |

### GlideRecord Tables Read (Platform Tables)

| Table | Access Type | Purpose | Risk |
|-------|------------|---------|------|
| `sc_cat_item` | Read | Catalog item profiling | Low — stable table since Fuji |
| `sc_req_item` | Read | Channel affinity calculation (6-month lookback) | Medium — high row count, needs query limits |
| `incident` | Read | Incident category volume calculation | Medium — large table, 6-month filter |
| `sys_choice` | Read | Incident category labels | Low — small configuration table |
| `sys_glide_object` | None | Not accessed | N/A |

### GlideRecord Tables Written (Application Tables)

| Table | Scope | Purpose |
|-------|-------|---------|
| `x_snc_wco_profile` | Scoped | Workflow profiling results |
| `x_snc_wco_pricing` | Scoped | AI platform pricing models (config) |
| `x_snc_wco_routing` | Scoped | Optimal routing decisions |
| `x_snc_wco_scan_run` | Scoped | Execution audit trail |
| `x_snc_wco_findings` | Scoped | Detailed scan findings |

### JavaScript API Dependencies

| API | Used In | Purpose | Deprecation Risk |
|-----|---------|---------|-----------------|
| `Class.create()` | All Script Includes | OOP pattern for Script Includes | None — core platform API |
| `GlideRecord` | All components | Table CRUD operations | None — core platform API |
| `GlideAggregate` | CostCalculator, RoutingEngine | SUM aggregation for total volume | None — core platform API |
| `gs.monthsAgo()` | WorkflowProfiler | 6-month lookback calculation | None — core GlideSystem method |
| `JSON.parse()` | RoutingEngine | Parse compliance_certs from pricing table | None — standard JS |
| `Array.prototype.reduce()` | RoutingEngine | Cost summation | None — standard JS (ES5) |
| `Array.prototype.sort()` | CostCalculator, RoutingEngine | Platform ranking by cost | None — standard JS |
| `Array.prototype.map()` | RoutingEngine | Alternative platform extraction | None — standard JS |
| `Array.prototype.find()` | RoutingEngine | Find Now Assist platform object | None — standard JS (ES6, available in ServiceNow Rhino) |

### No External Integrations (Optional)

| Integration | Status | Notes |
|------------|--------|-------|
| ServiceNow AI Agent Studio | Optional | For AI-assisted remediation hints (v1.1+) |
| External CI/CD | Optional | JSON export via REST for pipeline consumption |
| Email (SMTP) | Optional | Report distribution to stakeholders |

## Role & ACL Dependencies

| Role | Access | Tables |
|------|--------|--------|
| `x_snc_wco_admin` | Full CRUD | All `x_snc_wco_*` tables |
| `x_snc_wco_viewer` | Read-only | `x_snc_wco_profile`, `x_snc_wco_routing`, `x_snc_wco_pricing` |
| `admin` | Full access (inherits) | All |

## Compatibility Matrix

| Release | Compatibility | Notes |
|---------|--------------|-------|
| Utah | ✅ Full | Minimum supported |
| Vancouver | ✅ Full | |
| Washington DC | ✅ Full | |
| Zurich | ✅ Full | Current target |
| Australia | ✅ Full | Future target |
| Pre-Utah | ❌ Not tested | Uses GlideAggregate patterns from Utah+ |
