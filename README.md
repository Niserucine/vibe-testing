# Vibe Testing

**Pressure-test your specs with LLM reasoning before writing code.**

Vibe testing is a technique for validating specification documents by simulating real-world scenarios against them. An LLM reads your spec docs, traces through a concrete user scenario step by step, and flags every gap, conflict, and ambiguity — before anyone writes a line of implementation.

## Why

We test code obsessively. Unit tests, integration tests, E2E tests. But specifications? We "review" them in a meeting.

Vibe testing moves the discovery of design flaws to the cheapest possible moment: before implementation begins.

## How It Works

```
1. Write a scenario: a named persona, a concrete goal, step-by-step interaction
2. Give an LLM all your spec docs + the scenario
3. The LLM traces each step, identifies governing specs, flags gaps
4. You get a structured gap report with severity ratings
```

No code. No test harness. Just reasoning.

## Install as Agent Skill

Works with Claude Code, OpenAI Codex, Gemini CLI, Cursor, GitHub Copilot, OpenCode, and any tool supporting the Agent Skills open standard.

### Claude Code

```bash
# Copy to your skills directory
cp -r vibe-testing ~/.claude/skills/

# Or clone directly
git clone https://github.com/jian-mo/vibe-testing.git ~/.claude/skills/vibe-testing
```

### OpenAI Codex

```bash
cp -r vibe-testing ~/.codex/skills/
```

### Gemini CLI

```bash
cp -r vibe-testing ~/.gemini/skills/
```

### Universal (works with most agents)

```bash
cp -r vibe-testing ~/.agent/skills/
```

### Project-level (shared with team)

```bash
# Add to your project repo
cp -r vibe-testing .claude/skills/
# or
cp -r vibe-testing .agent/skills/
```

## Usage

Once installed, the skill activates when you ask your coding agent to validate specs:

```
> /vibe-testing

> "Test my specs against a realistic deployment scenario"

> "Find gaps in the architecture docs before we start building"

> "Vibe test the design docs in docs/v2/"
```

## What's Included

```
vibe-testing/
├── SKILL.md                          # The skill definition (Agent Skills standard)
├── references/
│   └── simulator-prompt.md           # Copy-paste prompt templates
└── examples/
    └── example-vibe-test.md          # Complete example test case
```

## The Gap Report

Vibe tests produce a structured gap report:

| Severity | Meaning |
|----------|---------|
| **BLOCKING** | Spec cannot answer. Implementation impossible without resolution. |
| **DEGRADED** | Workaround exists but it's fragile. |
| **COSMETIC** | Missing convenience. Not a correctness issue. |

## Results

Applied to a 20+ document system specification with 4 vibe test cases covering different deployment contexts:

- **6 blocking gaps** found (no web UI spec, no persistent automation concept, no PostgreSQL schema for distributed mode, no HA story)
- **8 degraded gaps** found (no default security policy, no filesystem-watch trigger, no sleep/delay primitive)
- **5 cosmetic gaps** found (no one-click deploy, no monitoring dashboard)

Each would have been a rewrite-level discovery weeks into implementation.

## License

MIT
