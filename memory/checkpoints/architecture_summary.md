# Architecture Summary — Workflow Cost Optimizer (WCO)

## Product Identity

| Field | Value |
|-------|-------|
| Name | Workflow Cost Optimizer |
| Scope Prefix | `x_snc_wco` / `x_wco` |
| License | MIT |
| Repository | `vladarchitectservicenow-oss/workflow-cost-optimizer` |
| Author | Vladimir Kapustin |
| ServiceNow Target | Utah → Australia |

## Problem Domain

Enterprise organizations spend 30-40% of AI helpdesk budget on suboptimally routed workflows. Manual evaluation takes 4+ months and relies on biased vendor demos. WCO provides a data-driven engine that profiles every workflow by channel affinity, volume, complexity, and data sensitivity — then computes cost across AI platforms and generates an optimal routing map.

## Component Architecture

### Three-Tier Scoped Application

```
┌─────────────────────────────────────────────────────┐
│                   PRESENTATION                      │
│  Dashboard Widgets │ UI Actions │ REST API (v1)    │
└────────────────────────┬────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────┐
│                  BUSINESS LOGIC                     │
│  WorkflowProfiler  │  CostCalculator  │ RoutingEngine │
│  (channel+complexity) │ (cost-per-workflow) │ (CSAT optimization) │
└────────────────────────┬────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────┐
│                   DATA LAYER                        │
│  x_snc_wco_profile  │ x_snc_wco_pricing │ x_snc_wco_routing │
│  x_snc_wco_scan_run │ x_snc_wco_findings            │
└─────────────────────────────────────────────────────┘
```

### Core Script Includes

| Component | File | Responsibility | Input | Output |
|-----------|------|---------------|-------|--------|
| WorkflowProfiler | `src/script_includes/WorkflowProfiler.js` | Scans `sc_cat_item` + incident categories; classifies by channel affinity (PORTAL/SLACK/EMAIL/MIXED), complexity (SIMPLE/MEDIUM/COMPLEX), data sensitivity (LOW/MEDIUM/HIGH) | `sc_cat_item`, `sc_req_item`, `sys_choice`, `incident` | Profile records in `x_snc_wco_profile` |
| CostCalculator | `src/script_includes/CostCalculator.js` | Computes per-workflow cost across all configured AI platforms using pricing models from `x_snc_wco_pricing` | Profile record, pricing models | Monthly/annual cost, cost-per-ticket, platform comparison |
| RoutingEngine | `src/script_includes/RoutingEngine.js` | Constraint-satisfaction optimization: compliance (hard), latency (soft), budget (hard). Generates optimal routing map + ROI projection | All profiles, platforms, optional budget constraint | Routing map, total costs, coverage %, savings vs all-Now-Assist baseline |

### REST API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/x_snc_wco/v1/profiles` | GET | List all profiled workflows with channel/volume data |
| `/api/x_snc_wco/v1/compare?platforms=X,Y` | GET | Cost comparison across platforms for all workflows |
| `/api/x_snc_wco/v1/optimize` | POST | Generate optimal routing map (body: `{"budget": 80000}`) |

### Scheduled Jobs

| Job | Schedule | Purpose |
|-----|----------|---------|
| Monthly Cost Scan | 1st of month, 02:00 | Re-profiles all workflows, recalculates costs, regenerates routing |
| Pricing Freshness Alert | Weekly | Flags pricing models not updated in >90 days |

## Data Flow

```
sc_cat_item ──┐
sc_req_item ──┤
incident ─────┤──→ WorkflowProfiler ──→ x_snc_wco_profile
sys_choice ───┘                                 │
                                                ▼
x_snc_wco_pricing ──────→ CostCalculator ──→ cost comparison
                                                │
                                                ▼
                                         RoutingEngine ──→ x_snc_wco_routing
                                                │
                                   ┌────────────┴──────────┐
                                   ▼                       ▼
                            ROI Projection          REST API response
                            (baseline vs optimized)
```

## Performance Benchmarks

| Operation | Expected Performance | Constraint |
|-----------|---------------------|------------|
| Full profile (5K catalog items) | <3 minutes | GlideRecord batch, setLimit(200) per table |
| Single workflow cost calc | <100ms | In-memory arithmetic |
| Full routing optimization | <2 seconds | Greedy CSAT algorithm, O(n×m) where n=profiles, m=platforms |
| REST API responses | <500ms | JSON serialization of GlideRecord results |

## Tables and Relationships

| Table | Fields | Relationships |
|-------|--------|---------------|
| `x_snc_wco_profile` | source_table, source_sys_id, name, monthly_volume, channel_affinity, complexity, avg_resolution_min, data_sensitivity | → `x_snc_wco_routing.workflow_profile` |
| `x_snc_wco_pricing` | platform, fixed_monthly_cost, cost_per_transaction, compliance_certs, typical_latency_ms | Referenced by CostCalculator, RoutingEngine |
| `x_snc_wco_routing` | workflow_profile, recommended_platform, estimated_monthly_cost, confidence_score | ← `x_snc_wco_profile.sys_id` |
| `x_snc_wco_scan_run` | start_time, end_time, status, profiles_scanned | Tracks execution history |
| `x_snc_wco_findings` | scan_run, profile_sys_id, severity, finding_text | Detailed scan results |

## Security Model

- Read-only access to `sc_cat_item`, `sc_req_item`, `incident`, `sys_choice` — no modifications to platform tables
- Application tables (`x_snc_wco_*`) isolated in scoped app namespace
- All API calls HTTPS-only
- Credentials not stored — pricing is config data, not secrets
- Role: `x_snc_wco_admin` for configuration, `x_snc_wco_viewer` for read-only dashboard access
- No PII collection — only workflow metadata and cost aggregates
