# CREATE SOMETHING: Experiment Tracking & Paper Writing Skill

Track agentic development experiments and write research papers documenting what works (and what doesn't) when building systems with Claude Code + Cloudflare.

## Quick Start

### For New Projects (Real-Time Tracking)

When starting a new experiment:

```
I want to build [project] using Claude Code. Let's track this as a CREATE SOMETHING experiment to document the process.
```

Claude will:
1. Initialize `.claude/experiments/[project-name]/` with tracking structure
2. Log prompts, errors, interventions as you work together
3. Track Cloudflare resources (Workers, Worker AI, Workflows, D1, KV, R2)
4. Generate a paper when you're ready to publish

### For Already-Deployed Projects (Retroactive)

When documenting existing work:

```
I want to write a CREATE SOMETHING paper about [deployed-project]. This is already in production, so we'll need to document it retroactively.
```

Claude will:
1. Analyze git history for timeline and changes
2. Fetch Claude Code Analytics API data (if built with Claude Code)
3. Fetch Cloudflare production analytics
4. Guide you through memory-based reconstruction
5. Generate a paper with clear notes about data limitations

## What Gets Tracked

### Real-Time Mode (Ideal)
- ✅ Every prompt and response
- ✅ Iteration-by-iteration progress
- ✅ Errors and resolution times
- ✅ Manual interventions
- ✅ Cloudflare resource usage (Worker AI, Workflows, D1, KV, R2)
- ✅ Precise costs from Claude Code + Cloudflare

### Retroactive Mode (Limited)
- ✅ Git commit history and timeline
- ✅ Cloudflare production analytics (if deployed)
- ✅ Claude Code Analytics API (if built with Claude Code)
- ✅ Architectural decisions from code analysis
- ✅ Memory-based intervention reconstruction
- ❌ Granular prompt logs (not captured)
- ❌ Precise iteration tracking (estimated)

## Examples

### Real-Time: New Project
See `examples/real-time-zoom-scraper.md` (coming soon)

### Retroactive: Already Deployed
See `examples/retroactive-workway-sync.md`
- Shows how to document syncflies.workway.co after deployment
- Demonstrates using Claude Code Analytics API for actual usage data
- Explains what can/can't be reconstructed

### Worker AI + Workflows Focus
See `examples/tracking-worker-ai-workflows.md`
- Specialized tracking for Cloudflare Worker AI inferences
- Workflow execution monitoring
- Combined cost analysis across AI + orchestration
- Production analytics retrieval

## File Structure

When tracking an experiment, you'll get:

```
.claude/experiments/[experiment-name]/
├── log.md              # Session notes, timeline, learnings
├── prompts.jsonl       # Every prompt/response (real-time only)
├── metrics.jsonl       # Timing, cost, performance data
├── errors.log          # Errors encountered and fixes
├── interventions.log   # Manual interventions required
├── cloudflare.jsonl    # Cloudflare-specific tracking
├── worker-ai.jsonl     # Worker AI inferences (if applicable)
├── workflows.jsonl     # Workflows executions (if applicable)
├── PAPER.md           # Generated paper (on request)
└── SUMMARY.md         # Quick metrics summary
```

## Paper Format

Papers follow CREATE SOMETHING brand guidelines:

- **Focus:** Building WITH AI agents (not just the system built)
- **Tone:** Direct, honest, research-oriented, no hype
- **Structure:** Hypothesis → Measurement → Results → Honest Assessment
- **Length:** 10-15 pages with real data
- **Audience:** Technical practitioners making decisions about AI-native development

### Key Sections:
1. **THE EXPERIMENT** - Problem, hypothesis, why it matters
2. **WHAT I MEASURED** - Success criteria, metrics tracked
3. **RESULTS** - Timeline, costs, performance data
4. **WHAT CLAUDE DID WELL** - Specific examples with code
5. **WHERE USER INTERVENED** - Issues, fixes, learnings
6. **HONEST ASSESSMENT** - What this proves/doesn't prove
7. **CLOUDFLARE ARCHITECTURE** - Platform decisions and outcomes
8. **REPRODUCIBILITY** - How others can replicate

## Cloudflare-Specific Tracking

### Automatically Tracks:
- **Workers:** Request counts, latency, errors
- **Worker AI:** Model usage, inference costs, token counts, latency
- **Workflows:** Executions, steps, duration, failures
- **D1:** Read/write operations, query performance
- **KV:** Operations, storage costs
- **R2:** Storage, bandwidth costs

### Integrations:
- Cloudflare Analytics API (production metrics)
- Cloudflare AI Logs API (Worker AI usage)
- Cloudflare Workflows API (execution history)

## Cost Tracking

### Development Costs:
- Claude Code token usage (from Analytics/Cost APIs)
- Time investment (sessions × duration)

### Runtime Costs:
- Cloudflare Workers (requests)
- Worker AI (inferences by model)
- Workflows (step executions)
- D1 (storage + operations)
- KV (operations)
- R2 (storage + bandwidth)

### ROI Calculation:
Total Project Cost vs. Estimated Manual Development Cost

## When to Document

### Start tracking when:
- Beginning a new experiment with Claude Code
- Testing a hypothesis about AI-native development
- Building on Cloudflare's stack (especially Worker AI/Workflows)
- Learning something you want to share

### Publish a paper when:
- Experiment completes with measurable results
- You learn something surprising (positive or negative)
- Hypothesis is validated or disproven
- Data could help others make decisions

## Integration with Claude Code

This Skill is designed to work seamlessly with Claude Code:

1. **Automatic Discovery:** Claude will offer to use this Skill when appropriate
2. **Parallel Tracking:** Logging happens as you build (minimal overhead)
3. **API Integration:** Real usage/cost data from Claude Code Analytics API
4. **Bash-Native:** All tracking uses simple bash scripts and JSONL files

## Retroactive Documentation: What's Possible?

### You CAN reconstruct:
✅ **Architectural decisions** - From code analysis
✅ **Timeline** - From git commit history
✅ **Costs** - From Claude Code + Cloudflare Analytics APIs (if available)
✅ **Production metrics** - From Cloudflare production analytics
✅ **Key learnings** - From memory + code inspection
✅ **Intervention moments** - From memory (approximate)

### You CANNOT reconstruct:
❌ **Iteration-by-iteration prompts** - No real-time logging
❌ **Precise error counts** - Not tracked without logging
❌ **Exact intervention times** - Memory-based estimates only
❌ **Granular cost attribution** - Can only estimate based on totals

### Paper Quality:
- **Real-time tracking:** High confidence, precise data
- **Retroactive documentation:** Lower confidence, acknowledged limitations, still valuable

## Tips for Great Papers

1. **Be honest about failures** - Failed experiments are valuable data
2. **Show actual code** - Reference specific files and line numbers
3. **Quantify everything** - Time, cost, iterations, errors
4. **Compare to baseline** - Manual development estimates for ROI
5. **Acknowledge limitations** - Especially for retroactive docs
6. **Focus on collaboration** - How you worked WITH Claude Code
7. **Make it reproducible** - Starting prompt + prerequisites

## Questions?

This Skill handles:
- "Track this experiment for CREATE SOMETHING"
- "Generate a paper about [project]"
- "Document [deployed-project] retroactively"
- "What were the costs for this experiment?"
- "How much time did we save building this way?"

Claude will recognize these patterns and invoke the Skill automatically.

---

**Version:** 1.0
**Compatible with:** Claude Code (Sonnet 4+)
**Platforms:** Cloudflare Workers, Worker AI, Workflows, D1, KV, R2
**Documentation Mode:** Real-time & Retroactive
