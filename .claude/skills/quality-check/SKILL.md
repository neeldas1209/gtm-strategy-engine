---
description: >
  Run quality checks on GTM agent output. Use after any subagent completes
  its work. Validates output against agent-specific quality criteria.
  Reports pass/fail with specific failure reasons for re-prompting.
disable-model-invocation: true
---

# Quality Check Skill

Run these checks after every subagent returns its output. Report pass/fail with specific failure reasons. The orchestrator uses failure reasons to re-prompt the subagent (max 2 retries before escalating to human review).

## Generic Validation (All Agents)

Run these checks first. If any fail, stop and return the failure before running agent-specific checks.

| Check | Pass Criteria | Failure Message |
|-------|---------------|-----------------|
| Valid JSON | Output parses as valid JSON | "Output is not valid JSON. Raw output returned." |
| Schema compliance | All required fields present per the agent's schema in `schemas/` | "Missing required fields: [list fields]" |
| No empty required fields | Every required field has a non-empty value | "Empty required fields: [list fields]" |
| Source citations | Every factual claim has a source field that is not empty | "Unsourced factual claims found in: [list sections]" |
| Confidence ratings | Every question/section has a confidence rating of High, Medium, or Low | "Missing confidence ratings in: [list sections]" |
| File saved correctly | Output file exists at the expected path | "Output file not saved to expected path: [expected path]" |

## Agent 1 Sub-Agent Checks

### market-sizing (Q1 + Q8)

| Check | Pass Criteria | Failure Message |
|-------|---------------|-----------------|
| Market size sourced | TAM/SAM figures cite at least 2 independent sources, OR explicitly states data is unavailable with search queries listed | "Market size claims need at least 2 sources or explicit 'unable to verify' with search queries" |
| Growth rate sourced | Growth rate or trajectory cites a source, not inferred from training data | "Growth rate appears unsourced. Cite analyst report, financial data, or state unable to verify" |
| Maturity classification evidenced | Market maturity stage (Emerging/Growth/Consolidating/Mature) has 3+ supporting evidence points | "Maturity stage needs at least 3 evidence points, found [N]" |
| Discrepancy flags | If multiple market size sources exist, discrepancies >30% are flagged | "Market size sources diverge >30% but no discrepancy flag set" |
| Condensed summary present | condensed_summary field exists and is 3-5 sentences | "condensed_summary missing or not 3-5 sentences (found [N] sentences)" |
| Macro trend specific | Q8 macro trend identifies a specific shift, not a generic trend like "digital transformation" | "Macro trend is too generic. Must identify a specific, named shift with evidence" |

### geography (Q2 + Q9)

| Check | Pass Criteria | Failure Message |
|-------|---------------|-----------------|
| Regional data present | At least 2 geographic regions analyzed with specific data points | "Geographic analysis needs at least 2 regions with specific data" |
| Buying process observable | Decision committee uses specific titles (e.g., "VP of Marketing") not generic roles (e.g., "decision maker") | "Buying process uses generic roles. Need specific titles sourced from job boards or reports" |
| Deal timeline specific | Sales cycle estimate uses specific ranges (e.g., "30-60 days") not vague language (e.g., "several weeks") | "Sales cycle needs specific timeframe, not vague language" |
| Condensed summary present | condensed_summary field exists and is 3-5 sentences | "condensed_summary missing or not 3-5 sentences" |
| Buying patterns sourced | Buying pattern claims cite Gartner/Forrester data, industry surveys, or observable evidence | "Buying patterns need sourced evidence, not assumptions" |

### competitive-landscape (Q3 + Q4 + Q5)

| Check | Pass Criteria | Failure Message |
|-------|---------------|-----------------|
| Competitor count | At least 3 direct competitors profiled (broaden scope if fewer than 4 exist) | "Need at least 3 competitor profiles" |
| Current data | Competitor profiles use data from web search (URLs cited), not training data | "Competitor profiles appear to use training data. All profiles must cite current web sources" |
| Substitutes beyond obvious | Substitutes include creative alternatives (spreadsheets, manual processes, adjacent tools), not just direct competitors repackaged | "Substitutes section looks like competitor list. Include non-obvious alternatives" |
| GTM patterns evidenced | Each GTM pattern inference cites observable evidence (job postings, pricing pages, content patterns) | "GTM patterns need observable evidence citations, not assumptions" |
| Weaknesses sourced | Competitor weaknesses cite observable evidence (G2/Capterra reviews, Reddit, feature gaps on product pages) | "Weaknesses need sourced evidence, not speculation" |
| Condensed summary present | condensed_summary field exists and is 3-5 sentences | "condensed_summary missing or not 3-5 sentences" |

### market-structure (Q6 + Q7 + Q10)

| Check | Pass Criteria | Failure Message |
|-------|---------------|-----------------|
| Pricing data sourced | Every pricing data point cites a source URL or notes "behind contact sales wall" | "Pricing claims need source URLs or explicit 'behind contact sales wall' notation" |
| Value metric identified | A dominant value metric is identified (per seat, per usage, per feature tier) | "No dominant value metric identified for the market" |
| Ecosystem mapped | At least one adjacent platform/ecosystem identified with integration relevance | "Ecosystem section empty. Identify platforms used alongside products in this category" |
| Disruption risks rated | Each disruption risk has both probability AND impact rating | "Disruption risks need both probability and impact ratings" |
| Platform risk assessed | At least one assessment of big-tech entry risk (Google, Microsoft, Salesforce, etc.) | "Missing platform/big-tech entry risk assessment" |
| Condensed summary present | condensed_summary field exists and is 3-5 sentences | "condensed_summary missing or not 3-5 sentences" |

## Agent 2 Checks (icp-definition)

| Check | Pass Criteria | Failure Message |
|-------|---------------|-----------------|
| Six dimensions covered | All 6 ICP dimensions present: firmographic, technographic, behavioral, need-based, buying committee, negative indicators | "Missing ICP dimensions: [list missing]" |
| Specific titles | Buying committee uses specific job titles, not generic roles | "Buying committee needs specific titles, not generic roles" |
| Negative indicators present | At least 3 negative/exclusion indicators defined | "Need at least 3 negative indicators (who is NOT the ICP)" |
| Grounded in Agent 1 | ICP claims reference or build on Agent 1 Market Intelligence findings | "ICP appears disconnected from Market Intelligence findings" |
| Prioritized segments | If multiple segments, they are ranked with rationale | "Multiple segments identified but not prioritized" |

## Agent 3 Checks (channel-strategy)

| Check | Pass Criteria | Failure Message |
|-------|---------------|-----------------|
| GTM motion defined | Primary GTM motion explicitly stated (PLG, sales-led, channel, hybrid) with rationale | "No primary GTM motion defined or missing rationale" |
| Channels prioritized | Channels ranked by priority with expected impact and rationale | "Channels listed but not prioritized with rationale" |
| Partnership strategy present | At least one partnership or channel partner type identified | "No partnership strategy defined" |
| Aligned with ICP | Channel recommendations reference ICP buying behavior from Agent 2 | "Channel strategy appears disconnected from ICP buying behavior" |
| Validation sourced | Channel recommendations cite validation evidence (competitor channel usage, industry benchmarks) | "Channel recommendations need validation evidence" |

## Agent 4 Checks (messaging-framework)

| Check | Pass Criteria | Failure Message |
|-------|---------------|-----------------|
| Narrative present | Market narrative section exists with a clear "from/to" shift articulated | "Missing market narrative or 'from/to' shift" |
| Positioning angles defined | At least 3 positioning angles with supporting evidence | "Need at least 3 positioning angles" |
| Differentiation grounded | Differentiation claims reference competitive findings from Agent 1 | "Differentiation claims not grounded in competitive analysis" |
| No copy production | Output contains frameworks and angles, NOT finished ad copy, email templates, or taglines | "Agent 4 produces messaging frameworks, not finished copy. Remove copy/templates" |
| Scenario alignment | Positioning reflects the scenario type (especially critical for product_repositioning) | "Positioning does not reflect scenario-specific requirements" |

## Agent 5 Checks (ninety-day-plan)

| Check | Pass Criteria | Failure Message |
|-------|---------------|-----------------|
| Three phases present | Month 1 (Foundation), Month 2-3 (Activation + Optimization) all defined | "Missing plan phases. Need Month 1, Month 2, and Month 3" |
| KPIs have benchmarks | Key metrics cite industry benchmarks or rationale for targets | "KPI targets need benchmark sources or rationale" |
| Builds on all agents | Plan references findings from Agents 1-4 (market, ICP, channels, messaging) | "90-day plan appears disconnected from prior agent outputs" |
| Actionable specificity | Actions are specific enough to execute (not "build awareness" but "publish 3 comparison blog posts targeting [keyword]") | "Actions too vague. Need specific, executable tasks" |
| Resource-realistic | Plan accounts for team size and budget from existing constraints input | "Plan does not account for stated resource constraints" |

## Failure Handling

When a check fails:

1. Return the failure message to the orchestrator
2. The orchestrator appends the failure message to the subagent's prompt and re-spawns it
3. Maximum 2 retries per subagent
4. After 2 failed retries, present the best output to the human reviewer with quality warnings attached
5. The human reviewer decides whether to accept, revise, or start over
