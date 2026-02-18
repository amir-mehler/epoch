<div align="center">

# Epoch Roadmap

### Two perspectives: Building Epoch, and Adopting Epoch

</div>

---

This document covers two distinct roadmaps:

1. **[Building Epoch](#part-1-building-epoch)** — the development roadmap for Epoch itself as an open source project
2. **[Adopting Epoch](#part-2-adopting-epoch)** — the incremental adoption path for a real company running microservices on Kubernetes

They are deeply linked: every phase of building Epoch should produce something a real team can adopt *that week*. If it doesn't deliver value until everything is built, it will never get adopted.

---

# Part 1: Building Epoch

## Design Principles for the Build

Before phasing, a few constraints that should shape every decision:

- **Value at every layer.** Each phase must be independently useful. No "wait until Phase 4 for this to matter."
- **Graph first, UI last.** The graph model is the core intellectual property. Everything else is a projection of the graph.
- **Adopt, don't rebuild.** Leverage existing open source aggressively — Tree-sitter for parsing, OPA/Cedar for policy, OpenTelemetry for signals, existing LSPs for language intelligence. Epoch's value is the *integration*, not reimplementing solved problems.
- **CLI-native, API-first.** The primary interface is a CLI and a well-documented API. GUIs come later and are projections of the same API.
- **Dogfood immediately.** Epoch should be used to build Epoch from Phase 1 onward.

---

## Phase 0: The Graph Kernel

**Duration: ~2-3 months**
**Goal: Define the core data model and prove that a product graph is queryable and useful.**

### What gets built

- **Graph schema definition** — the node types (Service, Package, API, Infrastructure, DataStore, Team, Policy) and edge types (dependsOn, ownedBy, exposesAPI, consumesAPI, deployedTo, governedBy)
- **Graph builder** — static analysis pipeline that scans a codebase and produces a graph
  - Language-level: AST parsing via Tree-sitter for dependency extraction
  - Infra-level: Kubernetes manifests, Terraform/Pulumi files, Docker Compose
  - Contract-level: OpenAPI specs, protobuf definitions, GraphQL schemas
  - Package-level: `package.json`, `go.mod`, `requirements.txt`, `Cargo.toml`, etc.
- **Graph storage** — in-memory for small orgs, pluggable backend (SQLite for local, something like Neo4j or Memgraph for larger graphs)
- **CLI v0** — `epoch init`, `epoch scan`, `epoch query`
- **`epoch.yaml`** — the root manifest that defines what the OmniRepo contains

### What a user gets

```bash
$ epoch init
$ epoch scan
Discovered 14 services, 6 APIs, 3 databases, 2 K8s clusters, 47 packages

$ epoch query "what depends on user-service?"
  → auth-service (API consumer)
  → billing-service (API consumer)
  → user-db (datastore)
  → api-gateway (routes /users/*)

$ epoch query "what breaks if I remove the GET /users/:id endpoint?"
  → auth-service (calls GET /users/:id in token validation)
  → admin-dashboard (calls GET /users/:id on user detail page)
```

### Why this phase matters

The graph is Epoch's foundation. If you can scan a real codebase and answer questions like "what depends on this?" and "what breaks if I change that?" — you already have something more useful than most architecture documentation. This is the wedge that earns adoption.

### Key technical decisions

| Decision | Recommendation | Rationale |
|----------|---------------|-----------|
| Graph storage | Start with SQLite + adjacency model | Simple, portable, no infra dependency. Migrate to a graph DB later if needed |
| Parsing | Tree-sitter for AST, custom for manifests | Tree-sitter is battle-tested, multi-language, and incremental |
| Schema format | Typed schema in code (Rust or TypeScript) | The graph schema *is* the product — it needs to be precise and versioned |
| CLI framework | Rust (clap) or Go (cobra) | Fast startup, single binary, cross-platform |

---

## Phase 1: Ownership & Boundaries

**Duration: ~2 months**
**Goal: Make the graph organizational, not just technical.**

### What gets built

- **`OWNERS.epoch`** — a file format for declaring service/package/API ownership
- **Domain boundary definitions** — explicit boundaries between teams and systems
- **Tier classification** — Tier 1 (critical), Tier 2 (important), Tier 3 (internal)
- **Lifecycle states** — active, deprecated, migrating, sunsetting, archived
- **Graph enrichment** — ownership and lifecycle data become first-class nodes and edges
- **`epoch owners`** — query ownership across the graph
- **`epoch status`** — dashboard of the system's organizational health

### What a user gets

```bash
$ epoch owners user-service
  Team: platform-identity
  Oncall: @alice, @bob
  Tier: 1
  Status: active

$ epoch status --deprecated
  legacy-auth-service    deprecated since 2025-09-01    2 active consumers remaining
  v1-billing-api         deprecated since 2025-11-15    migration 60% complete

$ epoch affected-teams --change ./api/users/schema.yaml
  → platform-identity (owner)
  → platform-billing (consumer of /users/:id)
  → frontend-web (consumer of /users/me)
```

### Why this phase matters

This is where Epoch stops being a technical tool and starts being an organizational one. Every R&D leader has experienced the pain of "who owns this?" and "is this still active?" This phase makes Epoch valuable to engineering managers and staff engineers, not just individual contributors.

---

## Phase 2: Policy Engine

**Duration: ~2-3 months**
**Goal: Encode organizational guardrails as executable policy.**

### What gets built

- **Policy definition language** — likely a thin DSL on top of OPA (Rego) or Cedar, scoped to the Epoch graph
- **Built-in policy categories:**
  - **Architectural** — "no direct database access across domain boundaries"
  - **Security** — "all public APIs must require authentication"
  - **Cost** — "no infrastructure change that increases monthly cost by >$500 without VP approval"
  - **Reliability** — "Tier 1 services must have ≥99.9% SLO defined"
  - **Compliance** — "PII data stores must have encryption at rest"
  - **Dependency** — "no new dependencies with known critical CVEs"
- **Policy evaluation engine** — evaluates policies against graph mutations (proposed changes)
- **CI integration** — `epoch check` as a CI step that gates PRs on policy compliance
- **`epoch policy lint`** — local pre-commit policy evaluation

### What a user gets

```bash
$ epoch check
  ✓ Architectural: no cross-boundary DB access
  ✓ Security: all public APIs authenticated
  ✗ Cost: infrastructure change increases spend by ~$720/mo (threshold: $500)
      → Requires approval from @vp-engineering
  ✓ Reliability: SLO defined for affected Tier 1 services
  ✗ Dependency: new dependency 'leftpad@3.1.0' has known CVE-2026-1234
      → Use leftpad@3.1.1 or request exception

2 policy violations found. Run `epoch check --explain` for details.
```

### Why this phase matters

Policies are the "immune system" of a healthy engineering org. Today they're enforced by convention, code review fatigue, and tribal knowledge. Epoch makes them explicit, automated, and versioned. This is also the phase that makes the system trustworthy enough for AI integration — you can't let AI agents operate freely without guardrails.

---

## Phase 3: Skills Engine

**Duration: ~3-4 months**
**Goal: Make institutional knowledge executable.**

### What gets built

- **Skill definition format** — YAML + embedded scripts, or a lightweight DSL

```yaml
skill: rotate-database-credentials
version: 2.1.0
description: Rotate credentials for a managed database and propagate to consumers
inputs:
  - name: database
    type: graph.DataStore
    required: true
steps:
  - name: identify-consumers
    query: "graph.consumers(inputs.database, edge='readsFrom')"
  - name: generate-new-credentials
    action: secrets.rotate
    target: "{{ inputs.database.credential_ref }}"
  - name: update-consumers
    foreach: "{{ steps.identify-consumers.result }}"
    action: secrets.propagate
    target: "{{ item.secret_ref }}"
    policy: require-zero-downtime
  - name: verify-health
    foreach: "{{ steps.identify-consumers.result }}"
    action: health.check
    timeout: 5m
  - name: revoke-old-credentials
    action: secrets.revoke
    target: "{{ steps.generate-new-credentials.previous }}"
    depends_on: verify-health
policies:
  - require-change-window
  - notify-oncall
```

- **Skill runtime** — executes skills against the live graph with rollback support
- **Skill composition** — skills can invoke other skills as steps
- **Skill catalog** — `epoch skills list`, `epoch skills run`, `epoch skills test`
- **Dry-run mode** — `epoch skills run --dry-run` shows what *would* happen without executing
- **Built-in starter skills** — 10-15 common operations (deploy, rollback, scale, rotate-secrets, deprecate-api, etc.)

### What a user gets

```bash
$ epoch skills run rotate-database-credentials --database user-db --dry-run

Dry run: rotate-database-credentials v2.1.0
  Target: user-db (PostgreSQL, us-east-1)
  Consumers found: auth-service, billing-service, analytics-worker
  Steps:
    1. Generate new credentials in AWS Secrets Manager
    2. Update auth-service secret ref (zero-downtime: rolling restart)
    3. Update billing-service secret ref (zero-downtime: rolling restart)
    4. Update analytics-worker secret ref (zero-downtime: restart on next cycle)
    5. Health check all consumers (timeout: 5m)
    6. Revoke old credentials
  Policies enforced:
    ✓ Change window: current time is within allowed window
    ✓ Notify oncall: @alice will be notified
  Estimated duration: 8-12 minutes

Proceed? [y/N]
```

### Why this phase matters

This is Epoch's most differentiated feature. Nobody else is doing "executable institutional knowledge as versioned, graph-aware, policy-bound artifacts." This is also the foundation for AI integration — AI agents don't freestyle, they execute skills.

---

## Phase 4: AI Agent Runtime

**Duration: ~3-4 months**
**Goal: Give AI agents full system context and scoped execution.**

### What gets built

- **Context injection API** — feeds relevant graph context to LLMs based on the task scope
- **Agent scoping** — defines what an AI agent can see and do (read-only graph access, specific skill execution, bounded file mutations)
- **Agent-skill binding** — AI agents execute skills, not arbitrary code
- **Impact preview** — AI proposes a change, Epoch shows the multi-dimensional impact before approval
- **Conversation memory tied to graph** — AI remembers past interactions scoped to graph nodes ("last time we discussed migrating user-service, we decided to...")
- **MCP / tool-use integration** — Epoch exposes its graph, skills, and policies as tools that AI agents can call via standard protocols

### What a user gets

```bash
$ epoch agent "migrate user-service from Express to Fastify"

Agent: I'll use the framework-migration skill. Let me analyze the impact first.

Impact Analysis:
  Service: user-service (Tier 1, owned by platform-identity)
  Dependencies: 4 internal consumers, 2 external APIs
  Risk: Medium — Tier 1 service, but migration skill has been tested
  Estimated scope: 23 files, 3 test suites, 1 Dockerfile, 1 K8s deployment

  Policy check:
    ✓ Migration has approved change window
    ✓ Rollback plan defined in skill
    ✗ Requires platform-identity tech lead approval (Tier 1 service)

Agent: Approval needed from @alice. Shall I draft the migration PR
       and request review?
```

### Why this phase matters

This is where Epoch becomes an AI-native system. The key insight: AI is not a feature bolted on at the end — it's a first-class operator that was always intended to operate within the graph/skills/policy framework. Every prior phase was building the structure that makes AI trustworthy at the organizational level.

---

## Phase 5: Feedback Loop

**Duration: ~2-3 months**
**Goal: Close the loop between production reality and the graph.**

### What gets built

- **OpenTelemetry collector integration** — ingest traces, metrics, logs
- **SLO/SLI tracking** — defined in the graph, evaluated against real signals
- **Cost signal integration** — cloud billing APIs (AWS Cost Explorer, GCP Billing, Azure Cost Management) feeding into graph nodes
- **Anomaly correlation** — "this latency spike started 12 minutes after deploy #847 of billing-service"
- **Graph annotations** — production signals enrich the graph ("user-service p99 latency: 240ms, SLO: 300ms, budget remaining: 20%")
- **Feedback-driven policy** — policies that reference live signals ("block deploys to billing-service while error rate > 5%")

### What a user gets

```bash
$ epoch health user-service
  SLO: 99.95% availability (current: 99.97%, budget: 62% remaining)
  P99 latency: 240ms (target: 300ms)
  Error rate: 0.3% (threshold: 1%)
  Monthly cost: $2,847 (budget: $3,000, trend: +8% MoM)
  Last deploy: 3 hours ago (no anomalies detected)
  Risk: LOW

$ epoch health --portfolio
  12/14 services healthy
  billing-service: SLO budget at 11% — approaching violation
  legacy-auth: error rate elevated since deploy #612 — investigating
```

---

## Phase 6: Visualization & Collaboration

**Duration: ongoing**
**Goal: Make the graph visible and navigable for everyone.**

### What gets built

- **Web UI** — interactive graph explorer, health dashboards, skill catalog browser
- **Change review overlay** — "when I view this PR, show me the graph-level impact"
- **IDE extensions** — "while editing this file, show me what I'm affecting in the graph"
- **Notification system** — intelligent alerts based on ownership and impact
- **Collaboration features** — propose skills, review policy changes, discuss graph mutations

### Why this comes last

Not because it's unimportant — but because everything above it needs to be solid first. The UI is a projection of the graph. If the graph is wrong, a beautiful UI just makes wrong information more visible. CLI-first forces correctness.

---

## Build Timeline Summary

```
Month:  1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16+
        ├───┼───┼───┤
        Phase 0: Graph Kernel
                ├───┼───┤
                Phase 1: Ownership
                        ├───┼───┼───┤
                        Phase 2: Policy
                                ├───┼───┼───┼───┤
                                Phase 3: Skills
                                        ├───┼───┼───┼───┤
                                        Phase 4: AI Runtime
                                                    ├───┼───┼───┤
                                                    Phase 5: Feedback
                                                            ├───────→
                                                            Phase 6: UI
```

> **Realistic note:** This is an ambitious 16+ month roadmap for a core team of 3-5 strong engineers. Phases overlap intentionally — skills design should inform graph schema, policy work should start alongside ownership. The critical path is Phase 0 → Phase 3. If the graph is solid and skills work, everything else is acceleration.

---
---

# Part 2: Adopting Epoch

## Scenario

A company running **8-15 microservices** across **2-3 Kubernetes clusters** (e.g., staging + production, or multi-region). They have:

- Multiple Git repos (maybe a monorepo for some services, separate repos for others)
- Terraform or Pulumi for infrastructure
- Helm charts or Kustomize for K8s deployments
- A CI/CD pipeline (GitHub Actions, GitLab CI, or ArgoCD)
- Some OpenAPI specs, maybe some protobuf
- Observability via Datadog/Grafana/Prometheus
- The usual pain: "who owns what?", "will this break something?", "how do I do X?"

## Adoption Philosophy

Three rules:

1. **Never big-bang.** Epoch is adopted incrementally, one layer at a time.
2. **Read-only first.** Start by observing and modeling. Don't change any workflows until the graph is trusted.
3. **Value within the first week.** If the team doesn't see value in week 1, adoption fails.

---

## Week 1-2: Bootstrap the Graph

**Effort: 1 engineer, 2-3 days of focused work**

### What you do

```bash
# Install Epoch CLI
brew install epoch  # or cargo install epoch-cli

# Initialize in your root directory (or monorepo)
epoch init --scan-repos ./services/* ./infra/* ./shared/*

# Point at your K8s clusters
epoch discover k8s --kubeconfig ~/.kube/config --context staging
epoch discover k8s --kubeconfig ~/.kube/config --context production

# Discover API contracts
epoch discover apis --openapi ./services/*/api/openapi.yaml
epoch discover apis --proto ./shared/proto/

# See what Epoch found
epoch graph summary
```

### What you get

```
Epoch Graph Summary:
  Services:        12
  APIs:            8 (6 OpenAPI, 2 protobuf)
  Data stores:     4 (2 PostgreSQL, 1 Redis, 1 S3)
  K8s deployments: 24 (12 staging, 12 production)
  Packages:        89
  Dependencies:    147 edges
  Unresolved:      3 (unknown consumers of legacy-api)
```

### Immediate value

- **"What depends on X?"** — answer it in seconds instead of grep + Slack
- **Dependency visualization** — see your actual architecture, not the one on the whiteboard from 2 years ago
- **Drift detection** — "these 3 services reference an API that no longer exists in the spec"

---

## Week 3-4: Add Ownership & Classification

**Effort: 1 engineer + 30 min from each team lead**

### What you do

Create `OWNERS.epoch` files alongside services:

```yaml
# services/user-service/OWNERS.epoch
service: user-service
team: platform-identity
oncall:
  primary: alice@company.com
  secondary: bob@company.com
tier: 1
status: active
data_classification: pii
```

Classify all services:

```bash
epoch owners import --from ./OWNERS.epoch
epoch tier set billing-service 1
epoch tier set internal-tools 3
epoch status set legacy-auth deprecated --sunset 2026-06-01
```

### Immediate value

- **`epoch owners <anything>`** — instant ownership lookup from the terminal
- **Tier-aware reasoning** — "you're about to change a Tier 1 service, here's what depends on it"
- **Deprecation tracking** — "legacy-auth has 4 remaining consumers, here's who needs to migrate"

---

## Month 2: Connect to CI/CD

**Effort: 1 engineer, 1-2 days**

### What you do

Add `epoch check` to your CI pipeline:

```yaml
# .github/workflows/pr-check.yml
- name: Epoch Impact Analysis
  run: |
    epoch check --change ${{ github.event.pull_request.head.sha }}
    epoch affected --change ${{ github.event.pull_request.head.sha }} --format github-comment
```

### What you get

On every PR, a comment like:

> **Epoch Impact Analysis**
>
> **Changed:** `user-service` (Tier 1, owned by platform-identity)
>
> | Dimension | Status |
> |-----------|--------|
> | API compatibility | ⚠️ Breaking change to `GET /users/:id` response shape |
> | Affected consumers | `auth-service`, `admin-dashboard` |
> | Test coverage | ✅ Integration tests cover affected paths |
> | Infrastructure | No changes |
> | Security | ✅ No new public endpoints |
>
> **Action required:** Notify `auth-service` team (@charlie) about response shape change.

### Immediate value

- **Cross-service impact is visible on every PR** — no more "we didn't know this would break billing"
- **Ownership-aware review routing** — the right people get notified automatically
- **API compatibility checking** — contract-breaking changes are caught before merge

---

## Month 3: Define Your First Policies

**Effort: Tech lead + 1 engineer, 2-3 days**

### What you do

Start with 3-5 policies that encode your existing conventions:

```bash
epoch policy create architectural/no-cross-boundary-db-access
epoch policy create security/public-apis-require-auth
epoch policy create reliability/tier1-services-need-slo
epoch policy create dependencies/no-critical-cves
```

Example policy:

```rego
# policies/architectural/no-cross-boundary-db-access.rego
package epoch.architectural

violation[msg] {
    service := input.change.service
    db := input.graph.edge[service]["directDbAccess"]
    db.owner != service.owner
    msg := sprintf(
        "%s directly accesses %s's database — use their API instead",
        [service.name, db.owner.name]
    )
}
```

Wire them into CI:

```bash
epoch check --policies architectural,security,reliability
```

### Immediate value

- **Architectural conventions become automated** — no more "we talked about this in the architecture review 6 months ago"
- **Security baselines enforced automatically** — "every public API must be authed" isn't a guideline anymore, it's a gate
- **Gradual adoption** — start with `warn` mode, escalate to `block` once the team trusts the policies

---

## Month 4-5: Encode Your First Skills

**Effort: Platform/infra engineer, 1-2 weeks**

### What you do

Pick 2-3 operations your team does repeatedly and encode them:

1. **Deploy a service** (with pre/post checks)
2. **Rotate database credentials** (with consumer propagation)
3. **Add a new service** (with all the boilerplate, K8s manifests, CI config, ownership)

```bash
epoch skills create deploy-service
epoch skills create rotate-db-credentials
epoch skills create scaffold-new-service
```

### Immediate value

```bash
# Before Epoch: 45 minutes of following a wiki page, hoping it's up to date
# After Epoch:
$ epoch skills run deploy-service --service billing-service --env production --dry-run

Pre-flight checks:
  ✓ All tests passing on main
  ✓ No SLO budget violations on billing-service
  ✓ Change window: OK (Tuesday 2pm, outside freeze)
  ✓ Approval: @alice approved deploy #847
Steps:
  1. Build image: billing-service:v2.14.3
  2. Push to registry
  3. Update K8s deployment (rolling, maxUnavailable=1)
  4. Health check (timeout: 5m)
  5. Notify #deployments channel

Proceed? [y/N]
```

- **Runbooks become executable** — "how do I deploy?" is answered by running a skill, not reading a doc
- **New engineers are productive faster** — skills encode the senior engineers' knowledge
- **Consistency** — every deploy follows the same path, every time

---

## Month 6+: AI Agents & Feedback

**Effort: Ongoing, incremental**

### What you do

- Enable AI agents with graph context for code changes
- Connect observability signals to the graph
- Build more skills as you identify repeated operations
- Expand policies as new conventions emerge

### The flywheel

```
     Adopt Epoch → Model your system → Discover blind spots
         ↑                                      ↓
    Expand coverage ← Encode knowledge ← Fix problems
```

Every problem you discover becomes a skill or policy. Every skill and policy makes the system more capable. This is the adoption flywheel — and it's why the read-only graph in Week 1 matters so much. You have to see your system clearly before you can improve it.

---

## Adoption Timeline Summary

```
Week:   1    2    3    4    5    6    7    8   ...  12  ...  16  ...  24+
        ├────┼────┤
        Graph Bootstrap
        (scan, discover, query)
                  ├────┼────┤
                  Ownership & Boundaries
                  (OWNERS files, tiers, lifecycle)
                            ├────┼────┼────┤
                            CI Integration
                            (impact analysis on PRs)
                                      ├────┼────┼────┤
                                      First Policies
                                      (architectural, security)
                                                ├────┼────┼────┼────┤
                                                First Skills
                                                (deploy, rotate, scaffold)
                                                               ├───────→
                                                               AI + Feedback
                                                               (ongoing)
```

| Milestone | When | Team Effort | Value |
|-----------|------|------------|-------|
| **Graph is live** | Week 2 | 1 engineer, 3 days | Answer "what depends on X?" instantly |
| **Ownership is tracked** | Week 4 | 30 min per team lead | "Who owns this?" has a real answer |
| **Impact on every PR** | Month 2 | 1 engineer, 2 days | No more surprise cross-service breakage |
| **Policies enforced** | Month 3 | 1 engineer, 3 days | Conventions are automated, not aspirational |
| **Skills are executable** | Month 5 | 1 engineer, 2 weeks | Runbooks are code, not wiki pages |
| **Feedback loop closed** | Month 6+ | Ongoing | Production reality informs every decision |

---

## The Honest Caveats

A few things to be real about:

- **Graph accuracy is everything.** If the graph is wrong, nothing built on it is trustworthy. Invest heavily in graph validation and make it easy to report inaccuracies.
- **Skills require senior engineering time.** The people who know how things work are the same people who are too busy to document it. Encoding skills is an investment — frame it as "write it once, never explain it again."
- **Policy adoption needs cultural buy-in.** Engineers will resist automated gates if they feel arbitrary. Start with `warn`, involve the team in policy design, and only escalate to `block` when there's consensus.
- **The cold-start problem is real.** An empty graph has no value. The bootstrapping experience (Phase 0 / Week 1) must be near-magical: point at repos, get a useful graph in minutes, not days.
- **Multi-repo is harder than monorepo.** If the company has 15 separate repos, the graph needs to span them. This is solvable but adds friction. Monorepo teams will have an easier adoption path.
