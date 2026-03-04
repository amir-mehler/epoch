<div align="center">

# Inspiration & Prior Art

*What we've learned from other projects building in this space.*

</div>

---

## Anthropic — Agent Skills (`anthropics/skills` + agentskills.io)

**What it is:** An open standard for packaging AI agent capabilities as portable, self-describing instruction sets. A Skill is a directory with a `SKILL.md` file: YAML frontmatter (`name`, `description`) followed by markdown instructions. Optional `scripts/`, `references/`, and `assets/` directories for supporting resources.

**Key insights:**

**Progressive disclosure is the right mental model.** Metadata (~100 tokens) is always loaded. Full instructions load when the skill activates. Supporting files load only when referenced. This keeps context efficient without hiding capability. Epoch's Skills should follow the same loading hierarchy.

**The format is intentionally minimal.** No workflow engine, no rigid step sequencing. The instructions are written *for an AI to understand*, not for a machine to parse. Intelligence stays with the agent; the Skill provides domain context and patterns. This is the right level of abstraction — Epoch Skills should not become Kubernetes-style YAML manifests.

**It's an open standard, not an Anthropic product.** The spec lives at `agentskills.io` and is designed to work across any compliant agent. This means Epoch Skills built on this format are compatible with the broader ecosystem by default. We should treat agentskills.io as the base layer and Epoch as a profile on top of it — adding graph-awareness and policy-binding without forking the format.

**`allowed-tools` is a primitive policy mechanism.** The experimental field pre-approves specific tools a Skill can use. Epoch's policy layer is a more powerful, graph-aware version of this same idea.

---

## Superpowers (`obra/superpowers`)

**What it is:** A complete AI-assisted software development workflow implemented as a set of composable Agent Skills. The core loop: `brainstorming → writing-plans → subagent-driven-development → finishing-a-development-branch`. Skills trigger automatically based on context — the developer doesn't invoke them manually.

**Key insights:**

**Subagent-driven development is the right execution model.** Fresh subagent per task (no context pollution), two-stage review after each task (spec compliance first, then code quality), review loops that don't move forward until issues are resolved. This pattern produces consistently high-quality output and can run autonomously for hours. Epoch's agent runtime should adopt this pattern for Skill execution.

**Hard gates work.** The `brainstorming` skill uses a `<HARD-GATE>` tag: no implementation may begin until the user has approved a design. This is a forcing function that prevents the most common AI failure mode (confident, wrong code). Epoch's high-risk Skills — anything touching Tier 1 systems, production data, or irreversible operations — should have explicit hard gates before execution.

**DOT graph notation for decision trees is better than prose.** Skills in Superpowers use Graphviz DOT format for decision flows. It's readable by humans, parseable by machines, and forces clarity about branches and terminal states. Worth adopting for Epoch's complex Skills.

**Superpowers is the workflow layer; Epoch is the organizational layer.** Superpowers answers "how do I build one feature well with AI" — and answers it well. What it has no model of: other repos, ownership, API consumers, tier classification, cross-team policies, compliance constraints. A mature Epoch installation would likely use Superpowers-style Skills as the implementation workflow, with Epoch providing the graph context and guardrails those Skills operate within. They compose rather than compete.

**For small single-repo teams, this alone gets you surprisingly far.** Honest assessment: Superpowers + a thoughtful CLAUDE.md might deliver 80% of the L3→L4 value for a small team in one repo. Epoch's value emerges clearly when you have multiple repos, multiple teams, compliance requirements, and need the organizational context layer to scale.

---

## Awesome Agent Skills (`VoltAgent/awesome-agent-skills`)

**What it is:** A curated index of 383+ Agent Skills published by real engineering teams — Anthropic, Vercel, Stripe, Cloudflare, HashiCorp, Sentry, Hugging Face, Google, and many others. Also serves as a compatibility reference for which AI coding tools support Skills and where they look for them.

**Key observation:** The agentskills.io format has been adopted by every major AI coding assistant — Claude Code, Cursor, GitHub Copilot, Gemini CLI, Codex, OpenCode, Windsurf. It is now the de facto standard for packaging knowledge and workflows for AI agents. This is not a niche format; it's the ecosystem.

**Implication for Epoch:** Epoch Skills built on this format are immediately compatible with the tools every engineering team already uses. The organizational knowledge encoded in Epoch doesn't need a proprietary runtime to be useful — it works wherever agents run. This is a strong argument for treating agentskills.io as Epoch's base Skill format rather than inventing something new — and then layering graph-awareness and policy-binding on top.

**Reference:** https://github.com/VoltAgent/awesome-agent-skills — worth consulting when designing Epoch's starter Skill library to avoid rebuilding what already exists.

---

## Awesome Claude Code Subagents (`VoltAgent/awesome-claude-code-subagents`)

**What it is:** A collection of 127+ Claude Code subagents — specialized AI assistants defined as markdown files in `.claude/agents/`. Organized into categories: core development, language specialists, infrastructure, quality/security, data/AI, developer experience, specialized domains, business/product, meta-orchestration, research/analysis.

**Skills vs. Agents — a useful distinction this collection makes clear:**
- **Skills** teach Claude *how* to do something — methodology, decision trees, patterns, step-by-step processes
- **Agents** define *who* Claude is when doing something — persona, expertise scope, allowed tools, model selection

They compose: a Skill can invoke an Agent; an Agent can be instructed to follow a Skill. Epoch will likely need both layers — Skills encoding organizational procedures, Agents embodying specialized roles (graph analyst, policy reviewer, impact assessor).

**The meta-orchestration category is the most relevant for Epoch.** It includes agents for multi-agent coordination, workflow orchestration, task distribution, context management, and error coordination. The *intent* is right — this is exactly the layer Epoch's agent runtime needs. The *implementations* are thin: mostly aspirational checklists with made-up throughput metrics. Nobody has solved this well yet. The gap between "here's an orchestration agent definition" and "here's a working multi-agent system that actually coordinates reliably" is still wide open. Epoch's graph and Skills layer is what would actually fill that gap.

**Honest quality assessment:** The collection is uneven. The infrastructure and language specialist agents are genuinely useful — focused, specific, no fluff. Many meta-orchestration agents read as AI-generated with impressive-sounding but empty metrics ("87 agents, 234K messages/min"). Compare to Superpowers' `code-reviewer.md` — tight, actionable, no theater. Quality of agent definitions matters enormously; a bad agent definition gives the AI no useful guidance.

**Implication for Epoch:** When Epoch publishes agents (e.g., `epoch-impact-analyzer`, `epoch-policy-checker`, `epoch-graph-query`), they should follow the Superpowers quality bar — specific, decision-tree-driven, with real hard gates — not the aspirational-checklist pattern common here.

**Reference:** https://github.com/VoltAgent/awesome-claude-code-subagents

---

## Planning with Files (`OthmanAdi/planning-with-files`) — and the Manus acquisition

**What it is:** A Skill that implements "context engineering" — using persistent markdown files as an AI agent's working memory on disk. Three files per task: `task_plan.md` (phases, progress, decisions), `findings.md` (research, discoveries), `progress.md` (session log, errors). Hooks re-read the plan before every significant action. The author credits Manus AI's public writing on how they built their agent architecture.

**Context: Meta acquired Manus for $2 billion in late 2025.** Manus went from launch to $100M+ revenue in 8 months. Their core innovation was not a better model — it was context engineering: treating the filesystem as persistent agent memory, and treating re-reading state as a first-class operation in every workflow.

> *"Markdown is my 'working memory' on disk. Since I process information iteratively and my active context has limits, Markdown files serve as scratch pads for notes, checkpoints for progress, building blocks for final deliverables."* — Manus AI

**The fundamental mental model:**
```
Context Window = RAM  (volatile, limited)
Filesystem     = Disk (persistent, unlimited)
→ Anything important gets written to disk.
```

**Key insights:**

**Attention manipulation is a real technique.** The PreToolUse hook re-reads `task_plan.md` (first 30 lines) before *every* tool call. Not because the AI forgot — but because keeping goals explicitly in the active context window prevents goal drift over long task sequences. This is the same reason Epoch's graph should be queried at the start of every significant agent operation, not just once at the beginning.

**Hooks are the right mechanism for behavioral guardrails.** PreToolUse, PostToolUse, and Stop hooks let you inject behavior around agent actions without modifying the agent itself. This is Epoch's "glue at specific junctions" pattern in practice. Before a tool runs: check context. After a file write: remind to update state. Before stopping: verify completion. Epoch should adopt hooks as a first-class mechanism for injecting graph queries and policy checks around agent operations.

**The 3-file pattern is a minimal graph.** `task_plan.md` is state. `findings.md` is the knowledge base. `progress.md` is the audit trail. Epoch's OmniRepo graph is this same pattern at organizational scale — structured, queryable, versioned, and spanning the entire engineering surface rather than a single session.

**Session recovery matters.** When context fills up and resets, the skill recovers previous session context from Claude's project files and git diffs. Epoch's feedback loop — production signals continuously refreshing the graph — is the organizational-scale version of this same idea: agents always have current state, not stale memory.

**The $2B lesson for Epoch:** The most valuable thing Manus built was not AI capability — it was the infrastructure that made AI capability *reliable and persistent across time*. Epoch is building that same infrastructure layer, but at organizational scope: not one agent's working memory, but the entire engineering organization's memory — versioned, queryable, and always current.

**Reference:** https://github.com/OthmanAdi/planning-with-files

---

## Skill Prompt Generator (`huangserva/skill-prompt-generator`) — Chinese project

An AI image prompt generation system built on the Agent Skills format. Reviewed and noted — the domain (Midjourney/DALL-E prompt composition) is not relevant to Epoch.

**Reference:** https://github.com/huangserva/skill-prompt-generator

---

## Claude Mem (`thedotmack/claude-mem`) — 20K stars in 2 days

**What it is:** Persistent memory for Claude Code. Five lifecycle hooks automatically capture everything Claude does during a session — observations, decisions, discoveries — compress them into semantic summaries, and inject relevant context back into future sessions via hybrid SQLite + Chroma vector search.

**Why 20K stars in 2 days:** It's solving the single most frustrating thing about working with AI agents — they forget everything when the session ends. Every engineer has felt this. The market is screaming that persistent, automatic context is a top-tier unsolved problem.

**Key insights:**

**Automatic capture beats manual effort.** Planning-with-files requires the agent to consciously write findings to disk. Claude Mem captures automatically via hooks — no discipline required. For organizational memory (Epoch's graph), the implication is the same: the graph should be updated as a side effect of work, not as a deliberate extra step. Graph mutations should happen automatically when agents execute Skills, not only when humans manually update documentation.

**Vector search over keyword search.** The system uses Chroma for semantic similarity search over session history, not just keyword matching. An agent asking "what decisions did we make about the auth architecture?" gets relevant results even if those exact words weren't used. Epoch's graph query layer should think in terms of semantic relevance, not just exact-match graph traversal.

**Progressive disclosure of memory cost.** The system surfaces token costs per memory retrieval — you see how much context a memory injection costs before it's loaded. This is the right UX for a memory system: make the tradeoff visible. Epoch's context injection for agents should follow the same principle.

**Architecture parallel to Epoch:** Claude Mem is individual, session-scoped, automatic memory. Epoch's graph is organizational, permanent, structured memory. They solve the same problem at different scales. The architectural patterns (hooks for capture, vector search for retrieval, progressive disclosure) should inform Epoch's design.

**Reference:** https://github.com/thedotmack/claude-mem — worth cloning for a deeper look at the hook architecture and vector retrieval implementation.

---

## GSD — Get Shit Done (`glittercowboy/get-shit-done`)

**What it is:** A spec-driven development framework that interviews you, builds a phased plan, then executes phase by phase with verification. One command install (`npx get-shit-done-cc`). Supports Claude Code, OpenCode, Gemini CLI, Codex.

**How it advances beyond Superpowers:**

**Parallel wave execution.** Independent tasks in a phase run simultaneously in fresh 200k-token contexts. No context pollution, no sequential bottleneck. This is the key execution improvement over Superpowers' sequential subagent model — and it maps directly to Epoch's vision of running multiple agents across different parts of the codebase simultaneously.

**Model profiles per phase.** Opus for planning and research, Sonnet for execution, Haiku for verification. Right-sizing model capability to task complexity rather than using one model for everything. Epoch's agent runtime should think the same way: expensive reasoning for impact analysis and policy evaluation, cheaper models for mechanical execution tasks.

**XML-structured task plans for Claude.** Plans use XML (`<task>`, `<action>`, `<verify>`, `<done>`) optimized for Claude's parsing rather than prose or YAML. The structure is both human-readable and machine-parseable. Epoch's Skill execution plans should adopt a similarly structured format.

**Explicit "context rot" framing.** GSD names the problem clearly: as context fills, quality degrades. Their solution — modular reference files sized for consistent quality, fresh contexts per execution wave — is the right structural response. Epoch's progressive disclosure principle (metadata always loaded, full Skill loaded when activated, resources loaded on demand) is the same pattern applied to organizational context.

**Reference:** https://github.com/glittercowboy/get-shit-done — recommend cloning to study the wave execution implementation and XML task plan format.

---

## n8n-MCP (`czlonkowski/n8n-mcp`)

**What it is:** An MCP server that gives AI assistants structured access to all 1,084 n8n workflow automation nodes — full documentation, property schemas, configuration examples, and 2,709 workflow templates — via a precomputed SQLite database.

**The pattern, not the tool:** n8n itself is not relevant to Epoch. But the architecture demonstrates something directly applicable: take a large, complex domain (1,084 nodes with 87% documentation coverage), precompute it into a queryable SQLite database, expose it via MCP with tiered detail levels (minimal/standard/full for token efficiency). AI agents can then work competently with the entire domain without hallucinating configurations.

This is exactly how Epoch's organizational graph should be exposed: precomputed from static analysis, queryable at different depth levels, accessible via MCP so any agent can use it regardless of which AI tool the team uses.

**Reference:** https://github.com/czlonkowski/n8n-mcp

---

## Also Noted

**Obsidian Skills** (`kepano/obsidian-skills`, 7K stars) — Built by Obsidian's CEO. Lets Claude Code manage a personal Obsidian vault as an AI-powered knowledge base. Interesting as a "personal knowledge graph" concept — the parallel to Epoch's organizational graph is direct, just at individual scope. The Obsidian vault model (markdown files as linked knowledge nodes) is a simpler, human-maintained version of what Epoch's graph does with code-derived structure.

**UI/UX Pro Max Skill** (`nextlevelbuilder/ui-ux-pro-max-skill`, 16K stars) — 50+ UI styles, 97 color palettes, 57 font pairings, auto-generates a design system. Another instance of the "rich domain database + Skill interface" pattern (see huangserva). The star count confirms massive demand for Skills with deep domain knowledge preloaded. Epoch's starter Skill library should think similarly: rich organizational context preloaded, not generic instructions.

**Awesome Claude Code** (`hesreallyhim/awesome-claude-code`, 20K stars) — Master directory of Claude Code skills, plugins, hooks, and tools. Equivalent to awesome-agent-skills but Claude Code-specific. Reference index: https://github.com/hesreallyhim/awesome-claude-code

---
