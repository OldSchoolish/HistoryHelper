# HistoryHelper — Roadmap

This document tracks the current state of HistoryHelper and what's planned. No timeline is attached — this is a portfolio reconstruction project worked on alongside full-time AppSec engineering work.

---

## Current State

The core pipeline is fully functional end-to-end — from Jira harvest through rule validation. Current coverage is scoped to XSS and SQL Injection findings processed in year-by-year chunks. Metrics is the only incomplete component.

---

## Complete

- [x] Jira INFOSEC findings harvester
- [x] Git history analyzer (agentic diff retrieval per finding)
- [x] AI-driven dynamic Semgrep rule generator
- [x] Org ruleset comparator (gap / overlap / conflict analysis)
- [x] Attack type classifier
- [x] Validator (historical corpus + default regression set)

---

## In Progress

- [ ] Metrics dashboard (FP rate delta, coverage delta, redundancy reduction)

---

## Planned

- [ ] Expand attack type coverage beyond XSS and SQLi
- [ ] Automated re-validation loop (regenerate and retest rules that fail validation)
- [ ] Rule confidence scoring (weight rules by how many findings support them)
- [ ] Historical coverage report (what percentage of prior findings would current ruleset have caught)
- [ ] Semgrep registry integration (compare generated rules against public Semgrep community rules)
- [ ] CI/CD integration hooks (run HistoryHelper refinement as part of ruleset update pipeline)
- [ ] Multi-repo support (aggregate findings and history across multiple codebases)

---

## Known Constraints

- Requires Jira access with INFOSEC-labeled findings and Git history on the target codebase — not a standalone tool
- Year-by-year processing is a deliberate scope decision, not a technical limitation; full history can be processed incrementally
- Validator corpora require curation — historical corpus must be built from real findings, not synthetically generated
- Dynamic rule generation quality is dependent on the underlying LLM; rules should always be reviewed before production use

---

## Out of Scope (for now)

- Support for SAST tools other than Semgrep
- Real-time finding ingestion (designed for batch historical processing)
- Automatic rule deployment without human review

---

*See [ARCHITECTURE.md](./ARCHITECTURE.md) for the full technical design.*
