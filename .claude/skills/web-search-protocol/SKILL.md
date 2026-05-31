---
description: >
  Web search protocol for GTM agents. Use whenever a subagent needs to do
  web research. Covers query formulation, source evaluation, citation format,
  and handling conflicting data.
---

# Web Search Protocol

This skill defines how all GTM subagents conduct web research. Follow these rules for every search query and every factual claim.

## Query Formulation

- Keep queries short and specific: 1-6 words
- Start broad (1-2 words), then narrow based on initial results
- Every follow-up query must be meaningfully different from previous ones
- Include year or date when searching for current market data
- Never use `-`, `site:`, or quotes in queries unless specifically needed
- Use content nouns (the topic, company name, product), not meta-words like "discussed" or "information"

### Example Queries

Good: `AEO platform market size 2026`
Bad: `information about the current market size for answer engine optimization platforms`

Good: `Profound AI competitors`
Bad: `who are the main competitors of Profound AI in the AEO space`

Good: `enterprise content marketing budget trends`
Bad: `recent trends and data about how much enterprise companies spend on content marketing`

## Agent-Specific Search Budgets

Each agent type has a different search budget reflecting its research intensity:

| Agent | Budget | Rationale |
|-------|--------|-----------|
| Agent 1 sub-agents (market-sizing, geography, competitive-landscape, market-structure) | 8-12 queries each | Broad primary research, building the evidence base from scratch |
| Agent 2 (icp-definition) | 6-10 queries | Enrichment research, refining and validating Agent 1 findings for ICP |
| Agent 3 (channel-strategy) | 8-15 queries | Validation research, confirming channel assumptions with market data |
| Agent 4 (messaging-framework) | 5-10 queries | Targeted research, specific competitor messaging and positioning claims |
| Agent 5 (ninety-day-plan) | 3-5 queries | Benchmark queries only, looking up industry benchmarks for KPI targets |

Budget is a guideline, not a hard limit. If you have high confidence from fewer searches, stop early. If a critical question remains unanswered, go over budget and note it.

## Source Quality Hierarchy

Prefer sources in this order:

1. **Official company sources**: pricing pages, product pages, investor relations, press releases
2. **Industry analyst reports**: Gartner, Forrester, IDC, CB Insights (even partial access or summaries)
3. **Financial filings and data**: SEC filings, Crunchbase funding data, annual reports
4. **Industry publications**: TechCrunch, VentureBeat, industry-specific trade publications
5. **Review platforms**: G2, Capterra, TrustRadius (for competitive intelligence and user sentiment)
6. **Job boards**: LinkedIn, Glassdoor, Indeed (for inferring company GTM motions and growth signals)
7. **Community sources**: Reddit, Twitter/X, Hacker News (use cautiously, note as community sentiment)

Never treat Wikipedia, generic blog posts, or AI-generated content aggregators as primary sources.

## Citation Format

Every factual claim must cite a source. Use this format in your output:

```
"source": "Company Name or Publication - URL if available"
```

### Citation Rules

- If you found it via web search, include the URL
- If you found it but the URL is behind a paywall, cite the publication name and article title
- If multiple sources confirm the same fact, cite the strongest one and note "corroborated by [second source]"
- If you cannot find a source for a claim, do NOT make the claim. State "unable to verify" instead

## Handling Conflicting Data

When sources disagree:

1. **Note the discrepancy explicitly** in your output using the `discrepancy_flag` field
2. **Prefer the most recent source** if the data is time-sensitive (market size, funding, headcount)
3. **Prefer the most authoritative source** if the data is not time-sensitive (official filings over blog posts)
4. **If sources diverge by more than 30%** on a numerical claim (market size, growth rate), flag it as a discrepancy and present both figures with their sources
5. **Never silently pick one** and ignore the other

## Confidence Ratings

Rate confidence per question or per section, not per agent:

| Rating | Criteria |
|--------|----------|
| High | Multiple corroborating sources, official/analyst data, recent (within 12 months) |
| Medium | Limited but credible sources, some inference required, data may be 12-24 months old |
| Low | Single source, community/anecdotal evidence, significant inference, or data older than 24 months |

## Hallucination Prevention

This is the most critical section. GTM research is particularly prone to hallucination because market data, competitor details, and funding numbers sound plausible even when fabricated.

### Hard Rules

- **Never invent market size numbers.** If you cannot find TAM/SAM data, say "market size data not publicly available" and describe the market qualitatively
- **Never fabricate funding rounds.** If Crunchbase or press releases do not confirm a funding round, do not claim it happened
- **Never invent competitor features.** Only describe features you found on the product's website or in reviews
- **Never assume GTM motions.** GTM patterns are inferred from observable signals. Always cite what evidence you found:
  - Job postings (BDR roles = outbound, no BDR roles = likely PLG or inbound)
  - Pricing pages (self-serve pricing = PLG, "contact sales" = enterprise sales)
  - Partner programs (listed partner page = channel strategy exists)
  - Content patterns (high blog volume = content marketing motion)
- **Per-question confidence ratings** must honestly reflect source quality, not how confident the language sounds

### When You Cannot Find Data

Say exactly what you searched for and what you did not find:

```
"Unable to verify market size. Searched: '[query 1]', '[query 2]', '[query 3]'. 
No analyst reports or financial filings found for this specific segment. 
Qualitative assessment: [your inference based on available signals]."
```

This is vastly more useful than inventing a number.
