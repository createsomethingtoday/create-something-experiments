---
name: create-something-experiments
description: |
  Track agentic development experiments and write research papers in CREATE SOMETHING format.
  Use when building systems with Claude Code, testing hypotheses about AI-native development,
  documenting experiments (real-time or retroactive), measuring productivity, tracking Cloudflare
  deployments (Workers, Worker AI, Workflows, D1, KV, R2), or writing technical papers about
  building with LLMs. Automatically sets up experiment tracking, logs metrics, and generates
  papers from collected data.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

# CREATE SOMETHING: Experiment Tracking & Paper Writing

Track experiments building systems with Claude Code + Cloudflare and write research papers documenting what works (and what doesn't) when building agentically.

## When to Use This Skill

Use this Skill when the user:
- Starts a new project/experiment with Claude Code (real-time tracking)
- Wants to document an already-deployed project (retroactive documentation)
- Asks to track an experiment or document their work
- Wants to measure productivity or ROI of AI-native development
- Needs to write a paper about their experience building with Claude
- Asks about experiment metrics, costs, or performance
- Wants to document learnings from agentic development
- Mentions Cloudflare Worker AI or Workflows usage

## Core Principles

**CREATE SOMETHING brand positioning:**
- Focus: Building systems with AI agents (not just automation)
- Tone: Direct, honest, research-oriented, no hype
- Purpose: Real data from building real systems with AI
- Format: Experiments with testable hypotheses, not blog posts

**What to document:**
- The experience of building WITH Claude Code (not just the system built)
- Time comparisons (AI vs manual)
- Cost analysis (tokens vs developer hours)
- What Claude did well vs where human intervention was needed
- Honest assessment of when this approach works/doesn't work

## Two Tracking Modes

### Mode 1: Real-Time Tracking (Ideal)

For new projects, set up tracking from the start to capture:
- Every prompt and response
- Iteration-by-iteration progress
- Precise error logs and resolution times
- Exact intervention moments
- Granular cost data

**When to use:** Starting a new experiment with Claude Code

### Mode 2: Retroactive Documentation (Limited)

For already-deployed projects, reconstruct what you can:
- Architectural decisions from code/git history
- Cloudflare analytics (costs, usage, performance)
- Claude Code Analytics API data (if project used Claude Code)
- Worker AI inference logs (if used)
- Workflows execution history (if used)
- Memory-based timeline and key learnings

**When to use:** Documenting completed projects

**Limitations:**
- No granular prompt logs
- Approximate time/cost estimates
- Missing specific error/intervention details
- Less precise iteration tracking

## Initialization

When the user starts tracking (real-time or retroactive), create this structure:

```bash
.claude/experiments/[experiment-name]/
├── log.md              # Session notes, iteration tracking
├── prompts.jsonl       # Every prompt/response (real-time only)
├── metrics.jsonl       # Timing, cost, performance data
├── errors.log          # Errors and fixes (JSONL format)
├── interventions.log   # Manual interventions (JSONL format)
├── cloudflare.jsonl    # Cloudflare-specific tracking
├── PAPER.md           # Generated paper (created on request)
└── SUMMARY.md         # Quick metrics summary
```

For retroactive projects, some files may be incomplete or estimated.

## Real-Time Tracking Workflow

### Step 1: Initialize Experiment

When starting a new project:

```bash
#!/bin/bash
# Auto-run at project start
EXPERIMENT_NAME="zoom-transcript-scraper"
EXPERIMENT_DIR=".claude/experiments/${EXPERIMENT_NAME}"

mkdir -p "$EXPERIMENT_DIR"

# Create log.md with template
cat > "$EXPERIMENT_DIR/log.md" << 'EOF'
# Experiment: [Name]

## Hypothesis
[What are we testing about AI-native development?]

## Success Criteria
- [ ] Criterion 1
- [ ] Criterion 2

## Metrics to Track
- Time: AI vs manual estimate
- Cost: Token usage vs developer hourly rate
- Quality: Test coverage, error rate
- Interventions: Manual fixes needed

## Session Log
### Session 1: [Date/Time]
**Duration:** [X minutes]
**Goal:** [What we're building]
**Progress:** [What got done]
**Blockers:** [Issues encountered]
**Next:** [What's next]

EOF

# Initialize JSONL files
echo '{"experiment":"'$EXPERIMENT_NAME'","started":"'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'","hypothesis":"[HYPOTHESIS]"}' > "$EXPERIMENT_DIR/metrics.jsonl"
touch "$EXPERIMENT_DIR/prompts.jsonl"
touch "$EXPERIMENT_DIR/errors.log"
touch "$EXPERIMENT_DIR/interventions.log"
touch "$EXPERIMENT_DIR/cloudflare.jsonl"

echo "Experiment tracking initialized: $EXPERIMENT_DIR"
```

### Step 2: Log During Work

**After each iteration:**

```bash
# Log prompt and response
echo '{"iteration":1,"timestamp":"'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'","prompt":"Build a Zoom transcript scraper","response":"I'll create a serverless scraper using Cloudflare Browser Rendering...","files_created":["src/scraper.ts"],"files_modified":[],"decisions":["Use Workflows over Workers for long-running browser sessions"],"time_spent":"20min"}' >> .claude/experiments/[name]/prompts.jsonl
```

**When errors occur:**

```bash
# Log error
echo '{"iteration":5,"timestamp":"'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'","error_type":"timeout","error_message":"Browser automation timeout waiting for selector .transcript-text","context":"scraping Zoom transcript page","my_action":"inspected page, found selector changed to .meeting-transcript","fix_time":"2 min","fix_prompts":1,"learning":"web scraping is brittle, need fallback selectors"}' >> .claude/experiments/[name]/errors.log
```

**When user intervenes:**

```bash
# Log intervention
echo '{"iteration":12,"timestamp":"'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'","reason":"needed to extract cookies manually","what_user_did":"used browser DevTools to export cookies","why_needed":"no programmatic way to auth with Zoom web interface","could_ai_have_done_this":"no - security sensitive","time_spent":"5 min"}' >> .claude/experiments/[name]/interventions.log
```

**Cloudflare-specific tracking:**

```bash
# Log Cloudflare resource usage
echo '{"timestamp":"'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'","resource":"worker_ai","model":"@cf/meta/llama-3.1-8b-instruct","operation":"embedding_generation","tokens":1500,"cost_estimate":"$0.0015","latency_ms":450}' >> .claude/experiments/[name]/cloudflare.jsonl

echo '{"timestamp":"'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'","resource":"workflows","workflow_name":"zoom_scraper","execution_id":"abc123","duration_ms":45000,"steps":8,"cost_estimate":"$0.02"}' >> .claude/experiments/[name]/cloudflare.jsonl
```

### Step 3: Generate Paper

When ready to publish:

```bash
#!/bin/bash
# Generate paper from collected data
EXPERIMENT_DIR=".claude/experiments/[name]"

# Calculate metrics
TOTAL_ITERATIONS=$(wc -l < "$EXPERIMENT_DIR/prompts.jsonl")
TOTAL_ERRORS=$(wc -l < "$EXPERIMENT_DIR/errors.log")
TOTAL_INTERVENTIONS=$(wc -l < "$EXPERIMENT_DIR/interventions.log")

# Extract costs from Cloudflare logs
CLOUDFLARE_COST=$(grep -o '"cost_estimate":"[^"]*"' "$EXPERIMENT_DIR/cloudflare.jsonl" | cut -d'"' -f4 | sed 's/\$//' | awk '{s+=$1} END {printf "%.2f", s}')

# Generate PAPER.md with template and data
# [Template included below]
```

## Retroactive Documentation Workflow

### Step 1: Gather Available Data

For already-deployed projects:

```bash
#!/bin/bash
EXPERIMENT_NAME="existing-project"
EXPERIMENT_DIR=".claude/experiments/${EXPERIMENT_NAME}"
PROJECT_DIR="/path/to/deployed/project"

mkdir -p "$EXPERIMENT_DIR"

# Analyze git history
cd "$PROJECT_DIR"
git log --pretty=format:'{"commit":"%H","date":"%ai","message":"%s","files":%f}' --numstat > "$EXPERIMENT_DIR/git-history.jsonl"

# Count total commits, files changed
TOTAL_COMMITS=$(git rev-list --count HEAD)
FILES_CHANGED=$(git diff --name-only $(git rev-list --max-parents=0 HEAD) HEAD | wc -l)

# Estimate timeline
FIRST_COMMIT=$(git log --reverse --pretty=format:'%ai' | head -1)
LAST_COMMIT=$(git log --pretty=format:'%ai' | head -1)

# Create retroactive log
cat > "$EXPERIMENT_DIR/log.md" << EOF
# Experiment: $EXPERIMENT_NAME (Retroactive Documentation)

**Note:** This is retroactive documentation. Some metrics are estimates based on available data.

## Project Overview
- **Deployed:** Yes
- **Timeline:** $FIRST_COMMIT to $LAST_COMMIT
- **Commits:** $TOTAL_COMMITS
- **Files Modified:** $FILES_CHANGED

## Available Data Sources
- [x] Git commit history
- [x] Current codebase
- [ ] Claude Code Analytics API (check if available)
- [ ] Cloudflare Analytics (costs, usage)
- [ ] Cloudflare Worker AI logs (if used)
- [ ] Cloudflare Workflows execution history (if used)
- [x] Memory/notes

## What We're Reconstructing
[Describe the experiment/project retrospectively]

## Key Learnings
[Document what you learned building this]

EOF

echo "Retroactive tracking initialized: $EXPERIMENT_DIR"
```

### Step 2: Fetch Cloudflare Analytics

If project is deployed on Cloudflare:

```bash
#!/bin/bash
# Fetch Worker AI usage (if applicable)
# Note: Requires Cloudflare API credentials

ACCOUNT_ID="your-account-id"
API_TOKEN="your-api-token"

# Get Worker AI inference logs
curl -X GET "https://api.cloudflare.com/client/v4/accounts/${ACCOUNT_ID}/ai/runs/logs" \
  -H "Authorization: Bearer ${API_TOKEN}" \
  | jq '.result[] | select(.metadata.project_name=="'$EXPERIMENT_NAME'")' \
  > .claude/experiments/$EXPERIMENT_NAME/worker-ai-logs.json

# Get Workflows execution history
curl -X GET "https://api.cloudflare.com/client/v4/accounts/${ACCOUNT_ID}/workflows/executions" \
  -H "Authorization: Bearer ${API_TOKEN}" \
  | jq '.result[] | select(.workflow_name | contains("'$EXPERIMENT_NAME'"))' \
  > .claude/experiments/$EXPERIMENT_NAME/workflows-logs.json

# Calculate costs from logs
# [Parse logs and sum costs]
```

### Step 3: Interview Yourself

For retroactive docs, capture what you remember:

```bash
# Add to log.md
cat >> .claude/experiments/$EXPERIMENT_NAME/log.md << 'EOF'

## Memory-Based Reconstruction

### Major Challenges Faced
1. **[Challenge]:** [How you solved it]
2. **[Challenge]:** [How you solved it]

### Where Claude Code Excelled
- [Specific example with code location]
- [Specific example with code location]

### Where You Had to Intervene
- [Specific moment, why, time spent]
- [Specific moment, why, time spent]

### Architectural Decisions
1. **Why Cloudflare Workers instead of [alternative]:** [Reasoning]
2. **Why Worker AI for [use case]:** [Reasoning]
3. **Why Workflows instead of [alternative]:** [Reasoning]

### Estimated Timeline
- **Initial setup:** ~X hours
- **Core development:** ~X hours
- **Debugging/refinement:** ~X hours
- **Total:** ~X hours

### Cost Estimates
- **Claude Code token usage:** ~$X (from Analytics API if available)
- **Cloudflare costs:** ~$X (from Cloudflare Analytics)
- **Total project cost:** ~$X

### Would I Build This Way Again?
[Honest assessment]

EOF
```

### Step 4: Generate Retroactive Paper

Create paper acknowledging limitations:

```markdown
# Experiment #X: [Project Name] - Building with Claude Code (Retroactive Analysis)

**Note:** This paper documents an already-deployed project. Some metrics are estimates based on git history, Cloudflare analytics, and memory.

## THE EXPERIMENT

### What Was Built
[Description]

### Why This Matters
[Significance]

### Data Availability
- ✅ Git commit history
- ✅ Final codebase
- ✅ Cloudflare analytics
- ✅ Memory/notes
- ❌ Real-time prompt logs
- ❌ Precise iteration tracking

## WHAT I MEASURED (Retroactively)

### Available Metrics
- Total commits: [X]
- Development timeline: [X days]
- Cloudflare costs: $[X] (actual)
- Estimated development time: [X hours]

### Estimated Metrics
- Manual development time: ~[X hours] (estimate)
- Time saved: ~[X hours] (estimate)
- Cost vs traditional: ~[X]% savings (estimate)

[Rest of paper template...]
```

## Paper Template (Works for Both Modes)

```markdown
# Experiment #[Number]: [Testing Hypothesis] When Building with Claude Code

[If retroactive: Add note about data limitations]

## THE EXPERIMENT

### The Problem
[What real-world problem were you solving?]

### The Hypothesis
**I hypothesized that:** [Specific, testable claim about AI-native development]

Example: "Building a Cloudflare Worker with D1 database agentically would be 3x faster than traditional development, but require more debugging iterations."

### Why This Matters
[Why should anyone care about testing this?]

## WHAT I MEASURED

### Success Criteria
- [ ] [Specific, measurable outcome]
- [ ] [Specific, measurable outcome]
- [ ] [Specific, measurable outcome]

### Metrics Tracked
- **Time:** Claude sessions vs estimated manual development
- **Cost:** Token usage ($X) vs developer hourly rate ($Y)
- **Quality:** Test coverage, error rate, performance
- **Iterations:** Total prompts, error fixes, interventions
- **Cloudflare Resources:** Workers, Worker AI, Workflows, D1, KV, R2

## THE APPROACH

### Stack
- **Development:** Claude Code (Sonnet 4)
- **Runtime:** Cloudflare Workers
- **Data:** Cloudflare D1 (SQLite)
- **AI:** Cloudflare Worker AI (@cf/meta/llama-3.1-8b-instruct)
- **Orchestration:** Cloudflare Workflows
- **Storage:** Cloudflare KV, R2

### Initial Prompt
```
[Your exact starting prompt - or reconstructed for retroactive]
```

### How We Worked Together
[Describe the collaboration pattern between you and Claude Code]

## RESULTS: THE BUILD PROCESS

### Timeline

| Session | Date | Duration | What We Built | Blockers |
|---------|------|----------|---------------|----------|
| 1 | [date] | 45 min | Initial Worker setup | None |
| 2 | [date] | 30 min | D1 schema design | SQL syntax differences |
| 3 | [date] | 60 min | Worker AI integration | Rate limiting |
| ... | ... | ... | ... | ... |

**Total Development Time:** [X hours]
**Estimated Manual Development Time:** [Y hours]
**Time Savings:** [Z%]

### Success Criteria Results

- [✅/❌] **[Criterion 1]:** [Actual outcome]
- [✅/❌] **[Criterion 2]:** [Actual outcome]
- [✅/❌] **[Criterion 3]:** [Actual outcome]

### Performance Data

#### Cloudflare Metrics (Production)
- **Worker Requests:** [X] requests/day
- **Worker AI Inferences:** [X] inferences/day
- **Workflows Executions:** [X] executions/day
- **D1 Queries:** [X] queries/day
- **Average Latency:** [X] ms
- **Error Rate:** [X]%

#### Development Metrics
- **Total Iterations:** [X] prompts
- **Errors Encountered:** [X] errors
- **Manual Interventions:** [X] times
- **Files Created:** [X] files
- **Lines of Code:** [X] lines

### Cost Analysis

| Resource | Usage | Cost |
|----------|-------|------|
| Claude Code (Sonnet 4) | [X] tokens | $[X] |
| Cloudflare Workers | [X] requests | $[X] |
| Cloudflare Worker AI | [X] inferences | $[X] |
| Cloudflare Workflows | [X] executions | $[X] |
| Cloudflare D1 | [X] rows read/written | $[X] |
| Cloudflare KV | [X] operations | $[X] |
| **Total Project Cost** | | **$[X]** |
| **vs. Manual Development** | [Y] hours × $[rate]/hr | **$[Y]** |
| **ROI** | | **[Z]% savings** |

## WHAT I (CLAUDE CODE) DID WELL

### 1. [Capability]
**Example:** [Specific code snippet or file reference]

```typescript
// Example: Claude generated this Cloudflare Worker with D1 integration
export default {
  async fetch(request, env) {
    const { results } = await env.DB.prepare(
      "SELECT * FROM users WHERE email = ?"
    ).bind(email).all();

    return Response.json(results);
  }
}
```

**Why this worked:** [Explanation]

### 2. [Capability]
[Another specific example]

### 3. [Capability]
[Another specific example]

## WHERE USER INTERVENTION WAS NEEDED

### Issue #1: [Problem]
- **Iteration:** [X]
- **What happened:** [Description]
- **User intervention:** [What you had to do]
- **Time cost:** [X minutes]
- **Fix prompts:** [X]
- **Learning:** [What this revealed about AI-native development]

### Issue #2: [Problem]
[Same structure]

### Pattern Recognition
[What patterns emerged about when AI needs help?]

## HONEST ASSESSMENT

### What This Proves
✅ [Specific claim validated by data]
✅ [Specific claim validated by data]
✅ [Specific claim validated by data]

### What This Doesn't Prove
❌ [Overclaim avoided - what's still unknown]
❌ [Overclaim avoided - what's still unknown]

### When to Build This Way
**Use AI-native development when:**
- [Specific scenario]
- [Specific scenario]

**Don't use AI-native development when:**
- [Specific scenario]
- [Specific scenario]

### Hypothesis Outcome
[Was your hypothesis proven, disproven, or partially validated? What would you test next?]

## CLOUDFLARE ARCHITECTURE INSIGHTS

### Resources Used
- **Workers:** [How/why]
- **Worker AI:** [Models used, how/why]
- **Workflows:** [Orchestration patterns, how/why]
- **D1:** [Database design decisions]
- **KV:** [Caching strategy]
- **R2:** [Storage use cases]

### Architecture Decisions

#### Decision: [e.g., "Why Workflows instead of Durable Objects"]
**Reasoning:** [Your thinking]
**Claude's contribution:** [What Claude suggested]
**Outcome:** [Did it work?]

#### Decision: [e.g., "Why Worker AI embeddings instead of OpenAI"]
**Reasoning:** [Your thinking]
**Claude's contribution:** [What Claude suggested]
**Outcome:** [Did it work?]

### Deployment Experience
- **Initial deploy:** [Time, issues]
- **Iterations:** [How many redeploys]
- **Production stability:** [Issues encountered]

## REPRODUCIBILITY

### Prerequisites
- Cloudflare account with Workers enabled
- Claude Code (Sonnet 4 or better)
- [Other requirements]

### Starting Prompt
To replicate this experiment, use:
```
[Your exact initial prompt - or best reconstruction]
```

### Expected Challenges
Based on this experiment, you'll likely encounter:
1. [Challenge + how to handle it]
2. [Challenge + how to handle it]

### Data & Code
- **Repository:** [Link if public]
- **Experiment logs:** [Link to .claude/experiments/[name]/ if sharing]
- **Live demo:** [Link if applicable]

## CONCLUSION

### Key Takeaway
[One sentence: what's the most important thing you learned?]

### What I'd Do Differently Next Time
[Specific changes based on learnings]

### Next Experiment
[What question does this raise that you want to test next?]

---

**Experiment Date:** [Date range]
**Claude Code Version:** [Version]
**Total Cost:** $[X]
**Time Investment:** [X hours]
**Documentation Mode:** [Real-time / Retroactive]
```

## Cloudflare-Specific Tracking

### Worker AI Usage

Track all Worker AI inferences:

```jsonl
{"timestamp":"2025-01-14T10:30:00Z","resource":"worker_ai","model":"@cf/meta/llama-3.1-8b-instruct","operation":"text_generation","prompt_tokens":150,"completion_tokens":300,"cost_estimate":"$0.0045","latency_ms":650,"success":true}
{"timestamp":"2025-01-14T10:32:00Z","resource":"worker_ai","model":"@cf/baai/bge-base-en-v1.5","operation":"embedding","input_tokens":500,"cost_estimate":"$0.0005","latency_ms":120,"success":true}
```

### Workflows Execution

Track Workflow runs:

```jsonl
{"timestamp":"2025-01-14T10:35:00Z","resource":"workflows","workflow_name":"data_pipeline","execution_id":"exec_abc123","duration_ms":45000,"steps":12,"steps_failed":0,"cost_estimate":"$0.02","success":true}
{"timestamp":"2025-01-14T11:00:00Z","resource":"workflows","workflow_name":"data_pipeline","execution_id":"exec_def456","duration_ms":120000,"steps":12,"steps_failed":2,"cost_estimate":"$0.05","success":false,"error":"step_timeout"}
```

### D1 Database

Track database operations:

```jsonl
{"timestamp":"2025-01-14T10:40:00Z","resource":"d1","database":"prod_db","operation":"read","rows_read":1500,"query_time_ms":45,"cost_estimate":"$0.0001"}
{"timestamp":"2025-01-14T10:41:00Z","resource":"d1","database":"prod_db","operation":"write","rows_written":50,"query_time_ms":120,"cost_estimate":"$0.0003"}
```

### Cost Calculation

```bash
#!/bin/bash
# Calculate total Cloudflare costs for experiment

EXPERIMENT_DIR=".claude/experiments/[name]"

# Worker AI costs
WORKER_AI_COST=$(jq -s 'map(select(.resource=="worker_ai") | .cost_estimate | tonumber) | add' "$EXPERIMENT_DIR/cloudflare.jsonl")

# Workflows costs
WORKFLOWS_COST=$(jq -s 'map(select(.resource=="workflows") | .cost_estimate | tonumber) | add' "$EXPERIMENT_DIR/cloudflare.jsonl")

# D1 costs
D1_COST=$(jq -s 'map(select(.resource=="d1") | .cost_estimate | tonumber) | add' "$EXPERIMENT_DIR/cloudflare.jsonl")

# Total
TOTAL_CF_COST=$(echo "$WORKER_AI_COST + $WORKFLOWS_COST + $D1_COST" | bc)

echo "Cloudflare Costs:"
echo "  Worker AI: \$$WORKER_AI_COST"
echo "  Workflows: \$$WORKFLOWS_COST"
echo "  D1: \$$D1_COST"
echo "  Total: \$$TOTAL_CF_COST"
```

## Integration with Claude Code Analytics API

### Fetch Real Usage Data

```bash
#!/bin/bash
# Fetch actual Claude Code usage for this experiment
# Requires ANTHROPIC_API_KEY environment variable

ORGANIZATION_ID="your-org-id"
START_DATE="2025-01-14"
END_DATE="2025-01-15"

# Get usage metrics
curl -X GET "https://api.anthropic.com/v1/organizations/${ORGANIZATION_ID}/usage/code?start_date=${START_DATE}&end_date=${END_DATE}" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  | jq '.data[] | {
      date: .date,
      sessions: .sessions,
      accepted_lines: .code_generation.accepted_lines,
      rejected_lines: .code_generation.rejected_lines,
      acceptance_rate: (.code_generation.accepted_lines / (.code_generation.accepted_lines + .code_generation.rejected_lines) * 100),
      tool_uses: .tool_uses.total,
      commits: .repository_activity.commits,
      pull_requests: .repository_activity.pull_requests
    }' \
  > .claude/experiments/[name]/claude-code-analytics.json

# Get cost data
curl -X GET "https://api.anthropic.com/v1/organizations/${ORGANIZATION_ID}/usage/costs?start_date=${START_DATE}&end_date=${END_DATE}" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  | jq '.data[] | select(.model_name | contains("claude-sonnet")) | {
      date: .date,
      model: .model_name,
      input_tokens: .input_tokens,
      output_tokens: .output_tokens,
      cost: .total_cost
    }' \
  > .claude/experiments/[name]/claude-code-costs.json

echo "Claude Code analytics fetched"
```

## Quick Commands

### Start New Experiment
```bash
claude-experiments init [experiment-name] "[hypothesis]"
```

### Log Iteration
```bash
claude-experiments log-prompt "[prompt]" "[response summary]" --files-created "file1.ts,file2.ts"
```

### Log Error
```bash
claude-experiments log-error "[error message]" --context "[what you were doing]" --fix "[how you fixed it]"
```

### Log Intervention
```bash
claude-experiments log-intervention "[what you had to do manually]" --reason "[why AI couldn't do it]"
```

### Generate Paper
```bash
claude-experiments generate-paper [experiment-name]
```

### Generate Summary
```bash
claude-experiments summary [experiment-name]
```

### Start Retroactive Documentation
```bash
claude-experiments init-retroactive [experiment-name] --project-path "/path/to/deployed/project"
```

## Example: Real-Time Experiment

See `.claude/skills/create-something-experiments/examples/real-time-zoom-scraper.md`

## Example: Retroactive Documentation

See `.claude/skills/create-something-experiments/examples/retroactive-workway-sync.md`

## Tips for Great Papers

1. **Be honest about failures** - Failed experiments are valuable data
2. **Show actual code** - Don't just describe, show the generated code with file references
3. **Quantify everything** - Time, cost, iterations, errors - numbers matter
4. **Compare to baseline** - How long would this take manually? What's the ROI?
5. **Acknowledge limitations** - Especially for retroactive docs
6. **Focus on the collaboration** - Not just what was built, but how you built it with AI
7. **Make it reproducible** - Others should be able to replicate your experiment

## Common Pitfalls

- **Don't fake metrics** - If retroactive, acknowledge estimates as estimates
- **Don't overclaim** - "This approach works for X" ≠ "This approach always works"
- **Don't ignore context** - Your expertise level affects results
- **Don't skip failures** - Errors and interventions are the most valuable data
- **Don't forget Cloudflare costs** - Worker AI and Workflows costs add up

## When to Publish

Publish a paper when you:
- Complete an experiment with measurable results
- Learn something surprising (positive or negative)
- Validate or disprove a hypothesis about AI-native development
- Build something using Cloudflare's stack in a novel way
- Have data that could help others make decisions about agentic development

---

This Skill helps you systematically document your agentic development journey on CREATE SOMETHING, whether tracking in real-time or reconstructing past projects.
