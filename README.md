# GTM Strategy Engine

An AI-powered Go-To-Market strategy engine built on **Claude Code's multi-agent
orchestration**. It takes a product brief and scenario context, then produces a
complete, source-cited GTM strategy through a pipeline of specialized AI agents
with **human review gates** between each stage.

This repo is both the system (agents, skills, schemas) and a complete worked
example: a full GTM strategy for **Docebo AgentHub**, an enterprise AI learning
platform.

---

## Why this is interesting

Most "AI does your strategy" demos are a single prompt that returns a wall of
unverifiable text. This engine is built like a real analyst team:

- **Separation of concerns** — each agent owns one job (market sizing, ICP,
  channels, messaging, execution) instead of one model doing everything.
- **Grounded, not hallucinated** — agents do live web research and must **cite
  sources** for every factual claim, with explicit per-question confidence
  ratings and flagged gaps.
- **Structured output** — every agent emits JSON validated against a schema in
  [`schemas/`](schemas/), so outputs are consistent and machine-consumable.
- **Human-in-the-loop** — a review gate sits between every agent; downstream
  agents only run on *approved* upstream output.
- **Guardrails** — explicit constraints prevent the model from inventing product
  capabilities or recommending channels that earlier stages didn't justify.

---

## How it works

```
Product brief + scenario context
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│  Agent 1 — Market Intelligence  (10 questions, Q1–Q10)       │
│  built from 4 parallel research sub-agents:                  │
│   • market-sizing   • geography                             │
│   • competitive-landscape   • market-structure              │
└─────────────────────────────────────────────────────────────┘
        │   ▼ human review gate
        ▼
   Agent 2 — ICP Definition          (6 ICP dimensions)
        │   ▼ human review gate
        ▼
   Agent 3 — Channel Strategy        (GTM motion + channels + partnerships)
        │   ▼ human review gate
        ▼
   Agent 4 — Messaging Framework     (positioning + per-persona messaging)
        │   ▼ human review gate
        ▼
   Agent 5 — 90-Day Plan             (phased execution, milestones, metrics)
        │
        ▼
   Final GTM strategy (one JSON output per agent) + executive summary
```

An **orchestrator** agent collects inputs, runs each agent in sequence, manages
the review gates, and passes approved output downstream. Each later agent only
consumes the *approved* outputs of the agents before it — so the strategy
compounds instead of contradicting itself.

### The agents

| Stage | Agent | Job |
|------|-------|-----|
| Orchestrator | `gtm-orchestrator` | Runs the pipeline, manages review gates and data flow |
| 1 | `market-sizing`, `geography`, `competitive-landscape`, `market-structure` | Market intelligence research (10 questions) |
| 2 | `icp-definition` | Ideal Customer Profile across 6 dimensions |
| 3 | `channel-strategy` | GTM motion, channel prioritization, partnerships |
| 4 | `messaging-framework` | Positioning angles and per-persona messaging |
| 5 | `ninety-day-plan` | Concrete phased 90-day execution plan |

### The skills

Passive reference documents that enforce consistency across agents:
`web-search-protocol`, `quality-check`, `output-format`, `human-review`.

---

## Repo structure

```
.claude/
  agents/        # Orchestrator + 8 subagent definitions (the system prompts)
  skills/        # Shared protocols: web search, quality check, output format, review
schemas/         # JSON output schema per agent (validation contract)
runs/
  docebo-agenthub/
    inputs/      # The product brief + scenario that drove this run
    staging/     # Agent 1's intermediate sub-agent outputs
    outputs/     # Final output per agent (JSON) + executive summary
    review/      # Human review notes per agent
    gtm-engine-full-output.md   # All agent outputs consolidated in one file
CLAUDE.md        # Project instructions / constraints the agents operate under
```

---

## Worked example: Docebo AgentHub

The [`runs/docebo-agenthub/`](runs/docebo-agenthub/) directory is a complete run
for a real product scenario — launching Docebo AgentHub, an agentic-AI platform
that turns enterprise knowledge (Confluence, SharePoint, Google Drive, Slack,
Teams, etc.) into trackable learning content for enterprise L&D teams.

Start here:

- **[Executive summary](runs/docebo-agenthub/outputs/executive-summary.md)** — the one-page takeaway.
- **[Full agent outputs](runs/docebo-agenthub/gtm-engine-full-output.md)** — every agent's complete output in one file.
- Individual JSON outputs in [`outputs/`](runs/docebo-agenthub/outputs/).

---

## Built with

[Claude Code](https://claude.com/claude-code) — agents are defined as markdown
files with frontmatter; skills are reusable reference documents; the orchestrator
coordinates subagents and human review gates.
