# Contributing to CREATE SOMETHING Experiments

Thank you for your interest in contributing to CREATE SOMETHING Experiments! This document provides guidelines for submitting experiments, improving the methodology, and enhancing the Claude Code Skill.

## How to Contribute

### 1. Submit an Experiment

Share your tracked experiments with the community:

**Requirements:**
- Real-time, mid-flight, or retroactive tracking using the Skill
- Complete tracking data (prompts, errors, costs, interventions)
- Cloudflare deployment (Workers, Worker AI, Workflows, D1, KV, or R2)
- Honest assessment of results (what worked AND what didn't)

**Submission Process:**

1. **Fork the repository**
2. **Add your experiment** to `.claude/skills/create-something-experiments/examples/`
   - File name: `your-experiment-name.md`
   - Follow the paper format (see templates)
3. **Include tracking data:**
   - Session logs
   - Error counts and resolution times
   - Cost breakdown (AI tokens + Cloudflare infrastructure)
   - Intervention documentation
4. **Submit a Pull Request** with:
   - Clear description of the experiment
   - Hypothesis and results summary
   - Data quality acknowledgment (real-time, mid-flight, or retroactive)

### 2. Improve the Methodology

Help refine the research approach:

**Areas for improvement:**
- Tracking automation (new hooks, logging enhancements)
- Cost calculation methods
- Paper generation templates
- Reproducibility guidelines

**Process:**
1. Open an issue describing the proposed improvement
2. Discuss with maintainers
3. Submit a PR with changes to `.claude/skills/create-something-experiments/SKILL.md`

### 3. Enhance the Skill

Contribute to the Claude Code Skill functionality:

**Types of enhancements:**
- New tracking modes
- Additional Cloudflare services (R2, Queues, etc.)
- Integration with other tools (hooks, dashboards)
- Performance optimizations

**Process:**
1. Test changes locally with Claude Code
2. Document new functionality in `SKILL.md`
3. Provide usage examples
4. Submit PR with clear description

## Paper Format Guidelines

All experiments should follow the CREATE SOMETHING paper structure:

### Required Sections

1. **THE EXPERIMENT**
   - Problem statement
   - Hypothesis
   - Why it matters

2. **WHAT I MEASURED**
   - Success criteria
   - Metrics tracked
   - Data sources

3. **THE BUILD PROCESS**
   - Timeline
   - Costs (AI + infrastructure)
   - Performance

4. **WHAT CLAUDE DID WELL**
   - Specific examples with code
   - Iteration counts
   - Successful patterns

5. **WHERE I INTERVENED**
   - Issues encountered
   - Manual fixes required
   - Learnings and insights

6. **HONEST ASSESSMENT**
   - What this proves
   - What this doesn't prove
   - Limitations acknowledged

7. **CLOUDFLARE ARCHITECTURE**
   - Platform decisions
   - Service choices
   - Outcomes

8. **REPRODUCIBILITY**
   - Starting prompt
   - Tracking logs location
   - Architecture decisions documented

9. **CONCLUSION**
   - Key takeaway (one sentence)
   - Next experiment hypothesis

### Data Quality Standards

**Real-Time Tracking:**
- Precise metrics from start to finish
- All prompts logged via hooks
- Exact error counts and times

**Mid-Flight Tracking:**
- Mixed data (estimates + real-time)
- Git history for past work
- Real-time logs for future work
- Acknowledge limitations

**Retroactive Documentation:**
- Lower confidence metrics
- Production data included
- Clear acknowledgment of missing data
- Still valuable for community

## Code Style

- Follow existing file structure
- Use clear, descriptive variable names
- Comment complex logic
- Test with Claude Code before submitting

## Testing Methodology

Before submitting a PR:

1. **Test the Skill** with Claude Code
2. **Verify tracking** - Run through complete experiment workflow
3. **Generate paper** - Ensure paper generation works
4. **Check reproducibility** - Can someone else follow your experiment?

## Review Process

1. **Initial Review:** Maintainers check format and completeness
2. **Technical Review:** Verify tracking data and methodology
3. **Community Feedback:** Open for discussion
4. **Merge:** Once approved, merged into main branch

## Questions?

- Open an issue for questions about the methodology
- Check [SKILL.md](.claude/skills/create-something-experiments/SKILL.md) for Skill documentation
- See [examples/](.claude/skills/create-something-experiments/examples/) for reference experiments

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for helping make AI-native development more transparent and reproducible!**
