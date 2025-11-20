# Mid-Flight Tracking: Starting Experiment Documentation on Active Projects

Guide for starting CREATE SOMETHING experiment tracking on projects you're currently building (but haven't finished yet).

## Scenario

You're 6 hours into building a project with Claude Code. You realize "this would make a great CREATE SOMETHING paper!" but haven't been tracking metrics.

**Question:** Can you start tracking now and still write a valuable paper?

**Answer:** YES, with caveats.

## What You CAN Capture from This Point Forward

### ✅ Full Tracking Going Forward

Once you initialize tracking mid-flight, you'll get complete data from that point on:

- Every prompt/response (from now until completion)
- Every error and resolution (future iterations)
- Every intervention (future manual fixes)
- Precise costs (from this point forward)
- Exact time spent (remaining sessions)

### ✅ Partial Reconstruction of Past Work

You can reconstruct some metrics from earlier work:

- **Git history** - commits, files changed, timeline
- **Claude Code Analytics API** - token usage for the entire project
- **Memory** - major decisions, big interventions, key learnings

### ❌ What You Can't Recover

- Iteration-by-iteration prompts from before tracking started
- Precise error counts from early sessions
- Exact intervention moments and resolution times from past work
- Session-by-session cost breakdown for completed work

## Mid-Flight Initialization Workflow

### Step 1: Initialize Tracking Right Now

```bash
# Stop and initialize experiment tracking
EXPERIMENT_NAME="project-name-mid-flight"
EXPERIMENT_DIR=".claude/experiments/${EXPERIMENT_NAME}"

mkdir -p "$EXPERIMENT_DIR"

# Create log with mid-flight note
cat > "$EXPERIMENT_DIR/log.md" << 'EOF'
# Experiment: [Project Name] (Mid-Flight Tracking)

**⚠️ Note:** Tracking started mid-project. Earlier metrics are estimates based on git history and memory.

## Project Overview
- **Status:** In progress (tracking started on [DATE])
- **Progress at tracking start:** ~[X]% complete
- **Estimated time before tracking:** ~[X] hours
- **Estimated time remaining:** ~[X] hours

## What We've Built So Far (Pre-Tracking)

### Completed Features
1. [Feature 1] - [brief description]
2. [Feature 2] - [brief description]

### Key Decisions Made
1. **[Decision]:** [Why we made it]
2. **[Decision]:** [Why we made it]

### Major Interventions (From Memory)
1. **[Issue]:** [What broke, how fixed, ~time spent]
2. **[Issue]:** [What broke, how fixed, ~time spent]

## From This Point Forward

Tracking all prompts, errors, interventions, and costs in real-time.

## Session Log

### [Session X]: [DATE] - TRACKING STARTS HERE
**Duration:** [X minutes]
**Goal:** [What we're building]
**Progress:** [What got done]
**Next:** [What's next]
EOF

# Initialize JSONL files for future tracking
echo "{\"experiment\":\"$EXPERIMENT_NAME\",\"started_mid_flight\":true,\"tracking_start\":\"$(date -u +"%Y-%m-%dT%H:%M:%SZ")\"}" > "$EXPERIMENT_DIR/metrics.jsonl"
touch "$EXPERIMENT_DIR/prompts.jsonl"
touch "$EXPERIMENT_DIR/errors.log"
touch "$EXPERIMENT_DIR/interventions.log"
touch "$EXPERIMENT_DIR/cloudflare.jsonl"

echo "✅ Mid-flight tracking initialized: $EXPERIMENT_DIR"
echo "ℹ️  All future iterations will be tracked automatically"
```

### Step 2: Reconstruct What You Can from Past Work

```bash
#!/bin/bash
# Reconstruct past work metrics

EXPERIMENT_DIR=".claude/experiments/project-name-mid-flight"

# Analyze git history for work done so far
echo "## Git History Analysis" >> "$EXPERIMENT_DIR/log.md"
echo "" >> "$EXPERIMENT_DIR/log.md"

# Count commits before tracking started
COMMITS_BEFORE=$(git log --since="[START_DATE]" --until="$(date -u +"%Y-%m-%dT%H:%M:%SZ")" --oneline | wc -l)
echo "- **Commits before tracking:** $COMMITS_BEFORE" >> "$EXPERIMENT_DIR/log.md"

# List files changed
FILES_CHANGED=$(git diff --name-only $(git rev-list --max-parents=0 HEAD) HEAD | wc -l)
echo "- **Files modified:** $FILES_CHANGED" >> "$EXPERIMENT_DIR/log.md"

# Get timeline
FIRST_COMMIT=$(git log --reverse --pretty=format:'%ai' | head -1)
echo "- **Started:** $FIRST_COMMIT" >> "$EXPERIMENT_DIR/log.md"
echo "- **Tracking enabled:** $(date)" >> "$EXPERIMENT_DIR/log.md"
echo "" >> "$EXPERIMENT_DIR/log.md"
```

### Step 3: Fetch Historical Claude Code Usage (If Available)

```bash
#!/bin/bash
# Get Claude Code analytics for project period

ORGANIZATION_ID="your-org-id"
PROJECT_START="2025-01-10"  # When you started the project
TRACKING_START=$(date +%Y-%m-%d)  # Today

curl -X GET "https://api.anthropic.com/v1/organizations/${ORGANIZATION_ID}/usage/code?start_date=${PROJECT_START}&end_date=${TRACKING_START}" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  > ".claude/experiments/project-name-mid-flight/pre-tracking-claude-usage.json"

# Get costs
curl -X GET "https://api.anthropic.com/v1/organizations/${ORGANIZATION_ID}/usage/costs?start_date=${PROJECT_START}&end_date=${TRACKING_START}" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  > ".claude/experiments/project-name-mid-flight/pre-tracking-claude-costs.json"

echo "✅ Historical Claude Code data retrieved"
```

### Step 4: Document Current State

```bash
# Take snapshot of current codebase state
cat >> "$EXPERIMENT_DIR/log.md" << 'EOF'

## Current State Snapshot (When Tracking Started)

### Architecture Decisions Made
1. **[Decision]:** Using Cloudflare Workers instead of [alternative]
   - **Reasoning:** [Why]
   - **Outcome so far:** [How it's working]

2. **[Decision]:** Using D1 instead of [alternative]
   - **Reasoning:** [Why]
   - **Outcome so far:** [How it's working]

### What's Working Well
- [Feature/Pattern that Claude generated successfully]
- [Feature/Pattern that Claude generated successfully]

### What Needed Manual Intervention
- [Issue that required human debugging]
- [Issue that required human debugging]

### Remaining Work
- [ ] [Feature to build]
- [ ] [Feature to build]
- [ ] [Testing/deployment]
EOF
```

### Step 5: Continue Building with Full Tracking

From this point forward, track everything as if it were a new experiment:

```bash
# Example: Log next prompt
echo '{"iteration":1,"timestamp":"'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'","prompt":"Add error handling to transcript extraction","note":"First tracked prompt - project ~60% complete at this point"}' >> .claude/experiments/project-name-mid-flight/prompts.jsonl
```

## Paper Structure for Mid-Flight Experiments

When writing the paper, acknowledge tracking started mid-way:

```markdown
# Experiment #X: [Project Name] - Building with Claude Code

**⚠️ Tracking Methodology:** This experiment began mid-project. Pre-tracking metrics (first 6 hours) are estimates based on git history, Claude Code Analytics API, and memory. All metrics from hour 6 onward are precise real-time tracking.

## THE EXPERIMENT

### What Was Built
[Full description of the complete project]

### Data Sources
- ✅ **Hours 0-6 (before tracking):** Git history, Claude Code Analytics API, memory-based estimates
- ✅ **Hours 6-20 (tracked):** Real-time prompts, errors, interventions, costs
- ✅ **Cloudflare costs:** Actual production data (entire project)

## WHAT I MEASURED

### Pre-Tracking Phase (Hours 0-6)
**Available Metrics:**
- Estimated time: ~6 hours (based on git commit timestamps)
- Claude Code tokens: 450K tokens / $6.75 (from Analytics API)
- Files created: 12 files (from git history)
- Major interventions: 3 (from memory)

**Missing Metrics:**
- Iteration-by-iteration prompts
- Precise error counts
- Exact intervention timing

### Tracked Phase (Hours 6-20)
**Complete Metrics:**
- Prompts: 47 iterations logged
- Errors: 23 errors with resolution times
- Interventions: 5 manual fixes (precise timing)
- Costs: $11.75 Claude Code + $2.30 Cloudflare

### Combined Project Totals
- **Total time:** ~20 hours (6 estimated + 14 tracked)
- **Total cost:** $20.80 Claude + Cloudflare
- **Total interventions:** 8 (3 estimated + 5 tracked)

## HONEST ASSESSMENT

### Tracking Limitations
This experiment's early phase wasn't tracked in real-time, limiting precision for:
- Early error patterns
- Initial iteration speed
- First intervention moments

**However**, the tracked phase (hours 6-20) provides high-confidence data on:
- How Claude handles increasing complexity
- Late-stage debugging patterns
- Integration and deployment challenges

**Conclusion:** Mid-flight tracking still produces valuable data, especially for understanding the latter half of development where complexity peaks.
```

## When Mid-Flight Tracking is Valuable

### ✅ Good Scenarios for Mid-Flight Tracking

**Start tracking when:**
- You're <50% done with the project
- The remaining work is complex/interesting
- You can clearly document what's been done so far
- You want data on debugging/refinement/deployment phases

**Example:**
> "I've built the core feature in 6 hours. The next 10 hours will be error handling, edge cases, and deployment. That's where AI-native development gets tested—let's track it!"

### ⚠️ Less Valuable Scenarios

**Don't bother if:**
- Project is >90% complete
- Only trivial work remains (documentation, minor tweaks)
- You can't remember major decisions/interventions from earlier work

## Workflow: Starting Mid-Flight Tracking Right Now

If you're in a Claude Code session and want to start tracking immediately:

```
I'm currently building [project] and want to track this as a CREATE SOMETHING experiment from this point forward.

We're about [X]% complete. Can you:
1. Initialize mid-flight experiment tracking
2. Document what we've built so far
3. Start logging all future prompts, errors, and costs

Let's make this Experiment #[X]: [Project Name]
```

Claude Code will:
1. Create `.claude/experiments/[name]/` structure
2. Initialize tracking files
3. Document current state
4. Start logging all future work
5. Fetch historical Claude Code usage from Analytics API

## Example: Mid-Flight Tracking Success Story

**Scenario:** You're 8 hours into building a Notion API integration with Claude Code. You realize "this would make a great experiment!" but haven't been tracking.

**Action:** Initialize mid-flight tracking immediately.

**Result:**
- **Pre-tracking (hours 0-8):** Estimated 8 hours, ~$12 in tokens (from Analytics API), 3 major interventions (from memory)
- **Tracked (hours 8-16):** Precise data on 38 prompts, 15 errors, 4 interventions, $8.50 costs
- **Paper quality:** Still valuable! The tracked phase captured the most interesting part—debugging production edge cases

**Key insight:** The latter half of development (debugging, edge cases, deployment) is often MORE interesting than initial setup. Mid-flight tracking can capture the hardest problems.

## Best Practices

1. **Be honest about limitations** - Clearly mark estimated vs precise data in your paper
2. **Reconstruct what you can** - Use git + Claude Analytics API + memory
3. **Focus on tracked phase** - The real-time data is your strongest evidence
4. **Document "why tracking started mid-flight"** - What made you realize this was experiment-worthy?

## Summary

**Can you write a valuable CREATE SOMETHING paper from mid-flight tracking?**

**YES** - especially if:
- You're <50% done when tracking starts
- You document what's been done so far
- You acknowledge tracking limitations in the paper
- The tracked phase captures complex/interesting work

**The tracked portion will have high-confidence data**, and that's often the most interesting part (debugging, deployment, edge cases) where AI-native development faces its hardest tests.

**Start tracking NOW** if you think "this could be a paper"—don't wait!
