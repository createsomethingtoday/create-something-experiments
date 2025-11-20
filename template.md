# Experiment Template

This template defines the structure for CREATE SOMETHING experiments.

## Frontmatter (Required)

```markdown
---
id: unique-experiment-id
slug: unique-experiment-slug
title: "Experiment #N: Title of Your Experiment"
category: infrastructure | methodology | automation | ai-agents
description: |
  Single-sentence description (160 chars max for SEO).
  This appears in search results and social shares.

# Publishing
featured: true | false
published: true | false
is_hidden: false
archived: false
published_at: 2025-11-19T00:00:00.000Z

# Reading metrics
reading_time: 18  # minutes
difficulty_level: beginner | intermediate | advanced

# Technical focus (comma-separated)
technical_focus: cloudflare-workers, d1-database, ai-agents

# Implementation
implementation_time: "11 hours"
created_at: 2025-11-19T00:00:00.000Z
updated_at: 2025-11-19T00:00:00.000Z

# Excerpts
excerpt_short: "Brief one-liner summary for cards."
excerpt_long: |
  2-3 sentence summary with key metrics.
  Includes the "what" and the "why" of this experiment.
  Should entice readers to dive deeper.

# Metadata
meta_title: "Experiment Title | CREATE SOMETHING"
meta_description: "SEO-optimized description with keywords."
focus_keywords: keyword1, keyword2, keyword3

# Interactivity
interactive_demo_url: "https://createsomething.space/experiments/slug"  # Optional
is_executable: 0 | 1
terminal_commands: null  # JSON if applicable
code_lessons: null  # JSON if applicable

# ASCII art for visual identity
ascii_art: |
  ╔═══════════════════════════════════════╗
  ║  EXPERIMENT #N: TITLE                ║
  ║  ────────────────────────────────────  ║
  ║  One-line hook                        ║
  ║  Framework or approach                ║
  ║                                       ║
  ║  TIME: Xhrs | METRIC: Value          ║
  ║  OUTCOME: Result achieved            ║
  ╚═══════════════════════════════════════╝
---
```

## Markdown Content Structure

### 1. Problem / Context

What challenge or question prompted this experiment?

**Example:**
> Email was trapped in Gmail, but all our work lives in Notion. We needed a way to sync important emails into our project databases without manual copy-paste.

### 2. Hypothesis

What did you believe would work, and why?

**Example:**
> Hypothesis: Cloudflare Workers + Gmail API + Notion API could create a multi-user sync system in <15 hours (vs 25-30 manual development), saving 55-65% time.

### 3. Build Process

#### 3.1 Approach
What architecture/tools did you choose?

#### 3.2 Implementation Steps
Key phases of the build (numbered list).

#### 3.3 Challenges Encountered
What went wrong? What surprised you?

### 4. Results

#### 4.1 Quantitative Outcomes
- Time: X hours
- Cost: $X
- Performance: X metric
- Savings: X%

#### 4.2 Qualitative Outcomes
What did you learn? What changed your understanding?

### 5. Analysis

#### 5.1 What Worked
Key successes and why they worked.

#### 5.2 What Didn't Work
Failures and what they revealed.

#### 5.3 Hermeneutic Insights
How did understanding the **whole** (system architecture, philosophy) illuminate the **parts** (specific decisions)? How did working with the **parts** deepen understanding of the **whole**?

### 6. Code Examples (Optional)

```typescript
// Key implementation patterns
```

### 7. System Logic (Optional)

For complex systems, document:
- Database schema
- Business logic
- API patterns
- Scoring algorithms
- Cost calculations

See: `VIRALYTICS_SYSTEM_LOGIC.md` as reference.

### 8. Reproduction Guide

How can someone else reproduce this experiment?

**Prerequisites:**
- List requirements

**Steps:**
1. Step one
2. Step two
3. ...

**Expected Outcome:**
What success looks like.

### 9. Conclusions

#### 9.1 Hypothesis Validation
Was your hypothesis proven? Disproven? Partially validated?

#### 9.2 Transferable Knowledge
What principles from this experiment apply to OTHER domains?

#### 9.3 Next Steps
What questions did this experiment raise? What experiments should come next?

---

## Metadata Guidelines

### Category Taxonomy
- **infrastructure** — Cost optimization, system architecture, deployment
- **methodology** — Research methods, hermeneutics, frameworks
- **automation** — Workflow automation, integrations, sync systems
- **ai-agents** — AI-powered agents, LLM applications, intelligent automation

### Difficulty Levels
- **beginner** — Can follow along with basic knowledge
- **intermediate** — Requires familiarity with tools/concepts
- **advanced** — Assumes deep technical expertise

### Technical Focus (Tags)
Use kebab-case, comma-separated:
- `cloudflare-workers`
- `d1-database`
- `notion-api-2025`
- `oauth-2.0`
- `hermeneutic-analysis`
- etc.

---

## File Naming Convention

```
experiments/NN-kebab-case-title.md
```

**Examples:**
- `experiments/01-cloudflare-kv-quickstart.md`
- `experiments/03-kickstand-sustainability.md`
- `experiments/06-gmail-notion-sync.md`

**Why prefix with numbers?**
- Maintains chronological order
- Prevents naming conflicts
- Makes references clear ("See Experiment #3")

---

## Philosophy: "Weniger, aber besser"

Every experiment must justify its inclusion by answering:

1. **What essential question does this answer?**
2. **What transferable knowledge does this create?**
3. **How does this deepen understanding of AI-native development?**

If an experiment is "just a build log" without insights, it doesn't belong in this repository.

**Good experiments:**
- Reveal constraints
- Test hypotheses
- Produce transferable frameworks
- Document the hermeneutic circle

**Bad experiments:**
- Feature announcements
- Tutorial rewrites
- Generic "how-tos"
- Undifferentiated builds

---

## Usage

1. **Copy this template** when starting a new experiment
2. **Fill frontmatter** with accurate metadata
3. **Write content** following the 9-section structure
4. **Commit to experiments repo** with descriptive message
5. **Run sync script** to push to D1 database
6. **Verify on .io and .space**

---

**This template embodies the hermeneutic circle:** understanding the structure (whole) enables writing experiments (parts), and writing experiments deepens understanding of what makes a good experiment (whole).
