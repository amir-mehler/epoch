<div align="center">

# What Epoch Is

*A design note on the nature of the system.*

</div>

---

Before writing code, it's worth being precise about what Epoch actually is — because the answer shapes every architectural decision that follows.

Three reasonable models come to mind. Only one is right.

---

## Model 1: Distributed Context ("CLAUDE.md Everywhere")

The idea: place rich context files throughout your repository tree — one per service, one per directory — so that any AI agent crawling the repo finds local guidance wherever it lands.

This is appealing because it's simple, requires no new infrastructure, and works with every existing AI tool today.

But it solves the wrong problem.

Context files tell agents *how to behave locally*. They're instructions. An agent working on `services/billing/` with an excellent local context file still has no idea that the change it's making breaks `services/auth/` three directories away. Local guidance doesn't produce cross-system reasoning. There's no enforcement mechanism. There's no versioning of process. There's no way to express "Tier 1 services require two-person approval" as a machine-readable constraint.

This is a better Level 3 experience. It is not a path to Level 4.

---

## Model 2: Always-On Brain

The idea: a model with full organizational context running continuously, acting as the guiding intelligence for smaller worker-agent instances that execute discrete tasks.

This is architecturally appealing — it mirrors how high-performing human engineering teams work, with senior engineers providing direction and juniors executing.

But "full context always running" is the expensive, fragile version of what you actually need. For most questions an agent needs answered — "what depends on this service?", "who owns this API?", "what's the deployment policy for Tier 1 systems?" — you don't need inference. You need a **query**. A model burning context tokens to answer "what are the consumers of user-service?" is a $50 answer to a $0.001 question.

More importantly, a running model is not the right primitive for something that needs to be versioned, diffable, auditable, and correct. A graph database with an API is.

The AI is the brain. Epoch is not the brain. Epoch is what the brain queries.

---

## Model 3: Graph Translator

The idea: Epoch scans your repositories and infrastructure, produces a queryable graph of your system, and that graph is what AI engines and humans use to navigate and make decisions.

This is closest. But "translator" undersells the role.

A translator converts one representation to another and its job is done. Epoch's graph needs to stay accurate over time — derived continuously from source, not maintained by hand. It needs to be live, not a one-time snapshot. And the graph alone isn't sufficient: you need the layer that encodes *how your organization operates* (Skills, procedures) and the layer that encodes *what your organization will and won't allow* (policies). The graph is the foundation, not the whole system.

---

## What Epoch Actually Is: The Organizational Runtime

An operating system doesn't execute your programs. It provides the **environment** — memory management, filesystem, process model, system calls — that programs run within. Programs are more capable, more reliable, and safer because of the runtime beneath them, not despite it.

Epoch is that, for AI agents operating inside an engineering organization.

The AI agents are the worker ants — Claude, Cursor, whatever model is current or next. Epoch doesn't replace them, compete with them, or try to orchestrate them centrally. It provides the **organizational runtime** they operate within:

- **The graph** — a live, versioned, queryable model of what your system actually looks like: services, APIs, infrastructure, data stores, ownership, lifecycle states, and their relationships
- **Skills** — the operations your organization knows how to perform, encoded as versioned, graph-aware, executable procedures (not documentation — executable artifacts)
- **Policies** — the constraints that define what agents are allowed to do, at what tier, in what context, enforced at evaluation time not discovered in post-mortems
- **Feedback** — production signals (observability, cost, deployment history) flowing back into the graph so the runtime reflects what's actually happening, not just what's declared

An agent with access to this runtime isn't smarter. It's *grounded*. It can't accidentally break a contract it doesn't know exists. It doesn't have to guess your deployment process. It operates within a defined set of constraints and has access to a complete model of the system it's touching.

---

## The Compounding Stack

Epoch doesn't exist in isolation. It's the substrate beneath a stack of workflow tools that are already mature and already being adopted.

```
The developer describes intent
   ↓
An interview agent refines the spec, removes ambiguity, surfaces assumptions
   ↓
Epoch graph consulted — blast radius mapped, owners identified, policies evaluated
   ↓
Subagents execute task by task — fresh context, no pollution, hard gates before risk
   ↓
Session memory tracks state — goal drift prevented, plan stays in the attention window
   ↓
Cross-session memory persists decisions — the next session starts informed
   ↓
Epoch feedback loop — production signals flow back into the graph
   ↓
The next feature starts better than the last
```

Every layer in this stack exists today as an open-source tool or pattern. The interview-first agent. The parallel subagents in fresh isolated contexts. The session memory that re-reads the plan before every major action. The vector search over past decisions.

What each of these tools lacks is the organizational layer in the middle. They operate within their session, their context window, their repository. None knows what it's touching at the organizational level. None carries institutional knowledge across teams and time.

Epoch is what that middle layer looks like.

The key property: Epoch doesn't just enable the stack — it makes every layer in the stack smarter. An interview agent that can query the graph before asking questions already knows what services exist, who owns them, and what the blast radius is — so it asks about what it doesn't know instead of reconstructing what it could. A session memory tool that indexes against the Epoch graph surfaces relevant architectural decisions, not just conversation history. A subagent execution framework that checks Epoch before acting knows it won't accidentally break a contract it wasn't told about.

Without Epoch, each tool in the stack is working from a local, session-scoped picture of the world. With Epoch, every tool in the stack is working from the same organizational ground truth.

The flywheel is that this gets better with use. Every Skill encoded is institutional knowledge that future agents can leverage. Every policy defined is a guardrail that future executions stay within automatically. Every feedback cycle makes the graph more accurate. The machine improves with every rep — and that improvement accumulates across teams, repositories, and time.

---

## Where Epoch Sits: Glue, Not God

Epoch is not the orchestration layer. This is a deliberate choice.

The AI tooling landscape moves fast — models, agents, and frameworks that are state-of-the-art today are superseded within months. Building Epoch as a central orchestrator would mean coupling it to a specific generation of AI capability and a specific set of tools. That's not durable.

Instead, Epoch is **glue between existing systems** — with targeted coordination at specific, well-defined junctions where organizational context matters most.

Examples of what this looks like in practice:

**Code review integration** — An AI code reviewer (GitHub Copilot, Claude, etc.) flags a pattern as non-standard. Epoch consults the graph and recognizes the pattern is an intentional architectural decision for that domain — and resolves the comment automatically, with a citation to the relevant policy. The reviewer didn't know; Epoch did.

**Observability gap detection** — After a change ships, Epoch queries the graph to verify that the affected services have adequate monitoring coverage. If a new code path isn't covered by existing alerts, Epoch surfaces the gap — before it becomes an incident.

**Nightly validation scans** — Epoch launches lightweight agents on a schedule to verify graph accuracy: are declared owners still active? Are deprecated services still receiving traffic? Have any API contracts drifted from their specs? Discrepancies become issues or PRs, not tribal knowledge.

**Cross-repo impact on PRs** — A developer opens a PR. Epoch computes the graph-level impact across all repositories — not just the one being changed — and annotates the PR with affected owners, policy checks, and downstream risk before any human has reviewed a line.

In each case, Epoch is contributing organizational context that no single AI tool has — not because Epoch is smarter, but because Epoch is the system that maintains that context.

**This positioning is also more durable.** The graph, Skills, and policies that Epoch maintains are valuable regardless of which AI tools your team uses. They work with Claude today, and with whatever comes next. Epoch's value compounds as the graph grows and Skills accumulate — not as any particular AI capability improves.

---

## On the Term "Skills"

Epoch uses **Skills** for its executable organizational knowledge layer. This term also appears in *Claude Code* — where it refers to user-invocable prompt extensions for the Claude Code CLI itself. The concepts are distinct.

Epoch Skills are organizational artifacts: versioned, graph-aware, policy-bound procedures that encode how specific operations are performed *in your organization*. They live in your repository, evaluate against your graph, and are executable by humans or AI agents. They are institutional knowledge made concrete — not AI tool configuration.

Claude Code Skills are extensions to Claude Code's own behavior — custom slash commands a user defines to change how the AI assistant responds. Different layer, different purpose.

Where the distinction matters in this document: when you read "Skill", it means an Epoch organizational capability. When you read "Claude Code", it refers to the CLI tool and its own feature set.

---

## The One-Paragraph Version

Epoch is the organizational runtime for AI-assisted software development. It maintains a live, versioned graph of your engineering organization — services, APIs, infrastructure, ownership, lifecycle states, policies, and production signals — and exposes it as a queryable layer that AI agents, humans, and CI/CD systems can use to make grounded decisions. Epoch is not the AI. It is the environment the AI operates within: the context that turns a capable-but-narrow agent into one that can safely operate across your entire engineering surface.
