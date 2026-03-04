<div align="center">

# Epoch

### The AI Development Operating System

*The organizational runtime that makes your entire AI toolchain smarter — not just one agent at a time.*

</div>

---

Engineering teams are rapidly adopting AI workflow tools. Agents that interview you before starting. Session memory that persists decisions across context resets. Parallel subagents executing tasks in fresh, isolated contexts. Structured execution with hard gates before high-risk operations.

These tools work. The patterns behind them are real, and teams that adopt them move noticeably faster.

But they all share the same blind spot.

Every one of them operates within the scope of a session, a file, or a repository. None of them knows that the change they're executing will break a consumer three services away. None knows who owns the affected system, or that it's a Tier 1 service subject to a change freeze. None knows your deployment process, your compliance requirements, or the architectural decisions your team made eighteen months ago. They're capable within their scope — and completely blind outside it.

**The agents aren't missing capability. They're missing organizational context.**

And that context gap is what limits how autonomous AI can actually become in your engineering organization.

---

## The Missing Layer

A mature AI-assisted development stack looks something like this:

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

Every layer in this stack exists today as an open-source tool or pattern. The interview agent. The parallel subagents. The persistent session memory. The cross-session vector search.

What doesn't exist is the organizational layer in the middle: the component that knows the graph, enforces the policies, carries institutional knowledge into every agent execution, and gets smarter with every cycle.

That's Epoch.

---

## What Epoch Is

Epoch is the **organizational runtime** for AI-assisted development.

An operating system doesn't execute your programs — it provides the environment programs run within. Epoch is that, for AI agents operating inside an engineering organization. The agents are Claude, Cursor, Copilot, or whatever comes next. Epoch doesn't replace them or orchestrate them. It provides the substrate they operate on: the organizational context that turns a capable-but-narrow agent into one that can safely operate across your entire engineering surface.

Without Epoch, every agent starts from zero — no awareness of what it's touching, who owns it, what's downstream. With Epoch, every agent in the stack is grounded before it acts.

Four things compose the runtime:

---

### The OmniRepo Graph

The OmniRepo is not a monorepo. It's the next idea after monorepo.

The monorepo proved a thesis: co-locating code yields compounding benefits — atomic commits, unified CI, shared tooling, and the ability to reason across services in a single checkout. Teams that adopted monorepos well moved faster because of the visibility and cohesion they created.

The OmniRepo takes that same thesis and extends it in two directions. First, beyond code — infrastructure, API contracts, ownership, policies, and operational context belong in the same unified picture as the code they govern. Second, beyond a single repository — some repos are correctly kept separate (infrastructure carries different blast radius; GitOps config is watched directly by deployment systems). The OmniRepo is a virtual layer that spans as many repositories as your organization actually has. It doesn't mandate how you structure your source control. It connects what you have into a unified, queryable graph — so a single `epoch query` can reason across your application repos, your infrastructure, your API contracts, and your ownership declarations simultaneously.

This is also where Epoch and your agents live. The OmniRepo carries not just code and infrastructure, but the operational context that makes AI agents effective: service-level `CLAUDE.md` files that ground any agent touching that service, `OWNERS.epoch` files that declare ownership and tier classification, Skills that encode how your organization operates, and policies that define what's allowed. An agent arriving at any service in the OmniRepo finds everything it needs before writing a single line.

Some repositories belong inside the OmniRepo — application services, shared libraries, API contracts. Others are better kept separate:

| Repository type | Inside OmniRepo | Separate |
|---|---|---|
| Application services | ✓ | |
| Shared packages, APIs | ✓ | |
| Observability — monitoring, alerts, dashboards | ✓ | |
| Agent context, Skills, policies | ✓ | |
| Infrastructure (Terraform) | | ✓ — different blast radius, access controls |
| GitOps / ArgoCD config | | ✓ — deployment system watches this directly |

Epoch scans all of it — inside and out — and builds the graph. The boundary is about where code lives, not about what Epoch can see.

```bash
$ epoch scan
Discovered 14 services, 6 APIs, 3 databases, 2 K8s clusters, 47 packages

$ epoch query "what breaks if I remove GET /users/:id?"
  → auth-service (calls this endpoint in token validation)
  → admin-dashboard (calls this endpoint on user detail page)
  → mobile-api-gateway (proxies this route)

$ epoch owners user-service
  Team: platform-identity  |  Tier: 1  |  Status: active
  On-call: alice@company.com, bob@company.com
```

The graph is built by scanning what you already have: repositories, Kubernetes manifests, Terraform definitions, OpenAPI specs, protobuf schemas, package manifests, ownership files. No migration. No new workflow. It reads what exists — across all of it.

An agent with access to the graph can answer cross-repository questions before making a change — not by guessing, but by querying a model of what actually exists.

---

### Skills: Executable Institutional Knowledge

Engineering organizations know how to do things. Deploy a service. Rotate credentials. Deprecate an API. Scaffold a new service. Patch a vulnerability across ten repositories simultaneously. Respond to a production incident at 2am.

This knowledge exists — in senior engineers' heads, in wiki pages that lag reality, in Slack threads from six months ago. It's not executable. It's not versioned. And it doesn't transfer.

In Epoch, it becomes **Skills** — named, versioned capabilities your organization has formally defined and can execute. A Skill knows the graph it operates on. It respects the policies that govern it. It can be run by a human, triggered by CI, or handed to an AI agent. The same Skill a senior engineer runs manually today is the one an agent runs autonomously tomorrow.

Skills are the primary trust-building mechanism. Every operation encoded as a Skill is a surface area where AI can act consistently and auditably, without a human watching every step.

What they share: they encode *how your organization does things*, not just what tools exist. An agent executing a Skill isn't guessing — it's following the process your team defined, tested, and trusts.

---

### Policies: Encoded Guardrails

Autonomy without constraints is risk. Epoch's policy engine makes organizational guardrails executable — evaluated against proposed changes before they land, not discovered in post-mortems.

Policies express what your organization has decided, at a system level: a schema change to a shared database requires a migration in the same PR; a public API endpoint requires authentication; a change to a Tier 1 service during a freeze requires explicit override; a new dependency with a known critical CVE is blocked until reviewed.

Most engineering teams already hold these as conventions. The difference is that conventions are enforced by code review attention, tribal knowledge, and luck. Policies are enforced by the system — consistently, regardless of who opened the PR or how busy the reviewers are.

**Policies are what make it safe to extend AI autonomy.** The clearer the guardrails, the more confidently you can let agents roam within them. Policies run in CI, as pre-commit hooks, and as gates on Skill execution — starting in `warn` mode while a team builds confidence, escalating to `block` once trusted.

---

### Feedback: Production Reality in the Graph

A graph that only models static structure drifts from reality. Epoch closes this loop by ingesting production signals — observability, cost, deployment history — as first-class graph annotations.

SLO status, latency, error rates, cost trends, and recent deployment events attach to the graph nodes they belong to:

- An agent checks service health before operating on it
- A policy blocks deploys while error rate exceeds threshold
- A Skill verifies health at every step, not just at the end
- Impact analysis reflects current production state, not just topology

```bash
$ epoch health user-service
  SLO: 99.95% availability (current: 99.97%, budget: 62% remaining)
  P99 latency: 240ms (target: 300ms)
  Monthly cost: $2,847 (budget: $3,000, trend: +8% MoM)
  Last deploy: 3 hours ago — no anomalies detected
```

Production reality becomes a constraint the system enforces, not a dashboard engineers check after something breaks.

---

## Where Epoch Operates: Glue at Specific Junctions

Epoch is not a central orchestrator. The AI tooling landscape moves too fast — models, agents, and frameworks that are current today are superseded in months. Coupling Epoch to a specific generation of AI capability would be brittle.

Instead, Epoch operates as **glue between existing systems** — contributing organizational context at specific junctions where that context matters most.

**Code review augmentation** — An AI reviewer flags a pattern as non-standard. Epoch consults the graph, recognizes the pattern is an intentional architectural decision for that domain, and resolves the comment automatically — with a citation to the relevant policy. The reviewer didn't know; Epoch did.

**Observability gap detection** — After a change ships, Epoch checks whether the affected code paths have adequate monitoring coverage. If a new path isn't covered by existing alerts, Epoch surfaces the gap before it becomes an incident.

**Nightly validation scans** — Epoch launches lightweight workers on a schedule to verify graph accuracy: are declared owners still active? Are deprecated services still receiving traffic? Have API contracts drifted from their specs? Discrepancies become issues or PRs, not tribal knowledge.

**Cross-repo impact on every PR** — A developer opens a PR. Epoch computes graph-level impact across all repositories, annotates the PR with affected owners, policy results, and downstream risk — before any human reviews a line.

In each case, Epoch contributes what no single AI tool has: organizational context that spans the whole engineering surface, not just the open file.

---

## The Compounding Effect

Epoch's deepest value is not any single integration — it's what happens when the layers compound.

A session memory tool that can index against the Epoch graph surfaces relevant architectural decisions, not just conversation history. A subagent execution framework that consults Epoch before acting knows which services it's touching. An interview agent that queries Epoch before asking questions already knows what exists, and asks about what doesn't.

Each tool in the stack gets better because the organizational substrate beneath it is richer. Every Skill encoded makes the next agent more capable. Every policy defined makes the next execution safer. Every feedback cycle makes the graph more accurate. The machine gets better with every use.

The progression is incremental but the trajectory is clear: a flywheel where each delivery cycle produces not just the feature, but better infrastructure for the next feature.

---

## The Progression

Each layer increases the surface area where AI can operate independently.

| What you add | What agents can now do |
|---|---|
| Graph | Answer cross-repo impact questions; understand system topology |
| Ownership + tiers | Route changes appropriately; apply correct caution to critical systems |
| Policies | Evaluate changes against organizational rules automatically |
| Skills | Execute complex multi-step operations consistently and auditably |
| Feedback signals | Make decisions informed by production state; detect regressions |
| Agent integration | Operate across the full engineering surface within defined bounds |

Each layer is independently useful. None requires the next to deliver value. Adopt at the pace that matches your team's readiness — and apply different levels to different parts of your system simultaneously.

> The goal is not to push every system to full autonomy as fast as possible. The goal is to make the current level of AI autonomy in your organization **explicit, trustworthy, and improvable**. Every Skill you encode, every policy you define, every ownership boundary you declare narrows the gap between where agents operate today and where they could operate tomorrow.

---

## The Horizon: Full Autonomy

Epoch is built toward a specific horizon: an R&D organization where AI agents can receive high-level intent and produce working, production-grade outcomes without human involvement in the execution loop.

Not as a uniform state imposed everywhere at once — but as the ceiling any given system can reach, arrived at incrementally:

- A feature spec is submitted. An agent designs the approach, implements and tests it, routes for review based on ownership and tier, and merges when policies pass.
- A production incident fires. An agent diagnoses using graph-correlated observability, identifies the causative change, and executes a fix within the bounds of defined Skills and policies.
- A CVE is published. An agent queries the graph for all consumers of the affected dependency, opens targeted PRs, runs tests, and routes approvals — across every repository simultaneously.
- A quarterly infrastructure review runs automatically. An agent identifies waste, proposes changes, and stages them for human approval.

The humans in this picture are not absent. They define intent, evolve policies, build new Skills, and make decisions that require organizational judgment. The execution layer runs on its own.

---

## Design Principles

**Graph first.** The graph is the core of this system. Everything else is a projection of it. Accuracy of the graph is the highest-priority invariant.

**Glue, not god.** Epoch integrates with existing tools — version control, CI, observability, ticketing, AI assistants — rather than replacing them. The value is in connecting organizational context to these tools, not in owning the whole stack.

**Adopt, don't rebuild.** Tree-sitter for parsing. OPA or Cedar for policy. OpenTelemetry for signals. Epoch's value is the integration, not reimplementing solved problems.

**Value at every layer.** Each phase must be independently useful. If graph-only isn't worth adopting on its own, nothing built on top of it will be either.

**Gradual by design.** Teams, services, and products are not at the same point. Epoch supports heterogeneous autonomy levels within a single organization — and makes each system's current level explicit so it can be deliberately improved.

**AI-agnostic.** The graph, Skills, and policies Epoch maintains are valuable regardless of which AI tools your team uses. Works with today's tools and with whatever comes next.

**Dogfood immediately.** Epoch is used to build Epoch from the first working version onward.

---

## Open Source

Epoch is open source because an organizational runtime for AI-driven software development must be:

- **Inspectable** — trust requires transparency in how decisions are made and guardrails enforced
- **Forkable** — no organization should be locked into someone else's model of how engineering works
- **Composable** — the Skill and policy ecosystem grows through community contribution, not a single vendor's roadmap

---

## Where to Go Next

- **[ROADMAP.md](./ROADMAP.md)** — build phases and an incremental adoption path for a real engineering organization
- **[AI-LEVELS.md](./AI-LEVELS.md)** — how Epoch relates to the five levels of AI-assisted development, and why the L3→L4 transition is the core problem it solves
- **[WHAT-EPOCH-IS.md](./WHAT-EPOCH-IS.md)** — the reasoning behind Epoch's design: what it is, what it isn't, and why
