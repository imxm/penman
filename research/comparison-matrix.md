# 25-Dimension Comparison: Hound, Grimoire, LISA, Codex Security, Claude Code Security

## Dimensions

| # | Dimension | Hound | Grimoire | LISA | Codex Security | Claude Code Security |
|---|-----------|-------|----------|------|---------------|---------------------|
| 1 | **Core approach** | Relation-first knowledge graphs + belief system | Multi-agent toolkit with manual distillation | KB from past audits + multi-agent dispatch | Threat model → agentic search → sandbox | Context research → hypothesis → self-verify |
| 2 | **Codebase understanding** | Agent-built aspect graphs (auth, value flows, invariants) | GRIMOIRE.md context map + cartography flows | Pre-analysis: syntactic features, call graphs | Repo → project-specific threat model | Researches existing security patterns in repo |
| 3 | **Knowledge source** | Current codebase only (per-project graphs) | Personal grimoire (per-researcher) + external (Solodit, Context7) | Historical Code4rena audit reports (public) | Repository analysis only | Model training data (public) |
| 4 | **Knowledge persistence** | Hypotheses + coverage + plan ledger persist across sessions | Personal grimoire persists across audits (~/.grimoire/) | KB continuously updated with new findings | None (per-commit) | None (per-review) |
| 5 | **Knowledge scope** | Per-project | Per-researcher | Per-system (public data) | Per-project | Per-review |
| 6 | **Hypothesis handling** | First-class beliefs: proposed→investigating→supported→refuted→confirmed/rejected. Confidence updated. | Sigil agent: single-hypothesis hunting per invocation | Scheduler dispatches agents by KB match. No explicit lifecycle. | Findings validated in sandbox (binary) | Hypotheses with >0.8 confidence threshold. Single-pass. |
| 7 | **Planning** | Two-phase: Coverage sweep (90%) → Intuition/saliency deep dives | Human-directed. /summon initializes, researcher drives. | Scheduler assigns agents by code similarity to KB | Sequential 4-stage pipeline | 3-phase sequential analysis |
| 8 | **Verification method** | QA Finalizer over full source context | Familiar agent tries to DISPROVE findings ("skepticism with substance") | 3-stage merge: factual check → KB cross-validation → invariant check | Sandboxed reproduction in containers | Multi-stage self-verification by same model |
| 9 | **Multi-agent** | Strategist (senior) → Scout (junior) → Finalizer (QA). Concurrent teams. | 5 agents: Librarian, Scribe, Familiar, Gnome, Sigil | Scheduler → Rule + Logic + Project-Specific + General agents | Single agent, pipeline stages | Single agent, internal phases |
| 10 | **Skill/pattern generation** | No skill generation | Scribe distills confirmed findings → reusable detection modules (sigils) | No skill generation | No skill generation | No skill generation |
| 11 | **Skill/pattern format** | N/A | Checks: YAML frontmatter + grep patterns + assessment (30-line limit) | Vulnerability templates in KB | N/A | N/A |
| 12 | **Learning mechanism** | Belief revision within project. No cross-project learning. | Manual distillation per audit → personal grimoire accumulates | KB ingests new audit reports continuously | None | None |
| 13 | **Cross-project learning** | No | Personal grimoire carries sigils across audits (individual only) | Yes — KB generalizes across projects | No | No |
| 14 | **Organizational learning** | No | No (per-researcher only) | No (public data, not org-specific) | No | No |
| 15 | **Recall (benchmarked)** | 31.2% on ScaBench (5 projects, 109 issues) | No benchmarks published | 4/9 medium, 0/3 high on real audit. No recall number. | 45.6% detect on EVMbench | Not benchmarked on audit datasets |
| 16 | **Precision (benchmarked)** | 9.3% (high FP acknowledged) | No benchmarks | Not measured | <6% FP rate (general AppSec) | <6% FP rate (general AppSec) |
| 17 | **PoC generation** | Planned (not yet: "early PoC/test integration" in roadmap) | Full 6-phase workflow, 10 reference docs, ~90% one-shot success | No | Generates PoC exploits + patches | No dedicated PoC |
| 18 | **External data integration** | None | Solodit (20K+ findings), Context7 (library docs), vector search | Code4rena audit reports | None | None |
| 19 | **Retrieval method** | Reference-driven: graph edges → exact code cards (byte-accurate) | Semantic search (FastEmbed + Qdrant) + grep patterns | KB similarity matching | Repo-level analysis | Repo-level analysis |
| 20 | **Language support** | Language-agnostic (raw code cards, no AST) | Multi-language (tree-sitter for Rust/Solidity annotations) | Solidity-focused | Language-agnostic | Language-agnostic |
| 21 | **Adaptive planning** | Yes — Coverage→Intuition with configurable threshold | No — human drives the flow | No — Scheduler assigns once | No — fixed pipeline | No — fixed phases |
| 22 | **Concurrent operation** | Yes — multi-agent teams, shared belief set, lock-guarded stores | No — single researcher | Parallel agent execution supported | No | No |
| 23 | **Backpressure/safety** | QA Finalizer, confidence thresholds | Explicit backpressure philosophy: static analysis verifies, PoCs verify. "Agent outputs are radioactive." | 3-stage merge filtering | Sandboxed validation | 18 hard exclusions, >0.8 threshold |
| 24 | **Human-in-the-loop** | Optional — Strategist can be steered via inbox | Central — researcher drives, agents assist | Agents semi-automatically designed, human expert confirmation | Patches as reviewable PRs | Findings for human review |
| 25 | **Deployment model** | Open-source CLI tool | Claude Code plugin (open-source) | Research prototype | SaaS (Codex Web, GitHub integration) | Built into Claude Code (Enterprise/Team) |
