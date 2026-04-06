# 1. Title

Penman: Organizational Knowledge Infrastructure for AI-Native Security Auditing

# 2. Problem

AI has made every auditor individually more productive. Coverage is up. Detection speed is up. The multi-auditor model produces more overlapping findings now, but that's a sign the tools are working — AI broadens each auditor's reach beyond their original specialty. Oak's auditor matching, performance tracking, and feedback systems are strong: post-audit hack rates are very low. None of this is broken.

What doesn't yet exist is infrastructure that turns individual AI productivity into organizational capability. An [NBER study of 6,000 executives](https://fortune.com/2026/02/17/ai-productivity-paradox-ceo-study-robert-solow-information-technology-age/) found 89% saw no organizational productivity change despite rising AI adoption. [MIT found only 5% of companies](https://inferencebysequoia.substack.com/p/the-ai-productivity-paradox-high) generate measurable organizational value from AI. The pattern is consistent across industries, including professional services like security auditing: individual productivity gains require deliberate infrastructure to become organizational capability.

Oak has 500+ engagements of analytical process — threat models, hypothesis chains, detection shortcuts, cross-protocol analogies — locked inside PDF reports and closed context windows. This isn't a failure; it's latent value. Structured and extracted, this history becomes:

- **Proprietary training data** for domain-specific AI models no competitor can replicate
- **Data-driven audit scoping** — pricing and time estimation grounded in actual base rates, not memory
- **Auditor development infrastructure** — new auditors learn from the organization's full history, not just their own engagements
- **Foundation for cyber reasoning systems**, continuous monitoring, and automated pre-audit scanning
- **Defensibility against AI commoditization** — when [generic detection hits >70% exploit rate](https://openai.com/index/introducing-evmbench/) on known bugs, every firm has the same baseline capabilities. The differentiator becomes organizational knowledge that generic AI doesn't have access to

Trail of Bits showed what this looks like in practice: [201 skills, 84 agents, 200 bugs/week](https://blog.trailofbits.com/2026/03/31/how-we-made-trail-of-bits-ai-native-so-far/) — built not by making better tools, but by building organizational infrastructure that makes tools compound. Oak's marketplace model is different from ToB's in-house model, but the principle is the same: individual excellence needs a system to compound into organizational capability.

# 3. Solution

Penman builds the organizational knowledge pipeline that tools with automated ingestion of historical audit data, numeric confidence from cross-engagement frequency, archetype-specific prioritization, and autoresearch loops for detection module improvement.

Penman processes Oak's audit history — PDF reports, GitHub PR discussions, fix branches — into confidence-scored detection modules and structured knowledge. It automates and scales the process I've been doing manually in my own audit workflow:

- I built a private **IBC Vulnerability Scanner** — a Claude Code skill with 17 detection patterns extracted from 50+ public IBC security findings (Zellic, Informal Systems, Oak Security, ibc-go GitHub Security Advisories). Each pattern contains: vulnerable code example, correct fix, and authoritative references. Built by manually gathering reports → summarizing → finding patterns → aggregating into a structured skill. 50% false positive rate in practice, but surfaces real issues fast.
- I built a **Cosmos World Mapper** — a static analyzer that produces 3-layer knowledge graphs for Cosmos SDK chains (structural topology: modules/keepers/message routes → execution topology: transaction flows/state transitions → security topology: trust boundaries/attack surfaces). Built using tree-sitter AST parsing with zero LLM speculation. Tested on Sei, Cosmos Hub, and Osmosis. Produces interactive HTML visualization. I built this because [Hound](https://github.com/scabench-org/hound) (an open-source AI auditor that uses knowledge graphs and a belief system for per-project analysis) ran for over an hour on a Cosmos L1 chain and produced only 3 unrelated findings — its agent-driven graph construction is too slow and token-intensive for large L1 codebases. Hound supports an explicit mode where pre-built knowledge graphs can be fed directly, bypassing agent-driven discovery. World Mapper produces the structural graph that Hound can't build efficiently on these codebases.
- I've been manually using World Mapper's HTML visualization to craft specific investigation prompts, then feeding those to Hound for independent hypothesis-driven investigation.

All of this is manual, experimental, and not yet measurable. Grimoire's framework provides the systematic structure (sigil distillation, swarm execution, cartography) for what I've been doing ad hoc. Penman automates the upstream pipeline: ingest historical data → classify → generate detection modules → autoresearch loops → feed into per-project tools like Hound and Grimoire.

**How it works:**

1. **Ingest.** Pull PDF reports and GitHub PR discussions (via API) for each engagement. Reconcile the two sources — GitHub has severity debates, FP flags, and auditor disagreements; PDFs have final findings and severity. Extract: finding with description, severity, code snippet with expanded context, fix status, discussion summary.

2. **Reconstruct.** For each finding, reconstruct the plausible detection chain — the analytical steps that would lead an auditor from entry point to finding. This is inference based on finding type, code context, and protocol characteristics (the actual process was never recorded).

3. **Classify.** Cluster findings across dimensions — protocol type, vulnerability characteristics, detection approach. Domain taxonomy is iterative: it emerges from the data combined with existing taxonomies (e.g., [Cosmos Security Handbook](https://x.com/AkhtarievRoman/status/1747325049308381539) categories: non-determinism, in-protocol panics, unmetered computation, key malleability, fee market).

4. **Generate detection modules.** Each meaningful cluster becomes a draft detection module in two formats: (a) Grimoire-compatible checks — structured markdown files with YAML metadata (name, severity, confidence, tags), grep-able search patterns, and assessment instructions that agents use to scan codebases; and (b) Claude Code skills. Each module carries a numeric confidence derived from cross-engagement frequency (e.g., "found in 8/12 lending audits = 0.67 confidence"). Each generated module is manually verified, then runs its own autoresearch loop: gather additional public data → find new patterns → refine.

5. **Compound.** Every new engagement feeds back. Auditors can add patterns during live audits. Fix branches update confidence (fix confirms the finding; FP determination reduces confidence). Detection modules and the knowledge base improve with every engagement.

**Future direction (post-MVP):** The full picture combines two types of knowledge seeding for per-project analysis tools: (1) structural security posture from static topology analysis (what World Mapper produces — understanding of the current codebase's architecture and attack surface) and (2) organizational beliefs from historical audit data (what Penman produces — knowledge of what typically goes wrong in this protocol archetype). Neither alone is sufficient. The MVP focuses on the organizational belief pipeline. Structural topology integration is a follow-on stage.

**What this unlocks:**

| Dimension | What Penman enables |
|-----------|-------------------|
| Auditor tooling | Domain-specific detection modules grounded in Oak's engagement history |
| Audit consistency | Shared organizational detection patterns, not individual memory |
| Archetype prioritization | Base rates from structured data — run oracle checks first on lending protocols because the base rate is highest |
| Auditor development | New auditors inherit the organization's full structured detection library |
| Future infrastructure | Proprietary training data, foundation for belief-seeded analysis tools (Hound, Grimoire) |
| Institutional memory | Detection patterns and confidence scores persist beyond individual auditor tenure |

# 4. MVP Scope

$5,000 over 3-4 weeks. I currently have access to 70+ Oak engagements (PDF reports + GitHub PR discussions via API). If Oak provides access to all 500+ engagements, the knowledge base becomes significantly more comprehensive.

The approach is validated by the personal experimentation described in Solution above — the IBC Scanner and World Mapper demonstrate the manual process that Penman automates. Both are experimental and not yet measurable.

Budget: ~$2,500 researcher time (ingestion pipeline, schema iteration, manual skill verification), ~$2,500 LLM API costs (extraction, reconstruction, classification, autoresearch loops across 70+ engagements).

Architecture follows [Karpathy's LLM Knowledge Base methodology](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): raw inputs → LLM processing → structured markdown with automated linting. Runtime: Claude Code with custom skills. Storage: structured markdown in git (versionable, diffable, auditable).

**Week 1: Ingestion and schema.** Build extraction pipeline for PDF reports and GitHub PR discussions. Reconcile both sources per engagement — findings, severity (including severity debates from GitHub), code snippets with expanded context, fix status, FP flags. Iterate on schema: start with core fields (finding, severity, code context, protocol type, vulnerability characteristics), validate against 5 manually curated engagements, refine. Deliverable: working ingestion pipeline, validated schema, 10+ engagements processed.

**Week 2: Classification and reconstruction.** Process all 70+ engagements. Reconstruct plausible detection chains for each finding. Cluster findings across dimensions — domain taxonomy emerges iteratively from data combined with existing taxonomies. Identify which clusters naturally scope into skills. Deliverable: structured knowledge base, classification across all available engagements, cluster map.

**Week 3: Detection module generation and autoresearch.** Auto-generate draft detection modules from each meaningful cluster — [Grimoire](https://github.com/JoranHonig/grimoire) compatible checks (YAML + grep patterns + assessment) and Claude Code skills, each with numeric confidence from cross-engagement frequency. Manually verify each generated module. Set up autoresearch loop per module — same process as IBC scanner (gather public data → find patterns → aggregate → refine), now automated. Deliverable: first batch of auto-generated and verified detection modules, working autoresearch loop.

**Week 4: Integration and documentation.** Run generated detection modules against held-out engagements to validate pattern quality. Package detection modules as a shared git repo (Claude Code plugin + Grimoire-compatible sigil directory) that any Oak auditor can load into their workflow. Document methodology, results, and classification taxonomy. Deliverable: validated detection modules deployed as shared plugin, methodology documentation.

**Success criteria:**
- 70+ engagements processed: 350+ findings extracted, reconstructed, and clustered
- 30-50 auto-generated detection modules, each manually verified:
  - Grimoire-compatible checks (YAML + grep patterns + assessment) — automated sigil distillation at scale
  - Claude Code skills — detection instructions, code patterns, assessment guidance
- Each module carries numeric confidence from cross-engagement frequency (e.g., "oracle staleness in X/Y lending audits")
- Detection modules validated against held-out engagements
- Autoresearch loop running per module (gather public data → find new patterns → refine)
- Domain taxonomy documented with archetype-specific base rates
- Modules deployed as shared git repo (Claude Code plugin + Grimoire-compatible directory) — every Oak auditor gets the organization's accumulated pattern library before reading a single line of code

**Risks and mitigations:**
- *Reconstruction quality:* Inferred detection chains may not match actual auditor process. Mitigation: this is expected — plausible reconstruction is the goal, not perfect recovery. Skills are manually verified before use.
- *Domain taxonomy ambiguity:* Scoping skills by domain vs. vulnerability class vs. hybrid is an open design question. Mitigation: let clusters emerge from data, validate with manual review, iterate. The IBC scanner (domain-scoped, multiple vulnerability classes) is the working model.
- *Schema iteration:* Schema will evolve during ingestion. Mitigation: budget Week 1 for 2-3 iterations validated against manually curated entries before scaling.

**If MVP succeeds — roadmap:**

- **Stage 2:** Scale to all 500+ Oak engagements. Build belief seeding for Hound (pre-populated hypothesis sets from organizational confidence) and Grimoire (archetype-ordered sigil swarms). Integrate World Mapper's structural topology with Penman's beliefs — WHERE to look (from codebase structure) + WHAT to look for (from organizational history). MCP server integration for live chain data during agentic flows.
- **Stage 3:** Pre-audit agent combining structural topology + organizational beliefs + agentic search to produce preliminary findings before human auditors begin. Eval pipeline measuring recall/precision against held-out audits. Proprietary training data from structured engagement history for domain-specific AI models.
- **Stage 4:** Continuous monitoring — detection modules run against protocol upgrades post-audit. Cyber reasoning system foundations. Cross-ecosystem expansion beyond Cosmos/EVM.

# 5. Competition

The AI audit tooling landscape has expanded significantly in 2025-2026. Tools fall into distinct layers, and Penman sits at the organizational knowledge layer — upstream of per-project analysis.

**Audit toolkits and per-project analysis:**

| Tool | What it does | Facts |
|------|-------------|-------|
| [Grimoire](https://github.com/JoranHonig/grimoire) | Multi-agent audit toolkit ("a toolkit that learns") | Alpha v0.1.0, solo developer, 7 weeks old. Promising framework: sigils, Scribe distillation, shared grimoire via git. Significant roadmap unbuilt (tomes, flows, reporting). No production usage. |
| [Hound](https://github.com/scabench-org/hound) | Knowledge graphs + belief system for per-project analysis | 31.2% recall on ScaBench (vs. 8.3% baseline). Persistent hypotheses with confidence. Adaptive planning: coverage sweep → intuition deep dives. In my experience, knowledge graph building is too slow on L1 chains (1 hour, 3 unrelated findings on a Cosmos project) — which motivated building World Mapper as a faster alternative for structural topology. |

Grimoire (a Claude Code plugin that structures the audit workflow into agents for research, distillation, QA, and hunting) provides the best framework design for organizational audit knowledge, but it's early-stage with no production deployment. Penman is the concrete implementation: an automated pipeline that processes historical audit data into confidence-scored detection modules. Output is Grimoire-compatible, so Penman's modules can feed into Grimoire when it matures. Hound (a CLI tool that builds knowledge graphs of a codebase and manages vulnerability hypotheses with confidence scores) provides the most sophisticated per-project analysis — Penman's belief seeds could inform Hound's planning from step 1, rather than starting blind.

**AI security analysis (per-project, start from zero):**

| Tool | What it does | Facts |
|------|-------------|-------|
| [Codex Security (OpenAI)](https://openai.com/index/codex-security-now-in-research-preview/) | Threat model → agentic search → sandboxed validation | 1.2M commits scanned, <6% FP rate. Builds project-specific threat model from scratch each time. |
| [Claude Code Security (Anthropic)](https://www.anthropic.com/news/claude-code-security) | Context research → hypothesis generation → self-verification | 500+ production vulns found. >0.8 confidence threshold. Reasons about code "the way a human researcher would." |

Both are powerful. Both start from zero on every engagement. Organizational knowledge (archetype-specific base rates, detection chains from past audits) could enrich both: Codex's threat model with archetype priors, Claude Code's hypothesis generation with organizational frequency data.

**Scanners:**

| Tool | What it does | Facts |
|------|-------------|-------|
| [Nethermind AuditAgent](https://www.nethermind.io/blog/how-nethermind-security-uses-auditagent-alongside-manual-audits) | AI pre-audit scanning | 30% avg recall, 42% on criticals. |
| [Savant Chat](https://savant.chat/) | Multi-agent parallel scanning | 6th in Sherlock contest vs. humans. |
| [Octane Security](https://www.octane.security/) | CI/CD security at PR level | $6.75M seed. Clients: Uniswap, Avalanche. |
| [EVMbench (OpenAI+Paradigm)](https://openai.com/index/introducing-evmbench/) | Benchmark | Best agent: 45.6% detect, 72.2% exploit. Generic detection commoditizing. |

Scanners find bugs in code. Penman structures organizational knowledge. Different layer.

**Learning from past audits:**

| Tool | What it does | Facts |
|------|-------------|-------|
| [LISA](https://arxiv.org/html/2509.24698) | KB from past Code4rena reports → agent dispatch | 4/9 medium found, 0/3 high on real audit. Academic, no follow-up. |
| [Sherlock AI](https://sherlock.xyz/solutions/ai) | Trained on competitive audit findings corpus | Largest contest data corpus in Web3. |
| [Trail of Bits](https://blog.trailofbits.com/2026/03/31/how-we-made-trail-of-bits-ai-native-so-far/) | 201 skills, 84 agents, organizational infrastructure | Maturity matrix, feedback loops. Ahead on implementation. |

LISA, Sherlock AI, and Trail of Bits all learn from past audits. The concept isn't unique. LISA uses public data. Sherlock uses contest data. ToB uses their own.


