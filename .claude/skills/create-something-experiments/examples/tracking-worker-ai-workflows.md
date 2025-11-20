# Example: Tracking Cloudflare Worker AI and Workflows Usage

This example shows how to track experiments that heavily use **Cloudflare Worker AI** and **Workflows** - capturing inference costs, execution patterns, and orchestration metrics.

## Why This Matters

Worker AI and Workflows have distinct cost models and usage patterns:

- **Worker AI:** Pay per inference (varies by model)
- **Workflows:** Pay per step execution and duration
- **Combined:** Need to track both to understand total project economics

## Scenario: AI-Powered Content Pipeline

Building an agentic content pipeline that:
- Uses Worker AI for embeddings and summarization
- Orchestrates multi-step processing with Workflows
- Stores results in D1 and R2

## Tracking Structure

```bash
.claude/experiments/ai-content-pipeline/
├── log.md
├── prompts.jsonl
├── metrics.jsonl
├── errors.log
├── interventions.log
├── worker-ai.jsonl          # Worker AI specific tracking
├── workflows.jsonl          # Workflows specific tracking
├── cost-breakdown.jsonl     # Combined cost analysis
└── PAPER.md
```

## Step 1: Track Worker AI Inferences

### During Development

Log every Worker AI call with cost estimates:

```bash
# worker-ai.jsonl format
{"timestamp":"2025-01-14T10:30:00Z","iteration":5,"model":"@cf/meta/llama-3.1-8b-instruct","operation":"summarization","input_tokens":1200,"output_tokens":150,"cost_estimate":"$0.0135","latency_ms":850,"success":true,"context":"summarizing blog post"}

{"timestamp":"2025-01-14T10:31:00Z","iteration":5,"model":"@cf/baai/bge-base-en-v1.5","operation":"embedding","input_tokens":500,"dimensions":768,"cost_estimate":"$0.0005","latency_ms":120,"success":true,"context":"generating embedding for semantic search"}

{"timestamp":"2025-01-14T10:35:00Z","iteration":7,"model":"@cf/meta/llama-3.1-70b-instruct","operation":"content_generation","input_tokens":800,"output_tokens":600,"cost_estimate":"$0.08","latency_ms":2400,"success":true,"context":"generating article outline"}
```

### Cost Tracking Script

```typescript
// src/utils/track-worker-ai.ts
export async function trackWorkerAI(
  operation: string,
  model: string,
  inputTokens: number,
  outputTokens: number,
  success: boolean,
  context: string
) {
  const costs = {
    '@cf/meta/llama-3.1-8b-instruct': { input: 0.00001, output: 0.00001 },
    '@cf/meta/llama-3.1-70b-instruct': { input: 0.0001, output: 0.0001 },
    '@cf/baai/bge-base-en-v1.5': { input: 0.000001, output: 0 },
  }

  const pricing = costs[model]
  const cost = (inputTokens * pricing.input) + (outputTokens * pricing.output)

  const logEntry = {
    timestamp: new Date().toISOString(),
    model,
    operation,
    input_tokens: inputTokens,
    output_tokens: outputTokens,
    cost_estimate: cost.toFixed(6),
    success,
    context
  }

  // Log to JSONL file (in development)
  await appendToFile('.claude/experiments/ai-content-pipeline/worker-ai.jsonl',
    JSON.stringify(logEntry) + '\n'
  )
}
```

### Usage in Worker

```typescript
// src/worker.ts
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const content = await request.text()

    // Generate embedding with tracking
    const startTime = Date.now()
    const { data } = await env.AI.run('@cf/baai/bge-base-en-v1.5', {
      text: content
    })
    const latency = Date.now() - startTime

    // Track the inference
    await trackWorkerAI(
      'embedding',
      '@cf/baai/bge-base-en-v1.5',
      estimateTokens(content),
      0, // embeddings don't have output tokens
      true,
      'generating content embedding'
    )

    return Response.json({ embedding: data })
  }
}
```

## Step 2: Track Workflows Executions

### During Development

Log workflow runs with step-level detail:

```bash
# workflows.jsonl format
{"timestamp":"2025-01-14T10:40:00Z","iteration":8,"workflow_name":"content_pipeline","execution_id":"exec_abc123","duration_ms":45000,"steps":12,"steps_succeeded":12,"steps_failed":0,"cost_estimate":"$0.02","success":true,"context":"processing uploaded PDF"}

{"timestamp":"2025-01-14T11:15:00Z","iteration":12,"workflow_name":"content_pipeline","execution_id":"exec_def456","duration_ms":120000,"steps":15,"steps_succeeded":13,"steps_failed":2,"cost_estimate":"$0.05","success":false,"error":"step_timeout on embedding_generation","context":"processing large document"}
```

### Workflow Definition with Tracking

```typescript
// src/workflows/content-pipeline.ts
import { WorkflowEntrypoint, WorkflowStep, WorkflowEvent } from 'cloudflare:workers'

export class ContentPipeline extends WorkflowEntrypoint {
  async run(event: WorkflowEvent<{ documentId: string }>, step: WorkflowStep) {
    const startTime = Date.now()
    const executionId = event.id
    const steps = []

    try {
      // Step 1: Fetch document
      const document = await step.do('fetch_document', async () => {
        steps.push({ name: 'fetch_document', status: 'success', duration_ms: 0 })
        return await this.env.DB.prepare(
          'SELECT content FROM documents WHERE id = ?'
        ).bind(event.payload.documentId).first()
      })

      // Step 2: Generate summary with Worker AI
      const summary = await step.do('generate_summary', async () => {
        const result = await this.env.AI.run('@cf/meta/llama-3.1-8b-instruct', {
          messages: [
            { role: 'system', content: 'Summarize the following document concisely.' },
            { role: 'user', content: document.content }
          ]
        })

        // Track Worker AI usage
        await trackWorkerAI(
          'summarization',
          '@cf/meta/llama-3.1-8b-instruct',
          estimateTokens(document.content),
          estimateTokens(result.response),
          true,
          `workflow ${executionId} - document summarization`
        )

        steps.push({ name: 'generate_summary', status: 'success', duration_ms: 0 })
        return result.response
      })

      // Step 3: Generate embeddings
      const embedding = await step.do('generate_embedding', async () => {
        const result = await this.env.AI.run('@cf/baai/bge-base-en-v1.5', {
          text: summary
        })

        await trackWorkerAI(
          'embedding',
          '@cf/baai/bge-base-en-v1.5',
          estimateTokens(summary),
          0,
          true,
          `workflow ${executionId} - embedding generation`
        )

        steps.push({ name: 'generate_embedding', status: 'success', duration_ms: 0 })
        return result.data
      })

      // Step 4: Store results
      await step.do('store_results', async () => {
        await this.env.DB.prepare(
          'UPDATE documents SET summary = ?, embedding = ? WHERE id = ?'
        ).bind(summary, JSON.stringify(embedding), event.payload.documentId).run()

        steps.push({ name: 'store_results', status: 'success', duration_ms: 0 })
      })

      // Track successful workflow execution
      const duration = Date.now() - startTime
      await trackWorkflow(
        executionId,
        'content_pipeline',
        duration,
        steps.length,
        steps.filter(s => s.status === 'success').length,
        0,
        true,
        null
      )

    } catch (error) {
      // Track failed workflow execution
      const duration = Date.now() - startTime
      await trackWorkflow(
        executionId,
        'content_pipeline',
        duration,
        steps.length,
        steps.filter(s => s.status === 'success').length,
        steps.filter(s => s.status === 'failed').length,
        false,
        error.message
      )

      throw error
    }
  }
}
```

## Step 3: Combined Cost Analysis

### Track Total Experiment Costs

```bash
# cost-breakdown.jsonl
{"timestamp":"2025-01-14T23:59:59Z","period":"day_1","worker_ai_cost":"$1.45","workflows_cost":"$0.82","d1_cost":"$0.03","kv_cost":"$0.01","r2_cost":"$0.05","claude_code_cost":"$3.20","total_cost":"$5.56"}
```

### Daily Cost Calculator

```bash
#!/bin/bash
# Calculate daily costs across all Cloudflare resources

EXPERIMENT_DIR=".claude/experiments/ai-content-pipeline"
DATE=$(date +%Y-%m-%d)

# Worker AI costs
WORKER_AI_COST=$(jq -s --arg date "$DATE" '
  map(select(.timestamp | startswith($date)) | .cost_estimate | tonumber) | add
' "$EXPERIMENT_DIR/worker-ai.jsonl")

# Workflows costs
# Pricing: $0.30 per million step-operations, $0.02 per million GB-s
WORKFLOWS_COST=$(jq -s --arg date "$DATE" '
  map(select(.timestamp | startswith($date))) |
  length as $executions |
  (map(.steps) | add) as $total_steps |
  (map(.duration_ms) | add) / 1000 as $total_seconds |
  ($total_steps * 0.30 / 1000000) + ($total_seconds * 0.128 * 0.02 / 1000000)
' "$EXPERIMENT_DIR/workflows.jsonl")

# D1 costs (from separate tracking)
D1_COST=$(jq -s --arg date "$DATE" '
  map(select(.timestamp | startswith($date)) | .cost_estimate | tonumber) | add
' "$EXPERIMENT_DIR/cloudflare.jsonl" || echo "0")

# Total
TOTAL=$(echo "$WORKER_AI_COST + $WORKFLOWS_COST + $D1_COST" | bc)

echo "{\"timestamp\":\"${DATE}T23:59:59Z\",\"period\":\"day_1\",\"worker_ai_cost\":\"\$$WORKER_AI_COST\",\"workflows_cost\":\"\$$WORKFLOWS_COST\",\"d1_cost\":\"\$$D1_COST\",\"total_cost\":\"\$$TOTAL\"}" >> "$EXPERIMENT_DIR/cost-breakdown.jsonl"

echo "Daily Costs ($DATE):"
echo "  Worker AI: \$$WORKER_AI_COST"
echo "  Workflows: \$$WORKFLOWS_COST"
echo "  D1: \$$D1_COST"
echo "  Total: \$$TOTAL"
```

## Step 4: Production Analytics (Retroactive)

### Fetch Actual Worker AI Usage

```bash
#!/bin/bash
# Fetch production Worker AI analytics from Cloudflare

ACCOUNT_ID="your-account-id"
API_TOKEN="your-api-token"

# Get AI inference logs (last 30 days)
curl -X GET "https://api.cloudflare.com/client/v4/accounts/${ACCOUNT_ID}/ai/runs/logs?per_page=1000" \
  -H "Authorization: Bearer ${API_TOKEN}" \
  | jq '.result[] | {
      timestamp: .timestamp,
      model: .model,
      input_tokens: .usage.input_tokens,
      output_tokens: .usage.output_tokens,
      success: .success,
      latency_ms: .latency_ms
    }' \
  > .claude/experiments/ai-content-pipeline/production-worker-ai.json

# Calculate actual costs
jq -s '
  group_by(.model) |
  map({
    model: .[0].model,
    total_inferences: length,
    total_input_tokens: (map(.input_tokens) | add),
    total_output_tokens: (map(.output_tokens) | add),
    avg_latency_ms: (map(.latency_ms) | add / length),
    success_rate: ((map(select(.success == true)) | length) / length * 100)
  })
' production-worker-ai.json
```

### Fetch Workflows Execution History

```bash
# Get Workflows analytics
curl -X GET "https://api.cloudflare.com/client/v4/accounts/${ACCOUNT_ID}/workflows/instances" \
  -H "Authorization: Bearer ${API_TOKEN}" \
  | jq '.result[] | {
      id: .id,
      workflow_name: .workflow_name,
      created_at: .created_at,
      status: .status,
      steps_total: .metadata.steps_total,
      steps_succeeded: .metadata.steps_succeeded,
      duration_ms: .duration_ms
    }' \
  > .claude/experiments/ai-content-pipeline/production-workflows.json
```

## Step 5: Generate Paper with Worker AI/Workflows Focus

```markdown
# Experiment #X: Building AI Content Pipeline with Worker AI + Workflows

## THE EXPERIMENT

### The Problem
Processing large volumes of content (PDFs, articles) requires:
- Summarization (AI inference)
- Embedding generation (AI inference)
- Multi-step orchestration
- Reliable execution at scale

### The Hypothesis
**I hypothesized that:** Using Cloudflare Worker AI + Workflows would be 10x cheaper than external AI APIs (OpenAI) while maintaining comparable quality, but might have higher latency.

### Why This Matters
Most AI pipelines use external APIs ($$$) and complex orchestration (Lambda + Step Functions). Testing if Cloudflare's integrated stack can replace this at lower cost/complexity.

## WHAT I MEASURED

### Success Criteria
- [ ] Process 1,000 documents/day reliably
- [ ] Average latency < 5 seconds per document
- [ ] Cost < $0.10 per document
- [ ] Accuracy comparable to GPT-4o
- [ ] 99%+ workflow success rate

### Metrics Tracked
- **Worker AI usage:** Inferences, tokens, models, latency, costs
- **Workflows execution:** Steps, duration, success rate, retry patterns
- **End-to-end cost:** Worker AI + Workflows + D1 + R2
- **Quality:** Human evaluation of summaries vs GPT-4o baseline

## RESULTS: THE BUILD PROCESS

### Worker AI Performance (Production - 30 days)

| Model | Inferences | Avg Tokens (In/Out) | Avg Latency | Cost | Success Rate |
|-------|-----------|---------------------|-------------|------|-------------|
| llama-3.1-8b | 45,230 | 1,200 / 150 | 850ms | $6.10 | 99.2% |
| llama-3.1-70b | 8,450 | 800 / 600 | 2,400ms | $13.52 | 98.8% |
| bge-base-en-v1.5 | 53,680 | 500 / 0 | 120ms | $0.27 | 99.9% |
| **Total** | **107,360** | - | - | **$19.89** | **99.3%** |

### Workflows Performance (Production - 30 days)

- **Total executions:** 42,890
- **Success rate:** 98.2%
- **Average duration:** 3.2 seconds
- **Average steps per execution:** 8
- **Failed executions:** 772 (mostly timeouts on large PDFs)
- **Total cost:** $12.87

### Cost Breakdown (Per 1,000 Documents)

| Resource | Cost | % of Total |
|----------|------|-----------|
| Worker AI (summarization) | $0.135 | 45% |
| Worker AI (embeddings) | $0.005 | 2% |
| Workflows orchestration | $0.30 | 30% |
| D1 storage/queries | $0.012 | 4% |
| R2 document storage | $0.008 | 3% |
| Claude Code (development) | $0.048 | 16% |
| **Total** | **$0.508** | **100%** |

### Comparison to OpenAI Stack

| Metric | Cloudflare Stack | OpenAI + AWS |
|--------|-----------------|--------------|
| Cost per 1K docs | $0.51 | $8.20 |
| Avg latency | 3.2s | 2.1s |
| Setup complexity | Low (single platform) | High (API + Lambda + S3) |
| Vendor lock-in | Moderate | Moderate |

**Result:** 93.8% cost reduction, 52% higher latency

## WHAT I (CLAUDE CODE) DID WELL

### 1. Worker AI Integration Patterns

**Generated inference wrapper with retry logic:** `src/utils/worker-ai.ts:12-45`

```typescript
export async function runInference<T>(
  ai: Ai,
  model: string,
  input: any,
  options: { maxRetries: number } = { maxRetries: 3 }
): Promise<T> {
  let lastError: Error

  for (let i = 0; i < options.maxRetries; i++) {
    try {
      const result = await ai.run(model, input)
      return result as T
    } catch (error) {
      lastError = error
      if (error.message.includes('rate_limit')) {
        await sleep(1000 * (i + 1)) // Exponential backoff
        continue
      }
      throw error
    }
  }

  throw lastError
}
```

This retry logic caught rate limits gracefully. Would have taken me 30+ minutes to write manually.

### 2. Workflow Step Decomposition

**Generated workflow with proper error boundaries:** `src/workflows/content-pipeline.ts:20-80`

Claude correctly identified that each AI operation should be a separate step (not combined) to allow Workflows to retry individual failures without re-running expensive AI inferences.

## WHERE USER INTERVENTION WAS NEEDED

### Issue #1: Worker AI Model Selection
- **Iteration:** 8
- **What happened:** Claude defaulted to llama-3.1-8b for all tasks
- **User intervention:** Manually tested llama-3.1-70b for complex summarization, found 15% quality improvement
- **Time cost:** 2 hours testing different models
- **Learning:** AI can't evaluate AI model quality - need human benchmarking

### Issue #2: Workflow Timeout Tuning
- **Iteration:** 15
- **What happened:** Large PDFs (>50 pages) timing out after 30s
- **User intervention:** Increased step timeout to 120s, added progress tracking
- **Time cost:** 1 hour debugging + tuning
- **Learning:** Claude doesn't know your data distribution - need real-world testing

## HONEST ASSESSMENT

### What This Proves
✅ Worker AI + Workflows can replace OpenAI + AWS for batch processing at 93% cost reduction
✅ Cloudflare's integrated stack significantly reduces complexity
✅ Latency tradeoff (52% slower) is acceptable for async workloads

### What This Doesn't Prove
❌ Quality matches GPT-4o (only compared to 3.5, need rigorous eval)
❌ Approach works for real-time use cases (latency may be prohibitive)
❌ Scales beyond 100K docs/day (didn't test at that scale)

### When to Build This Way

**Use Worker AI + Workflows when:**
- Batch processing (not real-time)
- Cost is primary concern
- Want single-platform deployment
- Content processing (summarization, embeddings, extraction)

**Don't use Worker AI + Workflows when:**
- Need GPT-4-class reasoning (quality gap)
- Real-time latency critical (<1s requirement)
- Need bleeding-edge models (Cloudflare catalog smaller)

## CONCLUSION

**Hypothesis outcome:** Validated with caveats

Built AI content pipeline using Worker AI + Workflows at 93.8% cost reduction vs OpenAI/AWS, but 52% higher latency. For async batch processing, the tradeoff is worth it.

**Key learning:** Cloudflare's AI stack is production-ready for cost-sensitive workloads, but requires quality evaluation against your specific use case.

**Next experiment:** Test Worker AI quality rigorously against OpenAI on industry-standard benchmarks to quantify the quality/cost tradeoff.
```

## Key Takeaways for Worker AI/Workflows Tracking

1. **Track at the inference level:** Every Worker AI call should log model, tokens, latency, cost
2. **Track at the workflow level:** Execution ID, steps, duration, failures
3. **Combine for total cost:** Worker AI + Workflows + storage = real project economics
4. **Use production analytics:** Cloudflare API provides actual usage data for retroactive analysis
5. **Model selection matters:** Claude can't predict which AI model will perform best - need testing
6. **Quality evaluation required:** Cost savings mean nothing if output quality is unacceptable

This tracking methodology gives you complete visibility into the AI pipeline economics and helps write papers that prove/disprove the value of building on Cloudflare's AI stack.
