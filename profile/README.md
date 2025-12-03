# 👋 Hi, we’re AGNT5

**AGNT5 is the durable runtime and observability layer for AI workflows and agents.**

We help teams move from brittle PoCs to production systems by handling the hard parts:
durability, retries, state, traces, and evals — so your workflows recover instead of fail.

---

## What you can build with AGNT5

- **Durable AI workflows**  
  Long-running, multi-step flows that survive crashes, restarts, and deploys.

- **Agents with tools & human handoffs**  
  Orchestrate LLM calls, tools, APIs, and human approvals as one workflow.

- **Context & RAG pipelines**  
  Ingestion, enrichment, chunking, retrieval, and re-ranking with retries and state built-in.

- **Evals & experiments in production**  
  Compare prompts, models, and configs using real traffic, not just notebook data.

---

## Core building blocks

AGNT5 gives you production primitives most teams end up rebuilding:

- **Durable runtime** – turn regular async functions into workflows.
- **State & idempotency** – persist state between steps; safely retry work.
- **Retries & schedules** – declarative retry policies and cron-like schedules.
- **Signals & human approvals** – wait for external events or approvals as first-class steps.
- **Observability** – traces, logs, and metrics for every run.
- **Eval hooks** – attach evals to workflows to catch regressions early.

---

## Quick taste (Python)

```python
from agnt5 import durable, Context

@durable("triage_ticket", retries=5)
async def triage(ctx: Context, ticket: dict):
    payload = {"role": "user", "content": ticket["text"]}

    # Step 1: draft a reply with an LLM
    draft = await ctx.step("draft", lambda: ctx.llm.chat(messages=[payload]))

    # Step 2: wait for human approval (can pause for hours or days)
    decision = await ctx.signal.wait("agent_approval", timeout="24h")
    if decision != "approve":
        return "held"

    # Step 3: send the email
    await ctx.step("send", lambda: ctx.email.send(
        to=ticket["user"],
        body=draft.text,
    ))

    return "sent"
