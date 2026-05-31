# GTM Strategy Engine

## Project Overview
AI-powered Go-To-Market Strategy Engine. Takes a product brief and scenario
context, then produces a complete GTM strategy through a 5-agent pipeline
with human review gates between each agent.

## Architecture
- 1 orchestrator agent (gtm-orchestrator.md, runs as main thread)
- 8 subagents (4 for Market Intelligence, 4 for Agents 2-5)
- 4 skills (web-search-protocol, quality-check, output-format, human-review)
- All subagents are spawned by the orchestrator only
- Skills are passive reference documents. Skills never call agents.

## Directory Conventions
- runs/<company-name>/inputs/ — user inputs for this run
- runs/<company-name>/staging/ — Agent 1 intermediate sub-agent outputs
- runs/<company-name>/outputs/ — final agent outputs (one per agent)
- runs/<company-name>/review/ — human review status and notes
- schemas/ — output schema files (shared across all runs)

## Universal Constraints
- Never invent product capabilities not stated in the product brief
- Never introduce channels in the 90-Day Plan not recommended by Channel Strategy
- Always save outputs as JSON to the correct run directory
- Always cite sources for factual claims
- Confidence ratings are per-question, not per-agent
- If the product brief is too thin to support a claim, flag it rather than guessing
- All 10 Market Intelligence questions run regardless of scenario type (scenario changes depth, not presence)

## Scenario Types
- new_product_launch: launching a new or early-stage product
- new_market_entry: expanding into a new geographic region
- segment_expansion: moving into a new customer segment (e.g., SMB to enterprise)
- product_repositioning: shifting positioning due to market changes or disruption
- competitive_response: reacting to a specific competitive threat

## Web Search Budgets
- Agent 1 sub-agents: 8-12 queries each (broad research)
- Agent 2 (ICP Definition): 6-10 queries (enrichment)
- Agent 3 (Channel Strategy): 8-15 queries (validation)
- Agent 4 (Messaging Framework): 5-10 queries (targeted)
- Agent 5 (90-Day Plan): 3-5 queries (benchmarks only)
