<div align="center">

# Epoch

### The AI Development Operating System

*The organizational runtime that lets AI agents understand, navigate, and operate across your entire engineering organization — not just one repository at a time.*

</div>

---

AI coding tools are good. Most engineering teams are already using them.

But they're all doing the same thing: AI works on one file, one repository, one task at a time. A fast and capable worker — but narrow. The AI doesn't know what it doesn't know, and what it doesn't know is almost everything about your organization.

Which services depend on the change it's making.
Whether that change violates an architectural rule your team agreed on two years ago.
Who owns the affected system, and how critical it is.
What your actual deployment process looks like — not the generic one, *yours*.
Which APIs are stable contracts and which are internal details that will change next quarter.
What migrations are in progress, what systems are being deprecated, what decisions have already been made.

So engineers compensate. They provide the organizational context manually. They do the cross-repo impact assessment, the ownership lookup, the policy check. The AI generates; the human coordinates.

**This doesn't scale. And it puts a ceiling on how autonomous AI can become in your organization.**

Epoch removes that ceiling — by building the organizational context layer that AI agents are missing.

---

## The Problem Is Context, Not Capability

The gap between "AI writes code I review" and "AI ships features while I sleep" is not a model capability problem.

It is an organizational context problem.

For an AI agent to operate autonomously across an engineering organization — not just within a single file — it needs a live model of that organization. Not documentation. Not wiki pages. A **versioned, queryable representation** of:

- Every service, API, data store, and infrastructure component, and how they relate
- Who owns what, what tier it is, what its current lifecycle state is
- The architectural rules, security constraints, and deployment guardrails that govern change
- The actual processes your team uses — the ones that live in senior engineers' heads
- The current state of production: health, cost, reliability, recent changes

None of this exists in a form that AI agents can reason over today. It lives in fragmented tools, tribal knowledge, and documentation that may or may not reflect reality.

**Epoch makes your organization legible to AI.**

---

## What Epoch Is

Epoch is the **organizational runtime** for AI-assisted development.

An operating system doesn't execute your programs — it provides the environment that programs run within. Epoch is that, for AI agents operating inside an engineering organization. The agents are Claude, Cursor, Copilot, or whatever comes next. Epoch doesn't replace them or try to orchestrate them. It provides the environment they operate within: the organizational context that turns a capable-but-narrow agent into one that can safely operate across your entire engineering surface.

Three things compose that environment:

---

### The OmniRepo Graph

The graph is the foundation. It is a live, versioned, queryable model of your engineering organization — derived continuously from source, not maintained by hand.

Node types: Service, Package, API, Infrastructure, DataStore, Team, Policy
Edge types: dependsOn, ownedBy, exposesAPI, consumesAPI, deployedTo, governedBy

The graph is built by scanning what you already have: repositories, Kubernetes manifests, Terraform definitions, OpenAPI specs, protobuf schemas, package manifests, and ownership files. It does not require a migration or a new way of working — it reads what exists.

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

An AI agent with access to the graph can answer cross-repository questions before making a change. Not by guessing — by querying a model of what actually exists.

> The monorepo movement proved that co-locating code yields massive benefits: atomic commits, unified CI, simplified dependency management. But monorepos only model *code*. The OmniRepo extends this to model the entire product surface — infrastructure, policies, ownership, contracts — as a single versioned graph. The difference is not organizational convenience. It is the difference between AI that can see your files and AI that can understand your system.

---

### Skills: Executable Institutional Knowledge

Engineering organizations know how to do things. Deploy a service. Rotate credentials. Deprecate an API version. Scaffold a new service. Patch a vulnerability across ten repositories at once. Respond to a production incident at 2am.

This knowledge exists — but it lives in senior engineers' heads, in wiki pages that lag reality, in Slack threads from six months ago, in playbooks that nobody updates. It's not executable. It's not versioned. And it doesn't transfer.

In Epoch, it becomes **Skills** — named, versioned capabilities your organization has formally defined and can execute. A Skill knows the graph it operates on. It respects the policies that govern it. It can be run by a human, triggered by a CI event, or handed to an AI agent. The same Skill that a senior engineer runs manually today is the one an agent runs autonomously tomorrow.

Some Skills look like playbooks — a sequence of steps for a complex, multi-party operation like a zero-downtime database migration or a cross-team API deprecation. Some look like protocols — a defined response to a class of incident, with clear decision points and escalation paths. Some are utilities — discrete, composable actions that larger Skills assemble into workflows.

What they share: they encode *how your organization does things*, not just what tools exist. An agent executing a Skill isn't guessing — it's following the process your team defined, tested, and trusts.

Skills are the primary trust-building mechanism. Every operation encoded as a Skill is a surface area where AI can act consistently and auditably, without a human watching every step.

---

### Policies: Encoded Guardrails

Autonomy without constraints is risk. Epoch's policy engine makes organizational guardrails executable — evaluated against proposed changes before they land, not discovered in post-mortems.

Policies express what your organization has decided, at a system level. Not style preferences or linting rules — structural constraints that span services, teams, and time. A schema change to a shared database requires a migration in the same PR. A public API endpoint requires authentication. A change to a Tier 1 service during a freeze requires explicit override. A new dependency with a known critical CVE is blocked until patched or reviewed.

These aren't new ideas — most engineering teams already hold these as conventions. The difference is that conventions are enforced by code review attention, tribal knowledge, and luck. Policies are enforced by the system, consistently, regardless of who opened the PR or how busy the reviewers are.

Policy categories include: architectural boundaries, security requirements, reliability thresholds, cost limits, compliance constraints, dependency health, and change management.

Policies run in CI, as pre-commit hooks, and as gates on Skill execution. They start in `warn` mode while a team builds confidence, then escalate to `block` once trusted.

**Policies are what make it safe to extend AI autonomy.** The clearer the guardrails, the more confidently you can let agents roam within them.

---

### Feedback: Production Reality in the Graph

A graph that only models static structure drifts from reality. Epoch closes this loop by ingesting production signals — observability, cost, deployment history — as first-class graph annotations.

SLO status, latency, error rates, cost trends, and recent deployment events attach to the graph nodes they belong to. This means:

- An agent can check service health before operating on it
- A policy can block deploys while error rate exceeds threshold
- A Skill can verify health at every step, not just at the end
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

Instead, Epoch operates as **glue between existing systems** — contributing organizational context at the specific junctions where that context matters most.

**Code review augmentation** — An AI reviewer flags a pattern as non-standard. Epoch consults the graph, recognizes the pattern is an intentional architectural decision for that domain, and resolves the comment automatically — with a citation to the relevant policy. The reviewer didn't know; Epoch did.

**Observability gap detection** — After a change ships, Epoch checks whether the affected code paths have adequate monitoring coverage. If a new path isn't covered by existing alerts, Epoch surfaces the gap before it becomes an incident.

**Nightly validation scans** — Epoch launches lightweight workers on a schedule to verify graph accuracy: are declared owners still active? Are deprecated services still receiving traffic? Have API contracts drifted from their specs? Discrepancies become issues or PRs, not tribal knowledge.

**Cross-repo impact on every PR** — A developer opens a PR. Epoch computes graph-level impact across all repositories, annotates the PR with affected owners, policy results, and downstream risk — before any human reviews a line.

In each case, Epoch contributes what no single AI tool has: organizational context that exists across the whole engineering surface, not just the open file.

---

## The Progression

Epoch is not a binary adoption. Each layer increases the surface area where AI can operate independently.

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
