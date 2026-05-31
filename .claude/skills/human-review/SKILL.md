---
description: >
  Present agent output for human review. Use after every agent completes
  and passes quality checks. Presents key findings in readable format
  and offers three review options.
disable-model-invocation: true
---

# Human Review Skill

This skill defines how the orchestrator presents agent output to the human reviewer and handles their feedback. Use after every agent completes and passes quality checks.

## Presentation Format

Do NOT dump raw JSON to the user. Present a readable summary of the key findings.

### Structure of the Review Presentation

**1. Agent Header**
```
## Agent [N]: [Agent Name] - Review

Status: Complete | Quality checks: Passed
Scenario type: [scenario_type] | Scenario emphasis applied: [what was emphasized]
```

**2. Key Findings Summary**

Present the 3-5 most important findings in natural language. These should be the findings that matter most for downstream agents and for the overall GTM strategy.

For Agent 1 (Market Intelligence), structure as:
- Market state: [headline finding, market size, maturity stage]
- Competitive landscape: [number of competitors, key dynamic]
- Buying environment: [how buyers purchase, typical timeline]
- Key opportunity: [the macro trend or gap that creates the window]

For Agents 2-5, structure around the agent's primary framework:
- Agent 2: Lead with primary ICP segment and top negative indicator
- Agent 3: Lead with recommended GTM motion and top channel
- Agent 4: Lead with market narrative shift and primary positioning angle
- Agent 5: Lead with Month 1 priority actions and key success metrics

**3. Confidence Overview**

Summarize confidence ratings across sections:
```
Confidence: [N] sections rated High, [N] Medium, [N] Low
Lowest confidence areas: [list the Low-rated sections with brief reason]
```

**4. Gaps and Flags**

List any data gaps, discrepancies, or quality warnings:
```
Flags:
- [Gap or discrepancy description]
- [Any quality check warnings if the output passed on retry]
```

**5. Scenario-Specific Notes**

Note what scenario-specific emphasis was applied:
```
Scenario adaptation: Because this is a [scenario_type], extra depth was applied to [areas].
```

## Review Options

After presenting the summary, offer exactly three options:

```
How would you like to proceed?

1. **Approve** - Lock this output and proceed to Agent [N+1]
2. **Revise with feedback** - Tell me what to change and I will re-run this agent with your feedback
3. **Partial re-run** - Flag specific sections that need more work (the rest stays as-is)

Your choice (1/2/3):
```

### Handling Each Option

**Option 1: Approve**
- Mark the agent output as approved
- Save review status to `runs/<company-name>/review/agent<N>-review.md`:
  ```
  Status: Approved
  Reviewed: [timestamp]
  Notes: None
  ```
- Proceed to the next agent

**Option 2: Revise with feedback**
- Ask the user to describe their changes in natural language
- Save the feedback to the review file:
  ```
  Status: Revision requested
  Reviewed: [timestamp]
  Feedback: [user's feedback text]
  ```
- Re-spawn the subagent with the original inputs PLUS the user's feedback appended to the prompt
- The re-spawned agent sees: "REVISION FEEDBACK FROM REVIEWER: [feedback text]. Incorporate this feedback into your revised output."
- After the revised output returns, run quality checks again and present for review again
- There is no limit on revision cycles (the user decides when it is good enough)

**Option 3: Partial re-run**
- Ask the user to specify which sections need more work
- Save to review file:
  ```
  Status: Partial re-run requested
  Reviewed: [timestamp]
  Sections flagged: [list of sections]
  Feedback per section: [user's notes]
  ```
- Re-spawn the subagent with: original inputs, the current output (so it does not redo everything), and specific instructions to improve only the flagged sections
- The re-spawned agent sees: "PARTIAL REVISION: Your previous output is attached. Only revise these sections: [sections]. Keep everything else as-is. Reviewer feedback: [feedback]."
- Present the updated output for review again

## Error Handling: Hallucinated Data

The human reviewer is the last line of defense against hallucinated market data, competitor information, or funding numbers. When presenting the review:

- Highlight any data points where the confidence rating is Low
- If market sizing numbers look suspiciously round or convenient, note it
- If competitor profiles lack URLs or cite only the company's homepage, note it
- If GTM pattern inferences lack observable evidence, note it

The reviewer can check cited sources. If a source does not actually say what the agent claims, the reviewer should use Option 2 (Revise with feedback) and specify: "Source [URL] does not support the claim about [topic]. Remove or correct."

## Review File Format

Save review status to `runs/<company-name>/review/agent<N>-review.md` after each review interaction:

```markdown
# Agent [N]: [Agent Name] - Review Log

## Review 1
- Status: [Approved | Revision requested | Partial re-run requested]
- Timestamp: [ISO 8601]
- Reviewer feedback: [text or "None"]
- Sections flagged: [list or "None"]

## Review 2 (if revision occurred)
- Status: [Approved | Revision requested | Partial re-run requested]
- Timestamp: [ISO 8601]
- Reviewer feedback: [text or "None"]
- Changes made: [summary of what changed]
```

Keep appending to the review file. Do not overwrite previous review entries. This creates an audit trail of the review process.
