# Experiment Pipeline: GitHub → D1

**Philosophy:** "Weniger, aber besser" (Less, but better)

---

## The Essential Pipeline

This repository is the **single source of truth** for CREATE SOMETHING experiments. The pipeline syncs experiments from here to the D1 database, which feeds both `.io` (theory) and `.space` (practice).

```
┌─────────────────────────────────────────┐
│  create-something-experiments (GitHub)  │
│  experiments/*.md                       │
│  THE SOURCE OF TRUTH (Vorhabe)         │
└─────────────────────────────────────────┘
                  │
                  │ Manual trigger:
                  │ npm run sync-experiments
                  ↓
        ┌──────────────────────┐
        │  sync-experiments.ts │
        │  ─────────────────   │
        │  1. Fetch from GH    │
        │  2. Parse markdown   │
        │  3. Upsert to D1    │
        └──────────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────┐
│  Cloudflare D1 (Shared Database)        │
│  papers table                           │
└─────────────────────────────────────────┘
         │                    │
         ↓                    ↓
  ┌─────────────┐      ┌──────────────┐
  │ .io         │      │ .space       │
  │ Full Papers │      │ Interactive  │
  └─────────────┘      └──────────────┘
```

---

## Quick Start

### 1. Write an Experiment

```bash
# Copy the template
cp template.md experiments/07-my-experiment.md

# Fill frontmatter + content
# (see template.md for structure)

# Commit to this repo
git add experiments/07-my-experiment.md
git commit -m "feat: add Experiment #7"
git push origin main
```

### 2. Sync to D1

```bash
cd ../create-something-svelte
npm run sync-experiments
```

### 3. Verify

- **Theory:** https://createsomething.io/experiments/my-experiment
- **Practice:** https://createsomething.space/experiments/my-experiment

---

## The Sync Script

**Location:** `create-something-svelte/scripts/sync-experiments.ts`

**What it does:**
1. Fetches all `.md` files from `experiments/` via GitHub API
2. Parses frontmatter (YAML metadata) + markdown content
3. Converts markdown → HTML using `marked`
4. Upserts to D1 `papers` table (INSERT OR REPLACE)
5. Reports success/failures

**What it doesn't do:**
- ❌ Automatic deployments
- ❌ Complex validation pipelines
- ❌ Git hooks or watchers
- ❌ Multi-stage transformations

**Why?** "Weniger, aber besser" - one script, one purpose, manual trigger.

---

## Usage

### Install Dependencies

```bash
cd create-something-svelte
npm install
# Installs: gray-matter, marked, wrangler
```

### Run Sync

```bash
npm run sync-experiments
```

**Output:**
```
╔═══════════════════════════════════════╗
║  EXPERIMENT SYNC: GitHub → D1        ║
║  ────────────────────────────────────  ║
║  "Weniger, aber besser"              ║
║  (Less, but better)                  ║
╚═══════════════════════════════════════╝

📡 Fetching experiments from createsomethingtoday/create-something-experiments...
✅ Found 6 experiment files

📄 Processing: 03-kickstand-sustainability.md
✅ Synced: Experiment #3: Bug Fixes as Sustainability (kickstand-sustainability-nov-2025)

════════════════════════════════════════════════════════════
📊 SYNC COMPLETE
════════════════════════════════════════════════════════════
✅ Success: 6
❌ Failed: 0

⏱️  Duration: 2.45s
✅ Sync completed successfully
```

---

## File Structure

### Experiment Markdown File

```markdown
---
id: unique-experiment-id
slug: unique-experiment-slug
title: "Experiment #N: Title"
category: infrastructure
description: One-sentence description
featured: true
published: true
reading_time: 25
difficulty_level: advanced
# ... (see template.md for all fields)
---

# Experiment Content

## 1. Problem / Context
...

## 2. Hypothesis
...

[9-section structure]
```

### Database Record

After sync, the `papers` table contains:

```sql
{
  id: 'unique-experiment-id',
  slug: 'unique-experiment-slug',
  title: 'Experiment #N: Title',
  content: '# Experiment Content\n\n...',  -- Raw markdown
  html_content: '<h1>Experiment Content</h1>...',  -- Rendered HTML
  category: 'infrastructure',
  reading_time: 25,
  difficulty_level: 'advanced',
  interactive_demo_url: 'https://createsomething.space/experiments/slug' or NULL,
  featured: 1,
  published: 1,
  ...
}
```

---

## Frontmatter Fields

### Required
- `id` - Unique identifier (e.g., `kickstand-sustainability-nov-2025`)
- `slug` - URL-safe slug (e.g., `kickstand-sustainability-nov-2025`)
- `title` - Full title (e.g., `Experiment #3: Bug Fixes as Sustainability`)
- `category` - One of: `infrastructure`, `methodology`, `automation`, `ai-agents`
- `description` - One-sentence summary (160 chars max)

### Recommended
- `reading_time` - Minutes (e.g., `25`)
- `difficulty_level` - `beginner`, `intermediate`, or `advanced`
- `implementation_time` - String (e.g., `"6 hours"`)
- `excerpt_short` - One-liner for cards
- `excerpt_long` - 2-3 sentences for previews
- `meta_title`, `meta_description`, `focus_keywords` - SEO

### Optional
- `featured` - Show first in lists (default: `false`)
- `published` - Make visible (default: `false`)
- `is_hidden` - Hide from lists (default: `false`)
- `archived` - Mark as old (default: `false`)
- `interactive_demo_url` - Link to .space experiment
- `ascii_art` - Visual identity block
- `pathway` - Curriculum grouping
- `order` - Position in pathway

See `template.md` for complete schema.

---

## The Hermeneutic Circle

### Why This Structure?

The experiments repo is the **Vorhabe** (pre-understanding) that enables the hermeneutic cycle:

```
EXPERIMENTS REPO (Methodology)
    ↓
Defines how to track AI-native development
    ↓
SYNC PIPELINE (Translation)
    ↓
Experiments → D1 Database
    ↓
┌───────────────┐              ┌────────────────┐
│  .io          │              │  .space        │
│  (Theory)     │◄────────────►│  (Practice)    │
│  Read & Learn │              │  Do & Build    │
└───────────────┘              └────────────────┘
    ↓                              ↓
Understanding deepens through oscillation
    ↓
Reveals gaps in methodology
    ↓
EXPERIMENTS REPO (Updated)
```

**Key Insight:** This repository is not *derived from* .io or .space—it is the **source** that grounds both platforms.

---

## Workflow

### 1. Write Experiment Locally

```bash
# In this repo
cd create-something-experiments

# Create experiment from template
cp template.md experiments/07-my-experiment.md

# Edit with your favorite editor
code experiments/07-my-experiment.md
```

### 2. Fill Frontmatter

```yaml
---
id: my-experiment-2025
slug: my-experiment-2025
title: "Experiment #7: My Experiment Title"
category: infrastructure
description: Built X with Y, achieved Z% improvement
featured: true
published: true
reading_time: 18
difficulty_level: intermediate
implementation_time: "11 hours"
# ...
---
```

### 3. Write Content (9 Sections)

1. **Problem / Context** - What challenge prompted this?
2. **Hypothesis** - What did you believe would work?
3. **Build Process** - How did you implement it?
4. **Results** - What happened (metrics)?
5. **Analysis** - What worked? What didn't?
6. **Code Examples** - Key patterns
7. **System Logic** - (Optional) For complex systems
8. **Reproduction Guide** - How can others do this?
9. **Conclusions** - Hypothesis validation + insights

### 4. Commit & Push

```bash
git add experiments/07-my-experiment.md
git commit -m "feat: add Experiment #7 - My Experiment"
git push origin main
```

### 5. Sync to D1

```bash
cd ../create-something-svelte
npm run sync-experiments
```

### 6. Verify on Platforms

- **Full paper:** https://createsomething.io/experiments/my-experiment-2025
- **Interactive:** https://createsomething.space/experiments/my-experiment-2025 (if applicable)

---

## Common Tasks

### Update an Existing Experiment

1. Edit the `.md` file in `experiments/`
2. Update `updated_at` field
3. Commit & push
4. Run `npm run sync-experiments`

**Note:** Sync script uses `INSERT OR REPLACE` (upsert), so updates are automatic.

### Add Interactive Version

1. Set `interactive_demo_url` in frontmatter:
   ```yaml
   interactive_demo_url: "https://createsomething.space/experiments/my-slug"
   is_executable: 1
   ```

2. Add execution data (if applicable):
   ```yaml
   terminal_commands: |
     [
       {
         "step": 1,
         "title": "Install Wrangler",
         "command": "npm install -g wrangler",
         "explanation": "..."
       }
     ]
   ```

3. Sync: `npm run sync-experiments`

4. Verify on .space:
   - Shows brief summary (not full paper)
   - Interactive terminal/editor
   - Link back to .io

### Unpublish an Experiment

1. Set `published: false` in frontmatter
2. Commit & push
3. Sync: `npm run sync-experiments`

**Result:** Experiment hidden from .io and .space lists

### Archive Old Experiments

1. Set `archived: true` in frontmatter
2. Optionally add note in content: "**Note:** This experiment is archived..."
3. Sync: `npm run sync-experiments`

**Result:** Experiment remains accessible by URL but hidden from browse/search

---

## Troubleshooting

### "Failed to fetch experiments"

**Cause:** GitHub API rate limiting or repo not public

**Fix:**
- If repo is private, add `GITHUB_TOKEN` env var:
  ```bash
  export GITHUB_TOKEN=ghp_your_token
  npm run sync-experiments
  ```

### "Missing required fields"

**Cause:** Frontmatter missing `id`, `slug`, or `title`

**Fix:** Add required fields to frontmatter:
```yaml
---
id: my-experiment  # REQUIRED
slug: my-experiment  # REQUIRED
title: "Experiment #N: Title"  # REQUIRED
---
```

### "Duplicate slug"

**Cause:** Another experiment uses the same slug

**Fix:** Change slug to be unique:
```yaml
slug: my-experiment-v2  # Make it unique
```

### Sync Succeeds but Experiment Not Visible

**Check:**
1. `published: true` in frontmatter?
2. `is_hidden: false` in frontmatter?
3. `.io` deployment updated? (may need to redeploy)

---

## Architecture Details

### Why Manual Sync?

**"Weniger, aber besser"** (Less, but better)

We chose manual sync over automatic CI/CD because:

1. **Intentionality** - Publishing experiments is a deliberate act
2. **Simplicity** - No GitHub Actions, no webhooks, no automation complexity
3. **Control** - You decide when to sync (batch updates, review first)
4. **Debugging** - Easy to see what's being synced and why

**If** you want automated sync later:
```yaml
# .github/workflows/sync.yml
name: Sync Experiments
on:
  push:
    paths:
      - 'experiments/**'
jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: cd ../create-something-svelte && npm run sync-experiments
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CF_API_TOKEN }}
```

But we **don't recommend it** - manual is better.

### Why Upsert (INSERT OR REPLACE)?

**Benefits:**
- ✅ Updates existing experiments automatically
- ✅ No need to check "does this exist?"
- ✅ Idempotent (safe to run multiple times)
- ✅ Simplified logic

**Tradeoff:**
- ⚠️ Overwrites entire record (not partial update)

If you need partial updates, modify the script to use `UPDATE` instead.

### Why Fetch from GitHub vs Local Files?

**Current:** Fetch from GitHub API
**Alternative:** Read local files directly

**Why GitHub API?**
- ✅ Works from any machine (not just local)
- ✅ Always syncs latest committed version
- ✅ No risk of syncing uncommitted changes

**Tradeoff:**
- ⚠️ Requires network access
- ⚠️ Subject to GitHub rate limits

For local dev, you could modify the script to read files directly:
```typescript
import { readFileSync, readdirSync } from 'fs';
import { join } from 'path';

const experiments = readdirSync('./experiments')
  .filter(f => f.endsWith('.md'))
  .map(f => readFileSync(join('./experiments', f), 'utf-8'));
```

---

## Next Steps

1. **Migrate existing experiments** from SQL to markdown
2. **Backfill experiments #1-2** (KV Quickstart, Claude Agent SDK)
3. **Test sync pipeline** with all 6 experiments
4. **Verify on .io and .space**
5. **Document edge cases** (if any arise)

---

## See Also

- **Template:** `template.md` - Full experiment structure
- **Example:** `experiments/03-kickstand-sustainability.md` - Real experiment
- **Sync Script:** `../create-something-svelte/scripts/sync-experiments.ts`
- **Database Schema:** `../create-something-space-svelte/db/schema.sql`

---

**Philosophy:** The experiments repo is the **Vorhabe** (pre-understanding) that enables the entire CREATE SOMETHING ecosystem. The pipeline is the **translation** that makes this understanding accessible on both `.io` (theory) and `.space` (practice).

**"Weniger, aber besser"** — Less, but better.
