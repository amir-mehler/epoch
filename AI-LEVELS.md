<div align="center">

# Epoch and the Five Levels of AI Development

</div>

---

Dan Shapiro's article [*The Five Levels: From Spicy Autocomplete to the Dark Factory*](https://www.danshapiro.com/blog/2026/01/the-five-levels-from-spicy-autocomplete-to-the-software-factory/) gives the clearest framework we've seen for describing where teams are in their AI development journey — and where they're trying to go.

The levels, briefly:

| Level | Who's driving | What it looks like |
|-------|--------------|-------------------|
| **0 — Manual** | Human | AI as search engine. Nothing ships without human keystrokes. |
| **1 — Assisted** | Human | Delegate discrete tasks. "Write a test for this." Workflow unchanged. |
| **2 — Autopilot** | Human + AI | AI as a junior pair. Flow state, high productivity. Feels like the finish line — it isn't. |
| **3 — Safety Driver** | AI, human reviews | AI is the senior developer. Life is diffs. You manage what the AI produces. |
| **4 — Robotaxi** | AI, human specifies | You write specs and requirements. You leave. You come back to test results. |
| **5 — Dark Factory** | AI, autonomous | High-level intent in, working software out. Humans neither needed nor expected in the loop. |

Shapiro puts himself at Level 4. He estimates only a handful of small teams operate at Level 5.

---

## Where Most Teams Are Stuck

Most teams using AI tools today are at **Level 2 or 3** — and plateau there.

Level 3 feels productive: AI writes the code, engineers review diffs, throughput is high. But the ceiling is real. Existing AI tools operate on **a single repository at a time**. They see files. They don't see the organization.

This means the AI cannot reason about:

- Which services depend on the code it's changing
- Whether a change violates an architectural boundary
- Who owns the affected system and what tier it is
- Whether a proposed infrastructure change will break cost or reliability budgets
- What the organization's deployment process actually looks like
- Which APIs are stable contracts vs. internal implementation details

At Level 3, humans compensate for this gap — manually. They do the cross-repo reasoning, the organizational context lookup, the impact assessment. The AI is powerful but narrow. The engineer is still doing coordination work that doesn't scale.

**This is not an AI capability problem. It is an organizational context problem.**

---

## The Gap Between Level 3 and Level 4

The jump from L3 to L4 is not about better models. It is about whether the AI has enough context to make autonomous decisions you can trust.

At L4, an engineer can hand a spec to an AI and leave. That only works if:

1. **The AI knows your system** — not just the code, but the services, their relationships, their owners, their tiers, their current health
2. **The AI knows your rules** — architectural policies, security constraints, deployment guardrails, approval requirements
3. **The AI knows how your org operates** — the actual processes, not generic best practices
4. **You can verify what it did** — the AI's actions are auditable, reversible, and explainable

None of this comes from a better model. It comes from infrastructure you build and maintain.

---

## What Epoch Provides

Epoch is the infrastructure layer that makes the L3→L4→L5 climb possible.

It does this by making your organization's reality **programmable, traversable, and trustworthy**:

- **The graph** gives AI a complete model of your system — services, contracts, infrastructure, ownership, lifecycle states — across all repositories, not just one
- **Skills** encode how your organization actually operates — the real deployment process, the real migration path, the real on-call rotation — as versioned, executable artifacts AI agents can follow
- **Policies** define the guardrails — what AI is and isn't allowed to do, at what tier, in what context — so autonomy can be extended gradually without trust being broken
- **The feedback loop** closes the gap between the graph and production reality — signals from observability, cost, and deployment feed back into the model AI operates on

Each layer increases the surface area where AI can operate independently. Each layer narrows the set of decisions that still require human judgment.

**You don't flip a switch to reach Level 5. You build toward it, one encoded constraint at a time.**

---

## The North Star: The Dark Factory

Everything in Epoch is designed with Level 5 in mind.

The dark factory — working software produced from high-level intent, with no human in the loop — is the destination. Not as a uniform state for all systems at once, but as the **ceiling for any given system, reached incrementally**.

A mature Epoch-managed codebase might look like:
- New feature spec submitted → AI designs, implements, tests, reviews, and ships to staging
- Production incident detected → AI diagnoses, patches, deploys fix, and notifies on-call with a summary
- Dependency CVE discovered → AI assesses blast radius, opens PRs across affected services, and routes approvals
- Quarterly infrastructure cost review → AI identifies waste, proposes changes, and executes after approval

The humans in this picture are not absent — they set intent, review outcomes, evolve policies, and build new skills. But the execution layer runs on its own.

---

## Gradual by Design

Not every team, service, or product is at the same point. Epoch does not assume a uniform level across an organization.

A Tier 1 production service might require human approval at every deployment step (L3). An internal tooling service with comprehensive test coverage might operate fully autonomously (L5). A new product in early development might sit somewhere in between.

Epoch's model supports this heterogeneity:

- Tier classification and lifecycle states inform how much autonomy is appropriate per system
- Policies can require human approval for high-stakes operations while allowing full autonomy for low-risk ones
- Skills encode the right level of guardedness for each operation type

The goal is not to push every system to L5 as fast as possible. It is to make each system's **actual level explicit and improvable** — and to give teams the tools to move up deliberately, when they're ready.

---

## In One Line

Epoch builds the organizational infrastructure — graph, skills, policies, feedback — that lets AI agents earn and exercise increasing levels of autonomy across your entire engineering surface, with Level 5 as the horizon.
