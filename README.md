# HistoryHelper 📜

> Your SAST findings have a history. Use it.

---

## The Problem

Alert fatigue is one of the most persistent challenges in SAST programs. Rules that generate too many false positives get ignored. Rules that miss real vulnerabilities create false confidence. Most rule tuning is reactive — someone complains about noise, someone tweaks a rule, repeat.

HistoryHelper takes a different approach: instead of tuning rules based on gut feel or recent complaints, it mines years of historical security findings to understand what vulnerabilities actually existed in a codebase, how they were fixed, and what patterns they shared — then uses that data to generate and refine Semgrep rules that are grounded in real outcomes.

The goal is fewer false positives, better true positive coverage, and rules that reflect the actual risk profile of the codebase they're protecting.

---

## How It Works

HistoryHelper runs a history-informed rule refinement pipeline:

1. **Harvest** — pulls historical INFOSEC-labeled findings from Jira, extracting vulnerability descriptions and remediation context
2. **Analyze** — correlates findings with Git history to surface the actual code patterns involved and how they were addressed, processed year-by-year to manage volume
3. **Generate** — AI dynamically generates Semgrep rules based on each finding's context, reasoning through the vulnerability pattern fresh rather than following a static template
4. **Compare** — new rules are compared against the existing org Semgrep ruleset to identify overlap, gaps, and conflicts
5. **Classify** — findings and rules are categorized by attack type for focused refinement
6. **Validate** — generated rules are tested against a historical corpus of known-vulnerable code and a default regression set to confirm coverage without breaking existing detections

---

## Current Focus

HistoryHelper has been validated against two of the most prevalent web application vulnerability classes:

- **XSS (Cross-Site Scripting)** — inline script injection, DOM-based patterns, reflected and stored variants
- **SQL Injection** — parameterization failures, unsafe query construction patterns

Additional attack types are on the roadmap as the pipeline matures.

---

## Why Dynamic Rule Generation

An earlier approach used a static generator script to produce Semgrep rules from finding descriptions. HistoryHelper moved away from this in favor of AI-driven dynamic generation for a few reasons:

- **Novel patterns** — a static generator needs to be updated to handle new finding types; dynamic generation handles them naturally
- **Richer reasoning** — the AI reasons through both the Jira finding context and the associated code diff, producing rules grounded in the actual vulnerability rather than a template approximation
- **Model improvements flow through** — as the underlying model improves, rule quality improves without code changes

---

## Validation Approach

Generated rules are validated against two corpora:

- **Historical corpus** — a curated set of real vulnerable code patterns drawn from prior findings, confirming the rule catches what it should
- **Default regression set** — a baseline set of known patterns confirming new rules don't introduce regressions in existing coverage

---

## Project Status

| Component | Status |
|---|---|
| Jira findings harvester | ✅ Complete |
| Git history analyzer | ✅ Complete |
| AI rule generator | ✅ Complete |
| Org ruleset comparator | ✅ Complete |
| Classifier | ✅ Complete |
| Validator | ✅ Complete |
| Metrics dashboard | 🔨 In progress |
| Additional attack type coverage | 📋 Roadmap |

---

## Stack

- **Python** — pipeline orchestration, harvesting, classification
- **Semgrep** — rule format, validation engine
- **AI (LLM)** — dynamic rule generation and refinement reasoning
- **Git** — historical code pattern analysis

---

## Architecture

See [ARCHITECTURE.md](./ARCHITECTURE.md) for a detailed breakdown of the pipeline, AI integration design, and validation approach.

---

## Roadmap

See [ROADMAP.md](./ROADMAP.md) for planned features and coverage expansions.

---

## Background

HistoryHelper grew out of a real problem: a mature codebase with years of security findings, an existing Semgrep ruleset of unknown accuracy, and the question of whether the rules in place would actually catch the kinds of vulnerabilities that had historically made it into the code. The answer to that question required going back through the history — so that's exactly what HistoryHelper does.

---

*HistoryHelper is a portfolio reconstruction project. No proprietary code, data, or organizational specifics from prior employment are included.*
