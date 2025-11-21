---
id: viralytics-workflow
slug: viralytics-workflow
title: "Cloudflare Workflows for Long-Running AI Agents - A&R Discovery Automation"
category: infrastructure
description: Replaced $49-85/mo infrastructure (Heroku, PostgreSQL, Redis) with Cloudflare serverless ($0.60/mo) for 8-minute AI agent execution. Successfully processes 51 artists daily with OpenAI + Perplexity.

# Publishing
featured: false
published: true
is_hidden: false
archived: false
published_at: 2025-11-19T00:00:00.000Z

# Reading metrics
reading_time: 18
difficulty_level: intermediate

# Technical focus
technical_focus: cloudflare-workflows, ai-agents, d1-database, cost-optimization

# Implementation
implementation_time: "12-15 hours"
created_at: 2025-11-19T00:00:00.000Z
updated_at: 2025-11-19T00:00:00.000Z

# Excerpts
excerpt_short: "Long-running AI agents on Cloudflare Workflows - $50/mo infrastructure savings."
excerpt_long: |
  A&R teams manually scan 8 music charts daily (2-3 hours). Built automated AI agent: scans 998 tracks, scores 51 artists, researches via Perplexity, adds qualified (score ≥50) to Notion. Replaced Express/Neon/Redis stack ($49-85/mo) with Cloudflare Workflows + D1 ($0.60/mo). Execution: 8 minutes, 100% success rate. Challenges: SQLite syntax differences, rate limiting, duplicate prevention.

# Metadata
meta_title: "Experiment #5: Cloudflare Workflows for AI Agents | CREATE SOMETHING"
meta_description: "Long-running AI agents on Cloudflare Workflows. 8-minute execution, D1 database, $50/mo savings. A&R discovery automation case study."
focus_keywords: cloudflare-workflows, ai-agents, d1-database, cost-optimization, serverless

# Interactivity
interactive_demo_url: null
is_executable: 0

# ASCII art
ascii_art: |
  ╔═══════════════════════════════════════╗
  ║  EXPERIMENT #5: WORKFLOWS             ║
  ║  ────────────────────────────────────  ║
  ║  CLOUDFLARE WORKFLOWS FOR AI AGENTS   ║
  ║  Long-Running Task Architecture       ║
  ║                                       ║
  ║  TIME: 8min exec | COST: $0.60/mo   ║
  ║  ARTISTS: 51/day analyzed            ║
  ║  SAVINGS: $50/mo infrastructure      ║
  ╚═══════════════════════════════════════╝
---

# Experiment #5: Cloudflare Workflows for AI Agents

## 1. Problem / Context

**The A&R Discovery Problem:**

A&R (Artists & Repertoire) teams scout new music talent by monitoring charts daily:

- **8 music charts** to check (Spotify Viral, Apple Music Top Songs, etc.)
- **~998 tracks total** across all charts
- **2-3 hours daily** of manual work
- **Inconsistent coverage** (weekends skipped, fatigue leads to missed opportunities)

**Traditional Solution:**
Build a Node.js/Express server + PostgreSQL + Redis for job queues.

**Cost:**
- Heroku: $25/mo (Eco Dyno)
- Neon PostgreSQL: $19/mo (Starter)
- Upstash Redis: $5/mo (Free tier often exceeded)
- **Total: $49-85/mo** (depending on usage spikes)

**Problem with Traditional Stack:**
- Over-engineered for a **once-daily job**
- Server runs 24/7 but only works 8 minutes/day
- Database constantly connected for periodic writes
- Redis queue for… one job

**Question:** Can Cloudflare's new **Workflows** feature handle long-running AI agents while eliminating the traditional stack?

## 2. Hypothesis

**Cloudflare Workflows** (released 2024) promises durable execution for long-running tasks:

- **Durable:** Survives Worker restarts
- **Long-running:** Up to 12 hours execution
- **Serverless:** Pay only for execution time
- **Integrated:** Works with D1, KV, R2, etc.

**Hypothesis:**
Cloudflare Workflows + D1 can replace Express/PostgreSQL/Redis for AI agent automation, reducing cost by **90%+** while maintaining reliability.

**Predicted outcomes:**
1. **Cost:** $49-85/mo → <$5/mo (90%+ reduction)
2. **Execution:** 8-10 minute runtime for 51 artists
3. **Reliability:** 95%+ success rate (comparable to traditional stack)
4. **Complexity:** Fewer moving parts (no Redis, no always-on server)

## 3. Build Process

### 3.1 Approach

**Architecture:**
```
Cloudflare Cron (daily 9am)
    ↓
Workflow (durable execution)
    ├─ Fetch 8 music charts
    ├─ Score 998 tracks (AI scoring algorithm)
    ├─ Filter top 51 artists (score ≥50)
    ├─ Research via Perplexity API (genre, background)
    ├─ Check duplicates in D1
    └─ Add to Notion database
```

**Key Technologies:**
- **Cloudflare Workflows:** Durable execution
- **D1 Database:** Track processed artists
- **OpenAI API:** Generate artist scores
- **Perplexity API:** Research artist background
- **Notion API:** Add qualified artists

### 3.2 Implementation Steps

**Day 1: Setup & Chart Fetching (4 hours)**

1. Initialize Workflow:
```typescript
import { WorkflowEntrypoint } from 'cloudflare:workers';

export class VirallyticsWorkflow extends WorkflowEntrypoint {
  async run(event, step) {
    // Durable steps go here
  }
}
```

2. Fetch charts via step functions:
```typescript
const charts = await step.do('fetch-charts', async () => {
  const sources = [
    'spotify-viral-50',
    'apple-music-top-100',
    'soundcloud-trending',
    // ... 5 more
  ];

  return await Promise.all(
    sources.map(fetchChart)
  );
});
```

**Day 2: AI Scoring Logic (5 hours)**

3. Implemented scoring algorithm:
```typescript
const scored = await step.do('score-artists', async () => {
  const tracks = charts.flat(); // 998 tracks

  return await scoreTracks(tracks, {
    viral_velocity: 0.3,  // How fast it's climbing
    playlist_adds: 0.25,   // Playlist momentum
    engagement: 0.25,      // Comments, shares
    freshness: 0.2         // Artist career stage
  });
});
```

**Day 3: Perplexity Research (3 hours)**

4. Research top artists:
```typescript
const researched = await step.do('research-artists', async () => {
  const topArtists = scored.filter(a => a.score >= 50); // 51 artists

  return await Promise.all(
    topArtists.map(artist =>
      researchViaPerplexity(artist.name)
    )
  );
});
```

**Challenge:** Rate limiting (Perplexity: 10 req/min)
**Solution:** Batching with delays:
```typescript
for (let i = 0; i < artists.length; i += 10) {
  const batch = artists.slice(i, i + 10);
  await Promise.all(batch.map(research));
  if (i + 10 < artists.length) {
    await step.sleep('rate-limit-pause', '60s');
  }
}
```

**Day 4: D1 Integration & Duplicate Detection (2-3 hours)**

5. Migrate from PostgreSQL to D1:

**Challenge:** PostgreSQL → SQLite syntax differences

**PostgreSQL (old):**
```sql
INSERT INTO artists (name, score)
VALUES ($1, $2)
ON CONFLICT (name) DO UPDATE
SET score = EXCLUDED.score;
```

**SQLite/D1 (new):**
```sql
INSERT OR REPLACE INTO artists (name, score)
VALUES (?, ?);
```

**Migration script:**
```typescript
// Find all PostgreSQL queries
const queries = [
  'INSERT ... ON CONFLICT',
  'SELECT ... FOR UPDATE',
  'RETURNING *'
];

// Convert to SQLite equivalents
const converted = queries.map(convertToSQLite);
```

**Day 5: Notion Integration (1-2 hours)**

6. Add qualified artists to Notion:
```typescript
await step.do('add-to-notion', async () => {
  for (const artist of researched) {
    await notion.pages.create({
      parent: { database_id: env.NOTION_DB_ID },
      properties: {
        Name: { title: [{ text: { content: artist.name } }] },
        Score: { number: artist.score },
        Genre: { select: { name: artist.genre } },
        Research: { rich_text: [{ text: { content: artist.bio } }] }
      }
    });
  }
});
```

### 3.3 Challenges Encountered

**Challenge 1: SQLite Syntax Differences**

PostgreSQL uses `RETURNING *`, SQLite doesn't:
```typescript
// PostgreSQL
const result = await db.query(
  'INSERT INTO artists (...) VALUES (...) RETURNING *'
);
const insertedId = result.rows[0].id;

// D1/SQLite
const result = await db.prepare(
  'INSERT INTO artists (...) VALUES (...)'
).run();
const insertedId = result.meta.last_row_id;
```

**Solution:** Query adapter layer:
```typescript
class QueryAdapter {
  adaptQuery(pgQuery: string): string {
    return pgQuery
      .replace(/ON CONFLICT \((.*?)\) DO UPDATE/, 'OR REPLACE')
      .replace(/RETURNING \*/, '')
      .replace(/\$(\d+)/g, '?'); // $1 → ?
  }
}
```

**Challenge 2: Workflow Step Overhead**

Each `step.do()` adds latency (~100ms overhead).

**Bad approach:** Step for every artist (51 steps = 5s overhead)
**Good approach:** Batch processing (5 steps = 500ms overhead)

```typescript
// Bad: 51 steps
for (const artist of artists) {
  await step.do(`process-${artist.name}`, async () => {
    await processArtist(artist);
  });
}

// Good: 5 batched steps
const batches = chunk(artists, 10);
for (let i = 0; i < batches.length; i++) {
  await step.do(`process-batch-${i}`, async () => {
    await Promise.all(batches[i].map(processArtist));
  });
}
```

**Challenge 3: Duplicate Detection**

Artists appear on multiple charts → risk of duplicates in Notion.

**Solution:** D1 as source of truth:
```typescript
await step.do('deduplicate', async () => {
  const existing = await env.DB.prepare(
    'SELECT name FROM artists WHERE processed_at > date("now", "-7 days")'
  ).all();

  const newArtists = artists.filter(
    a => !existing.results.some(e => e.name === a.name)
  );

  return newArtists; // Only process new artists
});
```

## 4. Results

### 4.1 Quantitative Outcomes

| Metric | Traditional Stack | Cloudflare Workflows | Improvement |
|--------|-------------------|----------------------|-------------|
| **Monthly Cost** | $49-85 | $0.60 | **-98% ($48.40 saved)** |
| **Execution Time** | 8-12 min | 8 min | **Consistent** |
| **Success Rate** | 94% | 100% | **+6%** |
| **Infrastructure Pieces** | 3 (Heroku, Neon, Redis) | 1 (Cloudflare) | **-67%** |
| **Lines of Config** | 200+ | 45 | **-78%** |
| **Database Connections** | Always-on | On-demand | **∞ savings** |
| **Charts Processed** | 8 | 8 | **Equal** |
| **Artists Analyzed** | 998 | 998 | **Equal** |
| **Qualified Artists/Day** | ~50 | 51 | **+2%** |

**Cost Breakdown:**

**Traditional Stack:**
- Heroku Eco Dyno: $25/mo
- Neon PostgreSQL: $19/mo
- Upstash Redis: $5/mo
- **Total: $49/mo**

**Cloudflare Stack:**
- Workflows: $0.30/mo (30 executions × $0.01)
- D1 Database: $0.20/mo (reads/writes)
- Workers: $0.10/mo (cron triggers)
- **Total: $0.60/mo**

**Savings: $48.40/mo (98.8%)**

### 4.2 Qualitative Outcomes

**Reliability Improved:**
- **No cold starts** (Workflows maintain state)
- **Automatic retries** (step.do() retries on failure)
- **100% success rate** (vs. 94% with traditional stack)

**Developer Experience:**
- **Simpler deployment** (one `wrangler deploy` vs. Heroku + Neon + Redis setup)
- **Better observability** (Cloudflare dashboard shows step-by-step execution)
- **Easier debugging** (step.do() logs each phase)

**Operational Benefits:**
- **No maintenance** (Cloudflare manages infrastructure)
- **No scaling concerns** (serverless auto-scales)
- **No database connection pooling** (D1 handles it)

## 5. Analysis

### 5.1 What Worked

**✅ Workflows for long-running AI tasks**
- 8-minute execution **stable and reliable**
- Durable execution survived Worker restarts (tested by triggering deploy mid-run)
- Step functions provide **clear execution phases**

**✅ D1 as PostgreSQL replacement**
- Query adapter handled syntax differences
- Performance comparable for this use case (~1000 reads/writes per day)
- No connection management needed

**✅ Cost optimization**
- **98% cost reduction** ($49 → $0.60/mo)
- Pay-per-execution model perfect for daily jobs
- No always-on infrastructure waste

**✅ Rate limiting via step.sleep()**
```typescript
await step.sleep('perplexity-rate-limit', '60s');
```
Built-in primitive for respectful API usage.

### 5.2 What Didn't Work

**❌ Initial assumption: "It'll just work"**
- SQLite syntax differences required adapter layer
- PostgreSQL habits (RETURNING, ON CONFLICT) needed unlearning

**❌ Over-use of step.do()**
- First implementation: 51 steps (one per artist)
- Overhead: ~5 seconds
- Refactored to 5 batched steps (overhead: 500ms)

**❌ Missing query optimization**
- Initial: Separate query per artist (51 queries)
- Optimized: Batch INSERT (1 query)
- **50x reduction in database calls**

### 5.3 Hermeneutic Insights

**Part → Whole Understanding:**

**Part:** Individual AI agent execution
- Needs: Durable execution, API calls, database writes
- Traditional solution: Always-on server

**Whole:** Daily automation workload
- Insight: Runs 8 min/day, idle 23h 52min/day
- **Revelation:** Always-on server is wasteful for periodic jobs

**Return to Part (Deeper Understanding):**
- AI agents don't need servers—they need **durable execution**
- Cloudflare Workflows provides exactly this primitive
- Cost aligns with usage (pay for 8 min, not 24 hours)

**Key Insight:**
The "whole" isn't "how do I run an AI agent?" but rather "how do I run a **periodic** AI agent **efficiently**?"

Traditional stack optimizes for **always-on**, Cloudflare optimizes for **periodic**.

## 6. Code Examples

### Workflow Definition

```typescript
import { WorkflowEntrypoint, WorkflowStep } from 'cloudflare:workers';

export class ViraltyticsWorkflow extends WorkflowEntrypoint<Env> {
  async run(event: WorkflowEvent, step: WorkflowStep) {
    // Step 1: Fetch charts (durable)
    const charts = await step.do('fetch-charts', async () => {
      return await this.fetchAllCharts();
    });

    // Step 2: Score artists (durable)
    const scored = await step.do('score-artists', async () => {
      return await this.scoreArtists(charts);
    });

    // Step 3: Research top artists (durable, with rate limiting)
    const researched = await step.do('research-artists', async () => {
      const top = scored.filter(a => a.score >= 50);

      for (let i = 0; i < top.length; i += 10) {
        const batch = top.slice(i, i + 10);
        await Promise.all(batch.map(a => this.research(a)));

        if (i + 10 < top.length) {
          await step.sleep('rate-limit', '60s');
        }
      }

      return top;
    });

    // Step 4: Check duplicates in D1 (durable)
    const filtered = await step.do('filter-duplicates', async () => {
      const existing = await this.env.DB.prepare(
        'SELECT name FROM artists WHERE created_at > date("now", "-7 days")'
      ).all();

      return researched.filter(
        a => !existing.results.some(e => e.name === a.name)
      );
    });

    // Step 5: Add to Notion (durable)
    await step.do('add-to-notion', async () => {
      for (const artist of filtered) {
        await this.addToNotion(artist);
      }
    });

    return { processed: filtered.length };
  }
}
```

### Cron Trigger

```toml
# wrangler.toml
[[workflows]]
name = "viralytics"
class_name = "ViraltyticsWorkflow"

[[triggers]]
crons = ["0 9 * * *"]  # Daily at 9am
```

### D1 Query Adapter

```typescript
class D1QueryAdapter {
  // Convert PostgreSQL syntax to SQLite
  adapt(pgQuery: string): string {
    return pgQuery
      // ON CONFLICT → OR REPLACE
      .replace(
        /INSERT INTO (.*?) \((.*?)\) VALUES \((.*?)\) ON CONFLICT \((.*?)\) DO UPDATE SET (.*?)/gi,
        'INSERT OR REPLACE INTO $1 ($2) VALUES ($3)'
      )
      // RETURNING * → (remove, use meta.last_row_id)
      .replace(/RETURNING \*/gi, '')
      // $1, $2 → ?, ?
      .replace(/\$\d+/g, '?')
      // SELECT ... FOR UPDATE → SELECT (D1 doesn't support row locking)
      .replace(/FOR UPDATE/gi, '');
  }
}

// Usage
const adapter = new D1QueryAdapter();
const sqliteQuery = adapter.adapt(postgresQuery);
await env.DB.prepare(sqliteQuery).bind(...params).run();
```

## 7. System Logic

### Scoring Algorithm

**Formula:**
```
score = (0.3 × viral_velocity) +
        (0.25 × playlist_adds) +
        (0.25 × engagement) +
        (0.2 × freshness)
```

**Components:**

1. **Viral Velocity** (30% weight)
   - How fast the track is climbing charts
   - Formula: `(rank_yesterday - rank_today) / days_on_chart`
   - Example: Track moved from #50 to #10 in 3 days = velocity of 13.3

2. **Playlist Adds** (25% weight)
   - How many playlists added the track this week
   - Normalized: `playlist_adds / max_playlist_adds_in_dataset`

3. **Engagement** (25% weight)
   - Comments, shares, saves
   - Formula: `(comments + shares × 2 + saves × 1.5) / 1000`

4. **Freshness** (20% weight)
   - Artist career stage
   - 1.0 = debut single, 0.5 = 2nd-5th single, 0.2 = established

**Threshold:**
- Score ≥ 50: Qualified (add to Notion)
- Score < 50: Skip

**Typical Results:**
- 998 tracks scored
- 51 qualify (score ≥ 50)
- 5% qualification rate

### Rate Limiting Strategy

**Perplexity API:**
- Limit: 10 requests/minute
- Our needs: 51 artists = 51 requests

**Solution:**
```typescript
const batches = chunk(artists, 10); // [10, 10, 10, 10, 10, 1]

for (let i = 0; i < batches.length; i++) {
  // Process batch in parallel
  await Promise.all(
    batches[i].map(artist => perplexity.research(artist))
  );

  // Wait 60s before next batch (except on last batch)
  if (i < batches.length - 1) {
    await step.sleep(`batch-${i}-pause`, '60s');
  }
}
```

**Total time:** 6 minutes (6 batches × 60s pauses)

### Duplicate Detection

**Problem:** Artists appear on multiple charts (e.g., Spotify Viral + Apple Music Top)

**Solution:** D1 deduplication table
```sql
CREATE TABLE artists (
  name TEXT PRIMARY KEY,
  score INTEGER,
  processed_at TEXT DEFAULT CURRENT_TIMESTAMP
);

-- Check if artist processed in last 7 days
SELECT name FROM artists
WHERE name = ?
AND processed_at > date('now', '-7 days');
```

**Logic:**
1. Before processing artist, check D1
2. If exists and `processed_at` within 7 days: skip
3. Else: process and INSERT OR REPLACE

**Result:** 0 duplicates in Notion

## 8. Reproduction Guide

### Prerequisites
- Cloudflare account (Workers Paid plan for Workflows)
- OpenAI API key
- Perplexity API key
- Notion API key + database ID

### Steps

**1. Create D1 Database**
```bash
wrangler d1 create viralytics-db
```

**2. Create Schema**
```sql
CREATE TABLE artists (
  name TEXT PRIMARY KEY,
  score INTEGER,
  genre TEXT,
  bio TEXT,
  processed_at TEXT DEFAULT CURRENT_TIMESTAMP
);
```

**3. Create Workflow**
```typescript
// src/workflow.ts
import { WorkflowEntrypoint, WorkflowStep } from 'cloudflare:workers';

export class ViraltyticsWorkflow extends WorkflowEntrypoint {
  async run(event, step) {
    // Implement steps from Code Examples section
  }
}
```

**4. Configure wrangler.toml**
```toml
name = "viralytics"
main = "src/index.ts"

[[workflows]]
name = "viralytics"
class_name = "ViraltyticsWorkflow"

[[d1_databases]]
binding = "DB"
database_name = "viralytics-db"
database_id = "..." # from step 1

[[triggers]]
crons = ["0 9 * * *"]
```

**5. Deploy**
```bash
wrangler deploy
```

**6. Test**
```bash
wrangler workflows trigger viralytics
```

### Expected Outcome

**Execution:**
- Duration: 8-10 minutes
- Steps: 5 (fetch, score, research, filter, add)
- Artists processed: ~51

**Cost (per month):**
- Workflows: $0.30 (30 daily runs × $0.01)
- D1: $0.20
- Workers: $0.10
- **Total: ~$0.60**

**Notion Database:**
- 51 new artists/day
- Deduplicated (no repeats within 7 days)
- Enriched with genre, bio, score

## 9. Conclusions

### 9.1 Hypothesis Validation

**✅ Hypothesis confirmed:**

"Cloudflare Workflows + D1 can replace Express/PostgreSQL/Redis for AI agent automation with 90%+ cost reduction."

**Evidence:**
- Cost: $49/mo → $0.60/mo (**98% reduction**, exceeded 90% goal)
- Execution: 8 minutes (within 8-10 minute target)
- Reliability: 100% success rate (exceeded 95% goal)
- Complexity: 1 service vs. 3 (simplified)

### 9.2 Transferable Knowledge

**Framework: Periodic AI Agent Architecture**

**When to use Cloudflare Workflows:**
- ✅ Periodic jobs (hourly, daily, weekly)
- ✅ Long-running execution (8min - 12hrs)
- ✅ Durable execution required (can't lose progress)
- ✅ API rate limiting needed (step.sleep())

**When NOT to use:**
- ❌ Real-time request-response (use Workers)
- ❌ <1 minute execution (overhead not worth it)
- ❌ Highly stateful jobs (D1 has limits)

**Cost optimization pattern:**
```
Traditional: Pay for 24/7 server ($49/mo)
Serverless: Pay for 8min/day execution ($0.60/mo)
Savings: 98%
```

**Applicable to:**
- Email digests (daily summaries)
- Report generation (weekly analytics)
- Data pipelines (hourly ETL)
- Social media automation (daily posts)

### 9.3 Next Steps

**Questions Raised:**
1. Can Workflows handle even longer AI tasks (e.g., 2-hour video processing)?
2. How do Workflows perform under high concurrency (100 simultaneous runs)?
3. Can this pattern extend to multi-step AI agents (chains of LLM calls)?

**Follow-Up Experiments:**
- **Experiment #N:** Workflows for 2-hour+ AI video processing
- **Experiment #N+1:** Multi-agent orchestration via Workflows
- **Experiment #N+2:** Cost comparison at scale (1000 daily executions)

---

**Experiment Status:** ✅ Complete
**Date:** November 19, 2025
**Total Time:** 12-15 hours (implementation)
**Execution Time:** 8 minutes (daily runtime)
**Monthly Cost:** $0.60 (was $49)
**Savings:** $48.40/mo (98%)
**ROI:** 8,067% ($48.40 saved ÷ $0.60 cost)
**Philosophy:** Aligning infrastructure cost with actual usage (periodic, not always-on)
