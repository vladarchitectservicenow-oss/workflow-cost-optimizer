# Execution Plan — Workflow Cost Optimizer (WCO)

## Current State Assessment

| Aspect | Status | Notes |
|--------|--------|-------|
| Source Code | COMPLETE | 3 Script Includes + 1 Scheduled Job + 1 REST API module |
| Architecture Docs | CREATED | architecture_summary.md, dependency_report.md, risk_report.md |
| Validation Suite | CREATING | SOP, regression, edge cases, checklist |
| README | PARTIAL | 2233 words but duplicate sections, mismatched license header (AGPL vs MIT in LICENSE) |
| LICENSE | PRESENT | Full MIT text, correct copyright |
| Git Push | NOT VERIFIED | DONE.marker absent |

## Execution Phases

### Phase 1: Documentation Foundation ✅

| Task | Status | Artifact |
|------|--------|----------|
| Architecture summary | ✅ Done | `memory/checkpoints/architecture_summary.md` |
| Dependency report | ✅ Done | `memory/checkpoints/dependency_report.md` |
| Risk report (15 risks) | ✅ Done | `memory/checkpoints/risk_report.md` |
| Execution plan | ✅ Done | `memory/checkpoints/execution_plan.md` |

### Phase 2: Validation Suite

| Task | Action | Artifact |
|------|--------|----------|
| Test Suite SOP | Write ≥10 scenarios with T01-TXX format, including negative cases | `Validation/TEST CASES/workflow-cost-optimizer/test_suite_SOP.md` |
| Regression Cases | Write ≥8 regression scenarios (R01-R08) covering profile, cost, routing, ROI, REST | `Validation/TEST CASES/workflow-cost-optimizer/regression_cases.md` |
| Edge Cases | Document data boundary conditions (empty table, null fields, 10K+ profiles, malformed JSON) | `Validation/TEST CASES/workflow-cost-optimizer/edge_cases.md` |
| Validation Checklist | Pre-release checklist with pass/fail gates | `Validation/TEST CASES/workflow-cost-optimizer/validation_checklist.md` |

### Phase 3: README Remediation

| Task | Action |
|------|--------|
| Deduplicate sections | Remove duplicate "Overview", "Architecture", "Features", "Installation", "Configuration", "ROI", "Troubleshooting", "Security", "API Reference", "Testing", "Roadmap", "License", "Support" blocks (appear twice — lines 8-89 and lines 90-268) |
| Add Mermaid diagram | Enhance architecture diagram with component-level detail |
| Expand ROI | Add multi-year ROI projection table |
| Add Troubleshooting | Expand to 8+ real scenarios with resolution steps |
| License header fix | Change "Licensed under GNU Affero General Public License v3.0" to "Licensed under MIT" (matching LICENSE file) |
| Word count verify | Target ≥2000 unique words (currently ~1100 unique after dedup) |

### Phase 4: LICENSE Verification

| Task | Action |
|------|--------|
| Copyright verification | Confirm `Copyright (C) 2026 Vladimir Kapustin` present |
| License type match | Confirm README says MIT (not AGPL) — matches LICENSE file |
| File header check | Verify all `src/*.js` files have copyright headers |

### Phase 5: Git Operations

| Task | Command |
|------|---------|
| Stage all changes | `git add -A` |
| Commit | `git commit -m "feat(validation): complete Phase 1+2 docs, dedup README, validation suite"` |
| Push | `git push origin main` |
| Verify | Check GitHub API for updated files |
| Marker | Create `DONE.marker` and push |

## Success Criteria

| Gate | Criterion | Target |
|------|-----------|--------|
| G1 | `test_suite_SOP.md` ≥10 scenarios with T01-TXX format | ≥10 TXX scenarios |
| G2 | `regression_cases.md` ≥8 cases with R01-R08 format | ≥8 RXX cases |
| G3 | `edge_cases.md` documented | ≥5 edge cases |
| G4 | `architecture_summary.md` ≥40 lines | ≥50 lines |
| G5 | `dependency_report.md` ≥30 lines | ≥40 lines |
| G6 | `risk_report.md` ≥10 risks with severity tags | ≥15 risks |
| G7 | README deduplicated (no repeated sections) | ≤18 `## ` headings |
| G8 | README ≥2000 words | ≥2000 words |
| G9 | README license matches LICENSE file | Both "MIT" |
| G10 | DONE.marker pushed | Present on GitHub |

## Timeline

| Phase | Estimated Effort | Dependency |
|-------|-----------------|------------|
| Phase 2 (Validation) | 30 min | Phase 1 complete |
| Phase 3 (README) | 20 min | Phase 2 complete |
| Phase 4 (LICENSE) | 5 min | Phase 3 complete |
| Phase 5 (Git) | 5 min | All prior phases complete |
| **Total** | **60 min** | |
