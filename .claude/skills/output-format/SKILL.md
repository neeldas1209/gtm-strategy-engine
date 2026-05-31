---
description: >
  Output formatting standards for GTM agents. Use when a subagent is
  producing its final output. Covers JSON schema compliance, condensed
  summary generation for Agent 1 sub-agents, and file save conventions.
---

# Output Format Skill

This skill defines how all GTM subagents structure and save their output. Follow these rules when producing your final deliverable.

## JSON Output Structure

All agent outputs are JSON. No markdown, no plain text, no mixed formats.

### Required Top-Level Fields (All Agents)

```json
{
  "agent": "agent-name",
  "company": "company-name-from-run",
  "scenario_type": "scenario_type_value",
  "generated_at": "ISO 8601 timestamp",
  "questions": {
    // Agent-specific question outputs
  },
  "metadata": {
    "search_queries_used": 0,
    "sources_cited": 0,
    "scenario_emphasis_applied": "description of emphasis"
  }
}
```

### Per-Question Output Structure

Each question within the `questions` object follows this pattern:

```json
{
  "question_id": "Q1",
  "question_title": "Short title",
  "confidence": "High | Medium | Low",
  "findings": {
    // Question-specific structured data (varies by question)
  },
  "sources": [
    {
      "name": "Source Name",
      "url": "https://...",
      "accessed": "date",
      "relevance": "What this source contributed"
    }
  ],
  "gaps": [
    "Description of what could not be found or verified"
  ]
}
```

## Condensed Summary Requirements (Agent 1 Sub-Agents Only)

Agent 1 sub-agents (market-sizing, geography, competitive-landscape, market-structure) must include a `condensed_summary` field at the top level of their output. This is critical because Stage 2 sub-agents receive Stage 1 condensed summaries as context.

### Rules for Condensed Summaries

- Length: 3-5 sentences, no more
- Content: the most important findings that downstream agents need to know
- No raw data dumps. Synthesize into actionable intelligence
- Must include: the single most important finding, any critical gaps or uncertainties, and the confidence level

### Example Condensed Summary

```json
{
  "condensed_summary": "The AEO/GEO monitoring market is an emerging category with an estimated TAM of $500M-$800M (sources diverge, flagged). Growth is accelerating as AI-generated search results capture 15-25% of traditional search traffic (Gartner 2025). The market is pre-consolidation with fewer than 10 dedicated players. Key gap: no reliable SAM data exists for the agency-focused segment specifically. Confidence: Medium overall due to limited analyst coverage of this specific category."
}
```

### What NOT to Include in Condensed Summaries

- Full competitor profiles (just mention count and key finding)
- Raw market size ranges with all sources (just the headline figure)
- Detailed methodology or search process
- Caveats that belong in the full output, not the summary

## File Save Conventions

### Agent 1 Sub-Agent Outputs (Staging)

Save to: `runs/<company-name>/staging/<subagent-name>.json`

Files:
- `runs/<company-name>/staging/market-sizing.json`
- `runs/<company-name>/staging/geography.json`
- `runs/<company-name>/staging/competitive-landscape.json`
- `runs/<company-name>/staging/market-structure.json`

These are intermediate outputs. The orchestrator reads them, synthesizes them, and produces the final Agent 1 output.

### Final Agent Outputs

Save to: `runs/<company-name>/outputs/agent<N>-<agent-name>.json`

Files:
- `runs/<company-name>/outputs/agent1-market-intelligence.json` (synthesized by orchestrator)
- `runs/<company-name>/outputs/agent2-icp-definition.json`
- `runs/<company-name>/outputs/agent3-channel-strategy.json`
- `runs/<company-name>/outputs/agent4-messaging-framework.json`
- `runs/<company-name>/outputs/agent5-ninety-day-plan.json`

### File Naming Rules

- All lowercase
- Hyphens between words, not underscores
- Company name in the path uses the same format the user provided (normalized to lowercase, hyphens for spaces)
- Always create the directory path before writing the file

## Schema Validation

Before saving output, validate against the corresponding schema in `schemas/`:

- `schemas/market-intelligence.schema.json` (for the final synthesized Agent 1 output)
- `schemas/icp-definition.schema.json`
- `schemas/channel-strategy.schema.json`
- `schemas/messaging-framework.schema.json`
- `schemas/ninety-day-plan.schema.json`

Sub-agent staging outputs (market-sizing, geography, competitive-landscape, market-structure) do not have individual schemas. They follow the per-question structure defined above plus the condensed_summary requirement.

## Output Size Guidelines

- Agent 1 sub-agent staging outputs: 2,000-5,000 tokens each
- Agent 1 final synthesized output: 8,000-15,000 tokens
- Agents 2-5 final outputs: 3,000-8,000 tokens each

These are guidelines. Thoroughness matters more than length, but padding with filler is worse than being concise.
