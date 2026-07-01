# HistoryHelper — Architecture

## Design Philosophy

Most SAST rule tuning is reactive and opinion-driven. A rule generates noise, someone complains, the rule gets loosened. A finding slips through, the rule gets tightened. Over time the ruleset drifts away from the actual risk profile of the codebase.

HistoryHelper replaces opinion with evidence. The codebase has years of security history — real vulnerabilities, real fixes, real patterns. That history is the most accurate signal available for what the rules should catch. HistoryHelper extracts that signal and uses it to drive rule generation and refinement systematically.

The pipeline is **history-in, rules-out.** Every generated rule is traceable back to a real finding.

---

## High-Level Pipeline

```
┌─────────────────────────────────────────────────────────┐
│                  HistoryHelper Pipeline                    │
│                                                         │
│  [1. Harvest]  → Jira INFOSEC findings by year          │
│       ↓                                                 │
│  [2. Git Analysis] → Code diffs per finding             │
│       ↓                                                 │
│  [3. AI Rule Generation] → Semgrep rules per finding    │
│       ↓                                                 │
│  [4. Org Rule Comparison] → Gap / overlap analysis      │
│       ↓                                                 │
│  [5. Classifier] → Rules grouped by attack type         │
│       ↓                                                 │
│  [6. Validator] → Coverage confirmed, regressions clear │
│       ↓                                                 │
│  [7. Metrics]*  → FP reduction, coverage delta          │
│                                                         │
│  * In progress                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Component Breakdown

### 1. Harvester

The harvester queries Jira for all findings labeled INFOSEC and extracts the relevant context from each ticket.

**What it collects:**
- Vulnerability description
- Affected component or file reference
- Remediation notes or linked fix
- Ticket metadata (date, severity, status)

**Processing approach:**
Due to the volume of historical findings, the harvester processes findings in year-long chunks rather than attempting to ingest the full history at once. This keeps individual runs manageable and allows the pipeline to be run incrementally.

**Output:**
A structured dataset of historical findings, organized by year and attack type, ready for git correlation.

---

### 2. Git History Analyzer

For each finding surfaced by the harvester, the git history analyzer retrieves the associated code changes to understand what the vulnerable pattern actually looked like and how it was remediated.

**Agentic approach:**
Rather than being handed pre-formatted diffs, the AI orchestrates its own git commands to retrieve the relevant history. The likely flow:

1. Get the list of INFOSEC findings from the harvester output
2. For each finding, run git commands to locate the associated commit(s)
3. Pull the full diff for each commit — vulnerable state and fix together

This agentic retrieval means the AI is working with raw git history rather than a summarized or pre-processed version, which preserves fidelity to the actual code patterns involved.

**Output:**
Per-finding context packages: Jira ticket content + associated code diff, paired for rule generation.

---

### 3. AI Rule Generator

The rule generator is the core of HistoryHelper. For each finding context package, the AI dynamically generates a Semgrep rule designed to detect the vulnerability pattern.

**Why dynamic generation:**
An earlier approach used a static generator script. HistoryHelper moved to dynamic AI-driven generation because:

- Static generators require updates to handle novel finding types; dynamic generation handles them naturally
- The AI reasons through both the ticket context and the code diff together, producing rules grounded in the actual vulnerability rather than a template approximation
- Rule quality improves as the underlying model improves, without code changes

**What the AI reasons through:**
- What the vulnerability was (from the Jira context)
- What the vulnerable code pattern looked like (from the diff)
- What a Semgrep rule would need to match to catch this pattern
- Whether the pattern is general enough to be useful or too narrow to be worth a rule

**Output:**
A generated Semgrep rule per finding, in valid Semgrep YAML format, with reasoning notes attached.

---

### 4. Org Rule Comparator

Generated rules are compared against the existing cloned org Semgrep ruleset to understand the relationship between new and existing coverage.

**What it identifies:**
- **Overlap** — new rule catches what an existing rule already catches (potential redundancy)
- **Gaps** — new rule catches something not covered by any existing rule (additive value)
- **Conflicts** — new rule contradicts or undermines an existing rule (needs resolution)
- **Refinement opportunities** — existing rule could be tightened based on new rule's pattern

**Output:**
A comparison report per generated rule — overlap/gap/conflict classification with reference to the specific existing rules involved.

---

### 5. Classifier

The classifier organizes findings and rules by attack type to enable focused analysis and avoid mixing signals across vulnerability classes.

**Current attack type coverage:**
- XSS (Cross-Site Scripting)
- SQL Injection

**Why classification matters:**
A ruleset refined against XSS history produces different signal than one refined against SQLi history. Keeping them separated allows the pipeline to be run and evaluated per attack type, and makes it easier to expand coverage incrementally.

**Output:**
Attack-type-tagged findings and rules, organized for targeted validation.

---

### 6. Validator

Generated rules are validated against two corpora before being considered for inclusion in the refined ruleset.

**Historical corpus:**
A curated set of real vulnerable code patterns drawn from prior findings. Confirms the generated rule catches the class of vulnerability it was designed for.

**Default regression set:**
A baseline set of known patterns confirming new rules don't introduce regressions in existing coverage — nothing that was working before should break.

**Validation logic:**
- Rule must fire on historical corpus patterns (true positive confirmation)
- Rule must not fire incorrectly on regression set (false positive check)
- Rules that fail either check are flagged for AI review and regeneration

**Output:**
Validated rule set with pass/fail status per rule and per corpus.

---

### 7. Metrics *(In Progress)*

The metrics component is designed to quantify the impact of the refined ruleset compared to the baseline org rules.

**Planned measurements:**
- False positive rate delta (before vs after refinement)
- True positive coverage delta
- Net new coverage added by generated rules
- Redundancy reduction from overlap identification

**Status:**
Partially implemented. Full metrics output is on the roadmap.

---

## Stack

| Component | Language/Tool | Role |
|---|---|---|
| Harvester | Python | Jira API integration, finding extraction |
| Git analyzer | Python + Git CLI | Agentic diff retrieval per finding |
| Rule generator | Python + LLM | Dynamic Semgrep rule generation |
| Org comparator | Python + Semgrep | Ruleset gap/overlap analysis |
| Classifier | Python | Attack type categorization |
| Validator | Python + Semgrep | Rule testing against corpora |
| Metrics | Python *(in progress)* | Coverage and FP delta measurement |

---

## Key Design Decisions

**History-first, not opinion-first**
Rules are generated from evidence — real findings, real diffs, real fixes — not from general best practices or gut feel. This grounds the ruleset in the actual risk profile of the codebase.

**Agentic git retrieval**
The AI orchestrates its own git commands rather than receiving pre-processed diffs. This preserves fidelity to the raw history and avoids information loss from summarization.

**Dynamic over static generation**
Static rule templates can't reason about novel patterns. Dynamic AI generation handles each finding on its own terms and improves as the model improves.

**Year-by-year processing**
Processing findings in annual chunks keeps runs manageable and allows incremental expansion of historical coverage without overwhelming the pipeline.

**Attack type scoping**
Starting with XSS and SQLi allows the pipeline to be validated against well-understood vulnerability classes before expanding to more complex or ambiguous patterns.

---

*See [ROADMAP.md](./ROADMAP.md) for planned features and coverage expansions.*
