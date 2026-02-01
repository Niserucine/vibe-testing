---
name: vibe-testing
description: >
  Use when validating spec documents, design docs, or architecture docs for
  completeness and coherence. Use when the user asks to "test my specs",
  "validate my design docs", "find gaps in my architecture", "stress-test
  the spec", "vibe test", "pressure test the docs", or mentions spec
  validation before implementation begins.
version: 1.0.0
---

# Vibe Testing

## Overview

Vibe testing validates specification documents by simulating real-world scenarios against them using LLM reasoning. Instead of writing code or test harnesses, write **natural-language scenarios** that exercise cross-cutting slices of the spec surface — then trace execution step-by-step, flagging gaps, conflicts, and ambiguities.

**Core principle:** If a realistic user scenario cannot be fully traced through the specs, the specs are incomplete.

**Best used:** After specs are written, before implementation begins.

## When to Use

- Spec docs exist but no implementation yet — validate before building
- After major spec changes — regression-test for new gaps
- Before implementation planning — find blocking gaps early
- Specs span multiple documents — test cross-doc coherence
- Designing for multiple deployment contexts — test each context separately

**When NOT to use:**

- Single-file specs with obvious scope — just review manually
- Implementation bugs — use actual tests
- API contract validation — use schema validation tools

## Core Method

```
1. GATHER    — Read all spec docs in the target directory
2. SCENARIOS — Write 3-5 vibe test cases (personas + goals + environments)
3. SIMULATE  — Trace each scenario step-by-step against the specs
4. CLASSIFY  — Tag findings as GAP / CONFLICT / AMBIGUITY
5. SEVERITY  — Rate as BLOCKING / DEGRADED / COSMETIC
6. REPORT    — Produce gap summary + spec coverage matrix
```

## Writing a Vibe Test Case

Every test case requires 7 sections:

### 1. Persona (WHO)

A concrete person with a name, role, and technical skill level. Not abstract — real enough to predict behavior.

```markdown
**Maya** — Owner of a 12-person e-commerce business.
Not a developer. Comfortable with Zapier and Shopify admin.
Has never written code or used a terminal.
```

Named personas force specificity. "A non-technical user" invites hand-waving. "Maya, who has never used a terminal" forces the spec to answer "how does Maya do this?"

### 2. Environment (WHERE)

Deployment mode, hardware, network, access method. Different environments exercise different spec paths.

```markdown
- **Deployment:** Cloud-hosted SaaS instance
- **Runners:** Platform runner only (gVisor sandbox)
- **Access:** Web browser only (no CLI, no SSH)
```

### 3. Goal (WHAT)

A single sentence in the persona's own words. Use a blockquote.

```markdown
> "When a new order comes in on Shopify, automatically create an invoice
> in QuickBooks and send a confirmation email."
```

### 4. Scenario Steps (HOW)

5-8 concrete steps the persona takes. Each step names:

- **The user action** — what they do
- **The primitives exercised** — which spec concepts activate
- **Gap detection questions** — 2-3 questions the simulator must answer

```markdown
#### Step 3: Agent installs connectors

The agent searches the registry and installs Shopify, QuickBooks connectors.

**Primitives:**
- `registry.md`: search, install
- `connectors.md`: ConnectorDefinition, AuthDefinition

**Questions:**
- Q3.1: Does the install flow handle OAuth2 consent?
- Q3.2: Who fills in the config — the agent or the user?
```

**Rules for good steps:**

- Each step must cite at least one spec doc
- Each step must ask at least one question the spec should answer
- Questions use `Q<step>.<number>:` format for traceability
- Questions must be spec-answerable (yes/no/how), not opinion questions

### 5. Spec Coverage Matrix (COVERAGE)

A table showing which spec docs were exercised at which steps.

```markdown
| Spec Doc | Steps Hit | Coverage |
|----------|-----------|----------|
| `connectors.md` | 3,4,5 | Core coverage; OAuth UX gap |
| `registry.md` | 3 | Search/install covered |
| `memory-module.md` | — | Not exercised |
```

Specs that no scenario touches are untested blind spots.

### 6. Gap Detection Questions Summary

Collect all Q-numbers for easy reference. The simulator answers every one.

### 7. Gap Classification (After Simulation)

Classify each finding by severity:

| Severity | Definition | Example |
|----------|-----------|---------|
| **BLOCKING** | Spec cannot answer; implementation impossible | No auth schema but multi-tenant mode requires it |
| **DEGRADED** | Spec is silent but a workaround exists | No file-watch trigger; use cron polling instead |
| **COSMETIC** | Missing convenience, not a correctness issue | No bulk enrollment CLI command |

## Running a Vibe Test

### Manual Simulation (Recommended)

Use as a prompt to a subagent or fresh LLM context with full spec access:

```
You are a spec validation simulator. You have been given all
specification documents for [system name].

Read the following vibe test case. Simulate executing the scenario
step by step against the specs.

For each step:
1. Identify the governing spec document and section
2. Trace the data flow through the system primitives
3. Answer every Q-numbered question by citing the spec

For each question, classify as:
- COVERED: The spec answers this clearly. Cite the section.
- GAP: The spec is silent. No document addresses this.
- CONFLICT: Two specs give contradictory answers. Cite both.
- AMBIGUITY: The spec addresses this but the answer is unclear.

After all steps, produce:
- Gap summary table (ID, description, severity, affected steps)
- Spec coverage heatmap (which docs exercised, which not)
- Recommended spec changes (which doc to update, what to add)
```

### Batch Execution

Run all test cases and aggregate:

```
for each test case:
    1. Load all spec docs as context
    2. Load one test case
    3. Run simulator prompt
    4. Collect gap report

Aggregate:
    - Cross-test gap summary (gaps appearing in multiple tests)
    - Spec coverage union (docs never exercised by any test)
    - Priority ranking (blocking > degraded > cosmetic)
```

### Regression Testing

After spec updates, re-run all vibe tests to verify:

1. Previously identified gaps are now COVERED
2. No new gaps were introduced
3. Cross-doc references remain consistent

## Designing Good Test Cases

### Scenario Selection Strategy

Choose scenarios that vary across dimensions:

| Dimension | Variation A | Variation B | Variation C |
|-----------|------------|------------|------------|
| **User skill** | Non-technical | Developer | SRE/Ops |
| **Deployment** | Local/single machine | Cloud SaaS | Distributed enterprise |
| **Scale** | Single user | Multi-tenant (1000s) | Global multi-region |
| **Access** | Web browser | CLI | API/programmatic |
| **Governance** | None (personal) | Moderate (team) | Strict (SOC2/compliance) |
| **Network** | Always online | Intermittent | Air-gapped |

Each test case should differ on at least 3 dimensions. 4 test cases covering 4 quadrants give good coverage.

### Question Design

Good gap detection questions are:

- **Specific:** "Who initiates the OAuth2 browser redirect?" not "How does auth work?"
- **Traceable:** Answerable by citing a spec section (or flagging its absence)
- **Boundary-probing:** Target edges between two specs' responsibilities
- **Scale-sensitive:** "What happens with 10,000 concurrent X?"
- **Failure-aware:** "What if Y fails/expires/crashes at this point?"

### Coverage Maximization

After writing all test cases, check the coverage union. Every spec doc should appear in at least one coverage matrix. If a doc is never exercised:

- Either add a step to an existing test that exercises it
- Or the doc may be specifying something no real scenario needs (flag for review)

## Gap Report Format

```markdown
## Gap Summary

### BLOCKING
| ID | Gap | Affected Tests | Recommended Fix |
|----|-----|---------------|-----------------|
| G-B1 | No web UI spec | VT-1, VT-2 | New: web-ui-spec.md |

### DEGRADED
| ID | Gap | Affected Tests | Workaround |
|----|-----|---------------|-----------|
| G-D1 | No FileWatch trigger | VT-3 | Use cron polling |

### COSMETIC
| ID | Gap | Affected Tests |
|----|-----|---------------|
| G-C1 | No bulk enrollment CLI | VT-4 |
```

Gap IDs use prefix: `G-B` (blocking), `G-D` (degraded), `G-C` (cosmetic).

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Abstract personas ("a user") | Give them names, roles, and constraints |
| Scenario only tests happy path | Add failure steps: "What if the token expires?" |
| Questions test opinions ("Is this good?") | Questions must be spec-answerable: "Which doc defines X?" |
| All tests use same deployment mode | Vary environment across tests |
| Ignoring coverage matrix | Every spec doc must appear in at least one test |
| Writing tests after implementation | Vibe tests validate specs BEFORE implementation |
| Too many steps per scenario | 5-8 steps. Focused scenarios find more gaps |

## Additional Resources

- `references/simulator-prompt.md` — Full simulator prompt template ready to paste
- `examples/example-vibe-test.md` — Complete example vibe test case
