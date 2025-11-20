# CREATE SOMETHING Experiments

> Systematic experiment tracking for AI-native development with Claude Code + Cloudflare

Transform "I built X with AI" (anecdote) into "Here's the data from building X with AI" (science).

## What This Is

A **Claude Code Skill** + **research methodology** for documenting experiments when building systems agentically with LLMs. Track prompts, errors, costs, and interventions in real-time—then generate reproducible research papers from the collected data.

This is the methodology behind [CREATE SOMETHING](https://createsomething.io), a platform documenting what works (and what doesn't) when building production systems with AI agents.

## Features

- **Automated Tracking** - Hooks capture prompts, errors, interventions automatically
- **Three Tracking Modes** - Real-time, mid-flight, or retroactive documentation
- **Cost Analysis** - Real costs from Claude Code Analytics API + Cloudflare billing
- **Paper Generation** - Transform tracking data into research papers
- **Cloudflare-Specific** - Built for Workers, Worker AI, Workflows, D1, KV, R2
- **Reproducible** - Others can verify your results and replicate experiments

## Quick Start

### 1. Copy the Skill to Your Project

```bash
# Copy .claude directory to your project root
cp -r .claude /path/to/your/project/
```

### 2. Start Building with Claude Code

```bash
cd /path/to/your/project
# Open in Claude Code
```

### 3. Enable Tracking

Just say to Claude:

```
Let's track this as a CREATE SOMETHING experiment
```

Claude will automatically:
- Initialize `.claude/experiments/[project-name]/`
- Log all prompts and responses
- Track errors and resolution times
- Document manual interventions
- Calculate costs from APIs

### 4. Generate a Paper When Done

```
Generate a CREATE SOMETHING paper from this experiment
```

Claude will create a complete research paper in the CREATE SOMETHING format with all tracked data.

## Three Tracking Modes

### Real-Time Tracking (Ideal)

Start tracking from day one for complete data:

```
I want to build [project] using Claude Code + Cloudflare.
Let's track this as CREATE SOMETHING Experiment #X.
```

**Data quality:** High confidence, precise metrics
**Use when:** Starting new experiments

### Mid-Flight Tracking (Practical)

Start tracking on in-progress projects:

```
I'm building [project] and want to start tracking this
experiment now. We're about 40% complete.
```

**Data quality:** Mixed (estimates for past work, precise for future)
**Use when:** You realize an active project is experiment-worthy

### Retroactive Documentation (Still Valuable)

Document already-deployed projects:

```
Let's write a CREATE SOMETHING paper about [deployed-project]
retroactively. It's already in production.
```

**Data quality:** Lower confidence, acknowledged limitations
**Use when:** Documenting completed work with production data

## What Gets Tracked

Every experiment captures:

| Metric | Description | Example |
|--------|-------------|---------|
| **Prompts** | Real-time logging via hooks | 47 iterations logged |
| **Errors** | Precise counts + resolution times | 23 errors, avg fix: 8 min |
| **Costs** | Token usage + infrastructure | $18.50 Claude + $8.30 Cloudflare |
| **Interventions** | When AI needed human help | 12 manual fixes documented |
| **Time** | Session duration, not estimates | 26 hours actual vs 120 estimated |
| **Architecture** | Decisions made, alternatives considered | Workflows over Workers (why) |

## Example Experiment

**[Zoom Transcript Automation](experiments/zoom-transcript-automation/)**

Building browser automation with Claude Code + Cloudflare Workers:

- **Time:** 26 hours (vs 120 hours estimated manually)
- **Savings:** 78% time reduction
- **Errors:** 47 errors encountered
- **Interventions:** 12 manual fixes required
- **Cost:** $26.80 development + $8.30/month runtime
- **Outcome:** Production system with 99.8% uptime

**Key learning:** AI-native development delivers massive time savings for well-defined automation, but web scraping brittleness still requires human debugging.

[Read the full paper →](experiments/zoom-transcript-automation/PAPER.md)

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

## Integration Opportunities

Enhance tracking with additional tools:

### 1. Automated Hooks ([claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery))
- Auto-log prompts via `UserPromptSubmit` hook
- Auto-log errors via `PostToolUse` hook
- Zero-effort tracking (no manual logging)

### 2. Real-Time Dashboard ([claude-code-hooks-multi-agent-observability](https://github.com/disler/claude-code-hooks-multi-agent-observability))
- Live experiment metrics
- Cost tracking visualization
- Error pattern analysis
- WebSocket real-time updates

### 3. Multi-Model Testing ([just-prompt](https://github.com/disler/just-prompt))
- Compare Claude vs GPT-4 vs Gemini
- Data on which models excel where
- "CEO and Board" consensus decisions

See [INTEGRATION_OPPORTUNITIES.md](.claude/skills/create-something-experiments/INTEGRATION_OPPORTUNITIES.md) for detailed guides.

## Paper Format

Papers follow CREATE SOMETHING structure:

1. **THE EXPERIMENT** - Problem, hypothesis, why it matters
2. **WHAT I MEASURED** - Success criteria, metrics tracked
3. **THE BUILD PROCESS** - Timeline, costs, performance
4. **WHAT CLAUDE DID WELL** - Specific examples with code
5. **WHERE I INTERVENED** - Issues, fixes, learnings
6. **HONEST ASSESSMENT** - What this proves/doesn't prove
7. **CLOUDFLARE ARCHITECTURE** - Platform decisions and outcomes
8. **REPRODUCIBILITY** - How others can replicate
9. **CONCLUSION** - Key takeaway + next experiment

## For Researchers

### Why This Methodology Matters

**Without tracking:**
- ❌ "I built X with AI" = anecdote
- ❌ No reproducibility
- ❌ Can't verify claims
- ❌ Just another AI blog

**With tracking:**
- ✅ "I built X: 26 hrs, $27, 78% savings" = data
- ✅ Others can replicate experiments
- ✅ Transparent methodology
- ✅ Scientific research platform

### Adopting for Your Work

This methodology works for:
- ✅ Testing AI-native development approaches
- ✅ Evaluating Claude Code vs manual development
- ✅ Comparing different AI models
- ✅ Building with Cloudflare's AI stack
- ✅ Research on agentic development

See [MID_FLIGHT_TRACKING.md](.claude/skills/create-something-experiments/MID_FLIGHT_TRACKING.md) for starting on active projects.

## Examples & Templates

### Example Experiments

- [Zoom Transcript Automation](experiments/zoom-transcript-automation/) - Browser automation with Cloudflare
- More coming soon...

### Templates

- [Real-Time Experiment Template](templates/real-time-experiment.md)
- [Retroactive Documentation Template](templates/retroactive-experiment.md)
- [Mid-Flight Tracking Template](templates/mid-flight-experiment.md)

## Documentation

- [SKILL.md](.claude/skills/create-something-experiments/SKILL.md) - Complete skill documentation
- [MID_FLIGHT_TRACKING.md](.claude/skills/create-something-experiments/MID_FLIGHT_TRACKING.md) - Start tracking on active projects
- [INTEGRATION_OPPORTUNITIES.md](.claude/skills/create-something-experiments/INTEGRATION_OPPORTUNITIES.md) - Enhance with hooks + dashboard
- [CONTRIBUTING.md](CONTRIBUTING.md) - How to contribute

## Tech Stack

- **Claude Code** - AI development partner (Sonnet 4+)
- **Cloudflare** - Workers, Worker AI, Workflows, D1, KV, R2, Browser Rendering
- **Analytics APIs** - Claude Code Analytics API, Cloudflare billing API
- **Tracking Format** - JSONL for easy parsing and analysis

## Learn More

- [CREATE SOMETHING](https://createsomething.io) - Platform using this methodology
- [Research Methodology](https://createsomething.io/methodology) - Full methodology explanation
- [Example Papers](https://createsomething.io/papers) - Published experiments

## Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for:
- How to submit experiments
- Paper format guidelines
- Code style
- Testing methodology

## License

MIT License - See [LICENSE](LICENSE) for details.

## Citation

If you use this methodology in your research, please cite:

```bibtex
@software{create_something_experiments,
  author = {Micah Johnson},
  title = {CREATE SOMETHING Experiments: Systematic Tracking for AI-Native Development},
  year = {2025},
  url = {https://github.com/createsomethingtoday/create-something-experiments},
  note = {Research methodology and Claude Code Skill for documenting agentic development experiments}
}
```

## Contact

- **Website:** [createsomething.io](https://createsomething.io)
- **Author:** [Micah Johnson](https://www.linkedin.com/in/micahryanjohnson/)
- **Issues:** [GitHub Issues](https://github.com/createsomethingtoday/create-something-experiments/issues)

---

**Status:** Active Development
**Version:** 1.0.0
**Last Updated:** January 2025
