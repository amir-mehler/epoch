# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

This is currently a **docs-only project** — a white paper and roadmap for a concept called Epoch. There is no code yet. When contributing, the primary artifacts are Markdown documents.

## What Epoch Is

Epoch is a proposed "Development Operating System" — a system that unifies code, infrastructure, API contracts, ownership, policies, observability, and institutional knowledge into a single versioned graph called the **OmniRepo**.

Core concepts:
- **OmniRepo** — the complete versioned product graph (not just a monorepo, but everything: code, infra, contracts, ownership, policies, knowledge)
- **Graph** — the foundational data model; node types are Service, Package, API, Infrastructure, DataStore, Team, Policy; edges are dependsOn, ownedBy, exposesAPI, consumesAPI, deployedTo, governedBy
- **Skills** — versioned, graph-aware, policy-bound executable knowledge artifacts (e.g. `rotate-database-credentials`, `deploy-service`)
- **Policies** — encoded organizational guardrails (likely OPA/Rego or Cedar) evaluated against graph mutations
- **AI Runtime** — AI agents that operate *within* the graph/skills/policy framework rather than as free agents

## Key Documents

- `README.md` — the white paper; covers the problem, core concepts (OmniRepo, Skills, Policy, AI Runtime, Observability), and the design philosophy
- `ROADMAP.md` — two tracks: (1) building Epoch in phases (Phase 0–6), (2) incremental adoption path for a real company

## Architecture Direction (from Roadmap)

When code work begins, the planned stack is:

| Concern | Direction |
|---------|-----------|
| CLI | Rust (`clap`) or Go (`cobra`) — single binary |
| Graph storage | SQLite initially, pluggable (Neo4j/Memgraph later) |
| Parsing | Tree-sitter for AST; custom for K8s/Terraform manifests |
| Policy engine | OPA (Rego) or Cedar |
| Schema format | Typed in code (Rust or TypeScript) |
| Root manifest | `epoch.yaml` |
| Ownership file | `OWNERS.epoch` per service |

Build phases: Graph Kernel → Ownership & Boundaries → Policy Engine → Skills Engine → AI Agent Runtime → Feedback Loop → Visualization

## Design Principles

From the roadmap, decisions should follow:
- **Graph first** — the graph model is the core; everything else is a projection of it
- **Value at every layer** — each phase must be independently useful before the next begins
- **Adopt, don't rebuild** — leverage existing open source (Tree-sitter, OPA, OpenTelemetry) for solved problems; Epoch's value is the integration
- **CLI-native, API-first** — primary interface is CLI + documented API; GUIs come later
- **Dogfood immediately** — Epoch should be used to build Epoch from Phase 0 onward
