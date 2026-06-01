# Validation Checklist — Workflow Cost Optimizer (WCO)

## Purpose
Pre-release validation checklist. Every item must be PASS before git push. Mark each item with [PASS] or [FAIL] with notes.

---

## Phase 1: Documentation Gates

| # | Check | Status | Notes |
|---|-------|--------|-------|
| 1.1 | `architecture_summary.md` exists and ≥40 lines | [ ] | |
| 1.2 | `architecture_summary.md` contains component table with I/O columns | [ ] | |
| 1.3 | `architecture_summary.md` contains Mermaid or ASCII data flow diagram | [ ] | |
| 1.4 | `architecture_summary.md` contains performance benchmarks | [ ] | |
| 1.5 | `dependency_report.md` exists and ≥30 lines | [ ] | |
| 1.6 | `dependency_report.md` lists all GlideRecord tables (read + write) | [ ] | |
| 1.7 | `dependency_report.md` lists all JS API dependencies with deprecation risk | [ ] | |
| 1.8 | `dependency_report.md` includes compatibility matrix (Utah→Australia) | [ ] | |
| 1.9 | `risk_report.md` exists and ≥10 risks with P0/P1/P2/P3 severity tags | [ ] | |
| 1.10 | `risk_report.md` includes risk matrix (impact × likelihood) | [ ] | |
| 1.11 | `risk_report.md` recommends remediation sequence | [ ] | |
| 1.12 | `execution_plan.md` exists and ≥30 lines | [ ] | |

## Phase 2: Validation Suite Gates

| # | Check | Status | Notes |
|---|-------|--------|-------|
| 2.1 | `test_suite_SOP.md` exists with ≥10 TXX scenarios | [ ] | |
| 2.2 | At least 3 TXX scenarios are negative/error cases | [ ] | |
| 2.3 | T01–T15 use table format with Precondition/Steps/Expected | [ ] | |
| 2.4 | `regression_cases.md` exists with ≥8 RXX scenarios | [ ] | |
| 2.5 | `edge_cases.md` exists with ≥10 edge cases | [ ] | |
| 2.6 | `edge_cases.md` covers division by zero (E01) | [ ] | |
| 2.7 | `edge_cases.md` covers malformed JSON (E05) | [ ] | |
| 2.8 | `edge_cases.md` covers concurrent execution (E08) | [ ] | |
| 2.9 | `validation_checklist.md` exists (this file) | [ ] | |

## Phase 3: README Quality Gates

| # | Check | Status | Notes |
|---|-------|--------|-------|
| 3.1 | README ≥2000 words (after deduplication) | [ ] | |
| 3.2 | No duplicate `## ` headings (`grep '^## ' README.md \| sort \| uniq -d` empty) | [ ] | |
| 3.3 | Mermaid architecture diagram present | [ ] | |
| 3.4 | ROI analysis table present with cost savings | [ ] | |
| 3.5 | Troubleshooting section with ≥5 symptom/cause/resolution rows | [ ] | |
| 3.6 | Installation instructions present | [ ] | |
| 3.7 | API Reference section present | [ ] | |
| 3.8 | Roadmap section present | [ ] | |
| 3.9 | License section matches LICENSE file type (MIT) | [ ] | |
| 3.10 | Copyright line visible: "Copyright (C) 2026 Vladimir Kapustin" | [ ] | |

## Phase 4: LICENSE & Source Code Gates

| # | Check | Status | Notes |
|---|-------|--------|-------|
| 4.1 | `LICENSE` file present at repo root | [ ] | |
| 4.2 | LICENSE contains "Copyright (C) 2026 Vladimir Kapustin" | [ ] | |
| 4.3 | LICENSE type is MIT (not AGPL) | [ ] | |
| 4.4 | README license section says "MIT License" (not "AGPL-3.0") | [ ] | |
| 4.5 | `src/script_includes/WorkflowProfiler.js` has copyright header | [ ] | |
| 4.6 | `src/script_includes/CostCalculator.js` has copyright header | [ ] | |
| 4.7 | `src/script_includes/RoutingEngine.js` has copyright header | [ ] | |
| 4.8 | No hardcoded credentials in source (G5) — all auth via env/REST headers | [ ] | |
| 4.9 | `.gitignore` exists and covers `__pycache__/`, `*.pyc`, `reports/` | [ ] | |

## Phase 5: Git & Deployment Gates

| # | Check | Status | Notes |
|---|-------|--------|-------|
| 5.1 | All files staged (`git add -A`) | [ ] | |
| 5.2 | Commit with conventional message format | [ ] | |
| 5.3 | `git push origin main` succeeds | [ ] | |
| 5.4 | GitHub API confirms new commit SHA on remote | [ ] | |
| 5.5 | `DONE.marker` file pushed to repo root | [ ] | |
| 5.6 | GitHub API confirms `DONE.marker` exists on remote | [ ] | |

---

## Validation Sign-off

```
Validator: ____________________
Date: _________________________
Overall Result: [ ] PASS / [ ] FAIL
Action: [ ] Proceed to release / [ ] Fix failures and re-validate
```

## Gate Counts

| Phase | Total Gates | Pass Required | Critical (non-negotiable) |
|-------|------------|---------------|---------------------------|
| Phase 1 | 12 | 12 | 1.1, 1.2, 1.9 |
| Phase 2 | 9 | 9 | 2.1, 2.2, 2.4 |
| Phase 3 | 10 | 10 | 3.1, 3.2, 3.9 |
| Phase 4 | 9 | 9 | 4.2, 4.3, 4.4 |
| Phase 5 | 6 | 6 | 5.3, 5.5 |
| **Total** | **46** | **46** | **10 critical** |
