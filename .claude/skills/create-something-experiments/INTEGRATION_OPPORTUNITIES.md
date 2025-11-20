# Integration Opportunities: Enhancing CREATE SOMETHING Experiment Tracking

Analysis of how [@disler's Claude Code repos](https://github.com/disler) can enhance the CREATE SOMETHING experiment tracking Skill.

## Summary of Evaluated Repos

1. **[claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery)** - Complete hook lifecycle implementation with Python UV scripts
2. **[always-on-ai-assistant](https://github.com/disler/always-on-ai-assistant)** - Voice-interactive AI assistant (Deepseek-V3 + RealtimeSTT)
3. **[multi-agent-postgres-data-analytics](https://github.com/disler/multi-agent-postgres-data-analytics)** - Multi-agent systems for database queries
4. **[claude-code-hooks-multi-agent-observability](https://github.com/disler/claude-code-hooks-multi-agent-observability)** - Real-time dashboard for monitoring Claude Code agents
5. **[just-prompt](https://github.com/disler/just-prompt)** - MCP server for unified LLM provider access

## HIGH-VALUE INTEGRATIONS

### 🔥 #1: claude-code-hooks-mastery → Automated Experiment Tracking

**What It Does:** Provides production-ready implementations of all 8 Claude Code hook lifecycle events using UV single-file Python scripts.

**Why It's Perfect for CREATE SOMETHING:**

The experiment tracking Skill currently requires **manual logging** of prompts, errors, and interventions. With hooks, this becomes **fully automated**.

**Integration Plan:**

#### 1. UserPromptSubmit Hook → Auto-Log Prompts

**Current Approach** (manual):
```bash
# User has to manually run this after each iteration
echo '{"iteration":1,"prompt":"Build X","response":"..."}' >> .claude/experiments/project/prompts.jsonl
```

**With Hooks** (automatic):
```python
# .claude/hooks/user-prompt-submit-experiment-tracker.py
#!/usr/bin/env -S uv run --quiet --script
# /// script
# dependencies = ["anthropic"]
# ///

import json
import sys
from datetime import datetime
from pathlib import Path

def main():
    # Read user prompt from stdin
    user_input = json.loads(sys.stdin.read())
    prompt_text = user_input.get("userMessage", {}).get("text", "")

    # Check if we're in an active experiment
    exp_dir = Path(".claude/experiments")
    if not exp_dir.exists():
        return  # No experiment active, exit silently

    # Find most recent experiment directory
    experiments = sorted(exp_dir.iterdir(), key=lambda p: p.stat().st_mtime, reverse=True)
    if not experiments:
        return

    current_exp = experiments[0]
    prompts_file = current_exp / "prompts.jsonl"

    # Log the prompt
    log_entry = {
        "timestamp": datetime.utcnow().isoformat() + "Z",
        "iteration": len(prompts_file.read_text().splitlines()) + 1 if prompts_file.exists() else 1,
        "prompt": prompt_text,
        "session_id": user_input.get("sessionId", "unknown")
    }

    with prompts_file.open("a") as f:
        f.write(json.dumps(log_entry) + "\n")

if __name__ == "__main__":
    main()
```

**Result:** Every prompt automatically logged to `.claude/experiments/[project]/prompts.jsonl`

#### 2. PostToolUse Hook → Auto-Log Tool Executions

Track what tools Claude uses, acceptance/rejection, and execution time:

```python
# .claude/hooks/post-tool-use-experiment-tracker.py
#!/usr/bin/env -S uv run --quiet --script
# /// script
# dependencies = []
# ///

import json
import sys
from datetime import datetime
from pathlib import Path

def main():
    event = json.loads(sys.stdin.read())

    exp_dir = Path(".claude/experiments")
    if not exp_dir.exists():
        return

    experiments = sorted(exp_dir.iterdir(), key=lambda p: p.stat().st_mtime, reverse=True)
    if not experiments:
        return

    current_exp = experiments[0]
    metrics_file = current_exp / "metrics.jsonl"

    # Extract tool info
    tool_use = event.get("toolUse", {})
    tool_result = event.get("toolResult", {})

    log_entry = {
        "timestamp": datetime.utcnow().isoformat() + "Z",
        "tool_name": tool_use.get("name"),
        "tool_input": tool_use.get("input"),
        "success": not tool_result.get("isError", False),
        "execution_time_ms": event.get("executionTimeMs", 0),
        "user_accepted": event.get("userAccepted", True)
    }

    with metrics_file.open("a") as f:
        f.write(json.dumps(log_entry) + "\n")

if __name__ == "__main__":
    main()
```

**Result:** Automatic tracking of:
- Which tools Claude uses most
- Acceptance vs rejection rates
- Execution times
- Error patterns

#### 3. Notification Hook → Experiment Milestones

Alert when key milestones are reached:

```python
# .claude/hooks/notification-experiment-milestones.py
#!/usr/bin/env -S uv run --quiet --script
# /// script
# dependencies = ["requests"]
# ///

import json
import sys
from pathlib import Path

def main():
    event = json.loads(sys.stdin.read())
    message = event.get("message", "")

    exp_dir = Path(".claude/experiments")
    if not exp_dir.exists():
        return

    experiments = sorted(exp_dir.iterdir(), key=lambda p: p.stat().st_mtime, reverse=True)
    if not experiments:
        return

    current_exp = experiments[0]
    prompts_file = current_exp / "prompts.jsonl"

    # Check for milestones
    if prompts_file.exists():
        prompt_count = len(prompts_file.read_text().splitlines())

        # Alert on iteration milestones
        if prompt_count in [10, 25, 50, 100]:
            milestone_msg = f"🎯 Experiment Milestone: {prompt_count} iterations completed!"

            # Inject message for Claude to announce
            result = {
                "decision": "allow",
                "modifiedMessage": f"{message}\n\n{milestone_msg}"
            }
            print(json.dumps(result))
            return

    # Default: allow notification as-is
    print(json.dumps({"decision": "allow"}))

if __name__ == "__main__":
    main()
```

**Result:** Automatic milestone announcements during long experiments

#### 4. SessionStart Hook → Experiment Auto-Init

Automatically detect when you're starting a new project and offer to track it:

```python
# .claude/hooks/session-start-experiment-init.py
#!/usr/bin/env -S uv run --quiet --script
# /// script
# dependencies = []
# ///

import json
import sys
from pathlib import Path

def main():
    event = json.loads(sys.stdin.read())

    # Check if .claude/experiments exists
    exp_dir = Path(".claude/experiments")

    # If no experiments directory, create one and initialize
    if not exp_dir.exists():
        exp_dir.mkdir(parents=True, exist_ok=True)

        # Inject context for Claude to suggest experiment tracking
        result = {
            "additionalContext": """
🔬 CREATE SOMETHING Experiment Tracking Available

I notice you don't have experiment tracking set up. Would you like me to:
1. Initialize experiment tracking for this session?
2. Track prompts, errors, costs, and interventions automatically?
3. Generate a CREATE SOMETHING paper when we're done?

Just say "yes, track this experiment" and I'll set it up.
"""
        }
        print(json.dumps(result))
        return

if __name__ == "__main__":
    main()
```

**Result:** Claude proactively offers experiment tracking on new sessions

### 🚀 #2: claude-code-hooks-multi-agent-observability → Real-Time Experiment Dashboard

**What It Does:** Provides a Vue 3 web dashboard that displays Claude Code agent activity in real-time via WebSocket connections.

**Why It's Perfect for CREATE SOMETHING:**

Instead of reading JSONL files to see experiment progress, you get a **live dashboard** showing:
- Prompts as they happen
- Tool executions in real-time
- Error patterns emerging
- Cost accumulation
- Session activity visualization

**Integration Plan:**

#### 1. Use Existing Server + Dashboard

The repo includes:
- **Bun server** (TypeScript) with SQLite for event storage
- **Vue 3 dashboard** with live updates
- **WebSocket protocol** for real-time streaming

#### 2. Add CREATE SOMETHING-Specific Views

Extend the dashboard with experiment-specific panels:

```typescript
// dashboard/src/components/ExperimentMetrics.vue
<template>
  <div class="experiment-metrics">
    <h2>Current Experiment: {{ experimentName }}</h2>

    <!-- Cost Tracker -->
    <div class="metric-card">
      <h3>💰 Cost</h3>
      <div class="cost-total">${{ totalCost }}</div>
      <div class="cost-breakdown">
        <span>Claude: ${{ claudeCost }}</span>
        <span>Cloudflare: ${{ cloudflareCost }}</span>
      </div>
    </div>

    <!-- Time Tracker -->
    <div class="metric-card">
      <h3>⏱️ Time</h3>
      <div class="time-total">{{ sessionHours }} hours</div>
      <div class="time-estimate">Est. manual: {{ manualEstimate }} hrs</div>
      <div class="savings">{{ timeSavings }}% savings</div>
    </div>

    <!-- Error Tracker -->
    <div class="metric-card">
      <h3>🐛 Errors</h3>
      <div class="error-count">{{ errorCount }} total</div>
      <div class="error-types">
        <div v-for="(count, type) in errorsByType" :key="type">
          {{ type }}: {{ count }}
        </div>
      </div>
    </div>

    <!-- Intervention Tracker -->
    <div class="metric-card">
      <h3>✋ Interventions</h3>
      <div class="intervention-count">{{ interventionCount }}</div>
      <div class="last-intervention">
        Last: {{ lastIntervention?.reason }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useExperimentStore } from '../stores/experiment'

const store = useExperimentStore()

const experimentName = computed(() => store.currentExperiment?.name)
const totalCost = computed(() => store.metrics.totalCost.toFixed(2))
const claudeCost = computed(() => store.metrics.claudeCost.toFixed(2))
const cloudflareCost = computed(() => store.metrics.cloudflareCost.toFixed(2))
const sessionHours = computed(() => (store.metrics.sessionTime / 3600000).toFixed(1))
const manualEstimate = computed(() => store.metrics.manualEstimate)
const timeSavings = computed(() => {
  const savings = ((store.metrics.manualEstimate - parseFloat(sessionHours.value)) / store.metrics.manualEstimate * 100)
  return savings.toFixed(0)
})
const errorCount = computed(() => store.errors.length)
const errorsByType = computed(() => {
  return store.errors.reduce((acc, error) => {
    acc[error.type] = (acc[error.type] || 0) + 1
    return acc
  }, {} as Record<string, number>)
})
const interventionCount = computed(() => store.interventions.length)
const lastIntervention = computed(() => store.interventions[store.interventions.length - 1])
</script>
```

#### 3. Hook Integration

Send experiment data from hooks to the observability server:

```python
# .claude/hooks/post-tool-use-send-to-dashboard.py
#!/usr/bin/env -S uv run --quiet --script
# /// script
# dependencies = ["requests"]
# ///

import json
import sys
import requests
from datetime import datetime

def main():
    event = json.loads(sys.stdin.read())

    # Send to observability server
    try:
        requests.post("http://localhost:3000/api/events", json={
            "source_app": "create-something",
            "session_id": event.get("sessionId"),
            "event_type": "tool_use",
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "data": {
                "tool_name": event.get("toolUse", {}).get("name"),
                "success": not event.get("toolResult", {}).get("isError", False),
                "execution_time_ms": event.get("executionTimeMs", 0)
            }
        }, timeout=0.5)
    except:
        pass  # Silent fail if dashboard not running

if __name__ == "__main__":
    main()
```

**Result:** Real-time dashboard at `http://localhost:8080` showing:
- Live experiment metrics
- Cost accumulation
- Time tracking
- Error patterns
- Intervention log

### ⚡ #3: just-prompt → Multi-Model Experiments

**What It Does:** MCP server providing unified interface to OpenAI, Anthropic, Google Gemini, Groq, DeepSeek, and Ollama.

**Why It's Perfect for CREATE SOMETHING:**

Test whether **different AI models** produce different results for the same experiment. Example experiments:
- "Does GPT-4 build faster than Claude Sonnet for X?"
- "Quality comparison: Claude vs Gemini for code generation"
- "Cost analysis: DeepSeek vs OpenAI for automation tasks"

**Integration Plan:**

#### 1. Install just-prompt MCP Server

```bash
# Add to Claude Code MCP configuration
mcp add github:disler/just-prompt
```

#### 2. Create Multi-Model Experiment Template

```markdown
# Experiment #X: Multi-Model Comparison for [Task]

## Hypothesis
Different LLMs will show different strengths for [specific task type].

## Models Tested
- Claude Sonnet 4.5 (baseline)
- GPT-4o (comparison)
- Gemini 2.0 Flash (comparison)
- DeepSeek V3 (cost-optimized comparison)

## Test Methodology
Same prompt sent to all models:
```
[exact prompt here]
```

## Results

### Claude Sonnet 4.5
- Time: X mins
- Cost: $X
- Lines of code: X
- Quality score: X/10
- What it did well: [...]
- What failed: [...]

### GPT-4o
[same metrics]

### Gemini 2.0 Flash
[same metrics]

### DeepSeek V3
[same metrics]

## Winner: [Model]
[Analysis of why]
```

#### 3. Use just-prompt's "CEO and Board" Feature

Test architectural decisions by getting consensus from multiple models:

```
Use the ceo_and_board tool to decide: Should we use Workflows or Durable Objects for this use case?

Board members:
- o:gpt-4o (OpenAI reasoning)
- a:claude-sonnet-4.5 (Anthropic reasoning)
- g:gemini-2.0-flash (Google reasoning)

CEO:
- a:claude-sonnet-4.5 (synthesizes recommendations)
```

**Result:** Data on which models excel at specific tasks, enabling informed model selection for future experiments.

## MEDIUM-VALUE INTEGRATIONS

### #4: multi-agent-postgres-data-analytics → Store Experiments in Postgres

**Current Approach:** JSONL files in `.claude/experiments/[name]/`

**With Postgres:**
```sql
CREATE TABLE experiments (
  id UUID PRIMARY KEY,
  name TEXT NOT NULL,
  hypothesis TEXT,
  start_time TIMESTAMP,
  end_time TIMESTAMP,
  total_cost DECIMAL,
  time_savings_percent INTEGER
);

CREATE TABLE experiment_prompts (
  id UUID PRIMARY KEY,
  experiment_id UUID REFERENCES experiments(id),
  iteration INTEGER,
  prompt TEXT,
  response TEXT,
  timestamp TIMESTAMP
);

CREATE TABLE experiment_errors (
  id UUID PRIMARY KEY,
  experiment_id UUID REFERENCES experiments(id),
  error_type TEXT,
  error_message TEXT,
  context TEXT,
  fix_time_minutes INTEGER,
  timestamp TIMESTAMP
);

CREATE TABLE experiment_interventions (
  id UUID PRIMARY KEY,
  experiment_id UUID REFERENCES experiments(id),
  reason TEXT,
  what_user_did TEXT,
  why_needed TEXT,
  time_spent_minutes INTEGER,
  timestamp TIMESTAMP
);
```

**Benefits:**
- Query experiments: "Show me all experiments with >50% time savings"
- Aggregate data: "Average error count across all Cloudflare experiments"
- Cross-experiment analysis: "Which tools does Claude use most across all experiments?"

**Implementation:** Swap JSONL writes with Postgres inserts (Cloudflare D1 or external Postgres)

### #5: always-on-ai-assistant → Voice-Driven Development Experiments

**Use Case:** Test if **voice-driven development** with Claude Code is faster than typing.

**Experiment:**
```markdown
# Experiment #X: Voice vs Keyboard for AI-Native Development

## Hypothesis
Speaking prompts to Claude Code via always-on voice assistant will be faster than typing, but less precise.

## Test Methodology
Build identical feature twice:
1. Trial A: Type all prompts (baseline)
2. Trial B: Speak all prompts via always-on assistant

## Metrics
- Time to completion
- Prompt precision (measured by Claude clarification requests)
- Error rate
- User fatigue (subjective)
```

**Result:** Data on whether voice interfaces improve developer velocity with AI.

## IMPLEMENTATION PRIORITY

### Phase 1: Automated Tracking (HIGH ROI)
1. **Implement hooks from claude-code-hooks-mastery**
   - UserPromptSubmit → auto-log prompts
   - PostToolUse → auto-log tool usage
   - SessionStart → offer experiment tracking
   - **Effort:** 2-4 hours
   - **Value:** Eliminates manual logging completely

### Phase 2: Real-Time Dashboard (HIGH ROI)
2. **Deploy claude-code-hooks-multi-agent-observability**
   - Set up Bun server + Vue dashboard
   - Add CREATE SOMETHING experiment views
   - Integrate hooks to send data to dashboard
   - **Effort:** 4-6 hours
   - **Value:** Visualize experiments in real-time

### Phase 3: Multi-Model Testing (MEDIUM ROI)
3. **Integrate just-prompt MCP server**
   - Install MCP server
   - Create multi-model experiment templates
   - Run comparative experiments
   - **Effort:** 2-3 hours
   - **Value:** Data on model selection for specific tasks

### Phase 4: Database Migration (OPTIONAL)
4. **Move to Postgres storage**
   - Design schema
   - Migrate JSONL to SQL
   - Build query tools
   - **Effort:** 6-8 hours
   - **Value:** Cross-experiment analytics

### Phase 5: Voice Experiments (EXPERIMENTAL)
5. **Test voice-driven development**
   - Set up always-on-ai-assistant
   - Run controlled experiments
   - **Effort:** 4-6 hours
   - **Value:** Explore alternative input methods

## NEXT STEPS

1. **Start with Phase 1** (automated tracking via hooks)
   - Clone `claude-code-hooks-mastery`
   - Copy hook scripts to `.claude/hooks/`
   - Customize for experiment tracking
   - Test with a simple experiment

2. **Run your first automated experiment**
   - Build something small with Claude Code
   - Let hooks capture everything automatically
   - Review JSONL files to verify tracking works

3. **Deploy Phase 2** (dashboard)
   - Once hooks work, add real-time visualization
   - See experiments progress live
   - Use dashboard to identify patterns

4. **Document learnings**
   - Write a CREATE SOMETHING paper about building the tracking system itself
   - Meta-experiment: "Building experiment tracking with AI"

## CONCLUSION

These repos provide **production-ready infrastructure** for automating CREATE SOMETHING experiment tracking:

**Biggest wins:**
1. **claude-code-hooks-mastery** → Eliminates manual logging entirely
2. **claude-code-hooks-multi-agent-observability** → Real-time visibility into experiments
3. **just-prompt** → Multi-model comparative analysis

**Recommended starting point:** Phase 1 (hooks). Get automatic tracking working, then add visualization and multi-model testing.

This infrastructure turns CREATE SOMETHING from "manually documented experiments" into "fully automated scientific research on AI-native development."
