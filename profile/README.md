<p align="center"><img src="../AGNT5.png" alt="AGNT5: the reliability platform for AI agents and workflows" width="100%"></p>

**The reliability platform for AI agents and workflows.** Build, run, and continuously improve your agents on one runtime.

Agents fail in three ways. A tool call errors out. Every step succeeds but the answer is still wrong. Or the server dies mid-run. Each of these usually shows up in a different tool: tracing, evals, a job queue, an infra alert. Nothing shares a run id, so you join it all with your own retry wrappers and replay scripts. AGNT5 runs your agent itself, so every run is a single record that holds its steps, retries, crash and resume, eval score, and cost.

[Website](https://agnt5.com) · [Docs](https://agnt5.com/docs) · [Start free](https://app.agnt5.com)

---

## How it works

**Build: agents that survive failure.**
- **Everything you need to build an agent system.** Write functions, workflows, agents, tools, and MCP servers with the same SDK, in Python, TypeScript, or Go. A tool or an MCP server can itself be a durable workflow.
- **Durable by design.** Every step is checkpointed. If a run crashes on step 5, it resumes at step 5, and steps that already completed are read back from the record instead of re-run.
- **Clear execution guarantees.** Steps run at least once. Add idempotency keys to steps that have side effects and you get effectively-once execution: a payment is charged once, an email is sent once.
- **Retries and human approvals built in.** Failed calls to flaky tools, APIs, and models retry with backoff. A run can pause for hours or days waiting on a person, then continue from that step.

**Run: deploy, observe, and get a verdict on every release.**
- **Deploy with one command.** `agnt5 deploy` ships your agents as a deployment, with workers that scale with load. There are no queues, workers, or databases for you to stand up.
- **Multiple environments.** Keep dev, staging, and production separate, each with its own deployments and secrets. Promote a deployment from one environment to the next, or roll it back, in one step. Preflight checks run before every deploy.
- **Observability built in.** Every run is traced step by step, with inputs, outputs, logs, retries, tokens, and cost, and you don't add any instrumentation. Fleet views show runs, errors, latency, and cost across every deployment.
- **A verdict on every deploy.** After each deploy, AGNT5 compares the new version with the previous one on live runs: error rate, p95 latency, cost per run, and eval score. The verdict is better, worse, or no change.

**Improve: find what's wrong, fix it, and prove the fix.**
- **Evals.** Attach scorers to your agents and score live production runs, not only a test set. A score is stored with the run it belongs to, alongside the evidence behind it.
- **Experiments.** Run a prompt, model, or code change against a dataset and compare it side by side with the current version before you ship. Failed production runs can be saved into datasets.
- **Command center.** AGNT5 groups failures into issues, such as tool timeouts after a deploy or a runtime fault, and links the runs that show each one. Every issue comes with a likely cause and the evidence for it. AGNT5 points at the problem, and you decide on the fix.
- **Replay to prove it.** Replay the failing runs with the same inputs and tool responses against your change, then score old against new on those same runs. Ship when the fix wins, and the deploy verdict confirms it in production.

```python
from agnt5 import function, workflow, FunctionContext, WorkflowContext

@function(retries=3, backoff="exponential")
async def issue_refund(ctx: FunctionContext, order_id: str) -> str:
    ...  # flaky payment API: retried on failure

@workflow
async def refund_order(ctx: WorkflowContext, order_id: str) -> dict:
    order = await ctx.step(fetch_order, order_id)            # checkpointed
    decision = await ctx.wait_for_user(                      # can wait for days
        question=f"Refund {order['total']}?", input_type="approval",
        options=[{"id": "approve", "label": "Approve"}, {"id": "reject", "label": "Reject"}],
    )
    if decision == "reject":
        return {"status": "declined"}
    await ctx.step(issue_refund, order_id)                   # skipped on resume once it has completed
    return {"status": "refunded"}
```

## Bring the agents you already have

You don't need to rewrite anything. Wrap your agent's entry point in an AGNT5 `@function` and keep calling your framework the way you do today. Prompts, tools, and model providers stay as they are.

Works with the OpenAI Agents SDK, Claude Agent SDK, Google ADK, Vercel AI SDK, and Cloudflare Workers.

```bash
curl -LsSf https://agnt5.com/cli.sh | bash   # or: brew install agnt5dev/agnt5/agnt5
agnt5 create --template <template-name> my-agent
agnt5 deploy
```

## Run it in our cloud or yours

Use AGNT5 fully managed, or run it inside your own environment so prompts, outputs, and customer data never leave your network. Build, Run, and Improve work the same way in both, deploy verdicts included.

---

## Repositories

| Repo | What it is |
|---|---|
| [sdk-python](https://github.com/agnt5dev/sdk-python) · [sdk-typescript](https://github.com/agnt5dev/sdk-typescript) · [sdk-go](https://github.com/agnt5dev/sdk-go) | Language SDKs for building agents and workflows on AGNT5 |
| [sdk-core](https://github.com/agnt5dev/sdk-core) | Shared Rust core behind the SDKs: worker lifecycle, checkpoints, providers, scorers, telemetry |
| [runtime](https://github.com/agnt5dev/runtime) | Open-source single-node runtime, one binary backed by PostgreSQL, for local and self-hosted use |
| [templates](https://github.com/agnt5dev/templates) | Starter templates in Python, TypeScript, and Go |
| [skills](https://github.com/agnt5dev/skills) | Agent Skill that teaches coding agents (Claude Code, Codex, …) to build on AGNT5 |
| [homebrew-agnt5](https://github.com/agnt5dev/homebrew-agnt5) | Homebrew tap for the AGNT5 CLI |

Questions or feedback: [info@agnt5.com](mailto:info@agnt5.com) · [Talk to an engineer](https://agnt5.com)
