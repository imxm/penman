# Findings as Beliefs: Cross-Engagement Hypothesis Infrastructure for Automated Security Auditing

**April 2026**

---

## Abstract

Security audit findings are typically treated as binary facts: a vulnerability either exists or it doesn't. We propose treating findings as long-lived hypotheses — beliefs with confidence scores, evidence chains, and archetype associations that aggregate across engagements. The design is informed by direct experimentation: running Hound on Cosmos L1 chains revealed knowledge graph construction as the bottleneck (>1 hour, 3 unrelated findings), motivating an external structural topology builder (Cosmos World Mapper) and manual skill building (IBC Vulnerability Scanner, 17 patterns from 50+ public findings). [Grimoire](https://github.com/JoranHonig/grimoire) provides a promising framework for organizational knowledge (sigils, distillation, shared grimoire) but is alpha-stage with no organizational deployment. We present the concrete implementation: a system that processes organizational audit history (reports, code review discussions, fix branches) into a persistent belief store, producing (1) auto-generated detection modules in Grimoire-compatible check format, (2) hypothesis seeds for per-project tools like [Hound](https://github.com/scabench-org/hound), and (3) a foundation for combining structural security posture with organizational beliefs. Every component traces to a proven approach: belief lifecycle from Hound, knowledge base ingestion from LISA, detection module format from Grimoire, threat model enrichment from Codex Security, and confidence thresholds from Claude Code Security.

---

## 1. Introduction

Five state-of-the-art approaches to AI-assisted security auditing have emerged, each addressing a different layer of the problem:

**Hound** ([Mueller, 2025](https://github.com/scabench-org/hound)) introduces relation-first knowledge graphs and a persistent belief system for per-project code analysis. Hypotheses are first-class objects with confidence scores that update as evidence accrues. An adaptive planning policy transitions from coverage sweep to intuition-guided deep dives. On ScaBench (projects averaging ~725 LOC), Hound achieves 31.2% micro recall (vs. 8.3% baseline). However, Hound's agent-driven graph discovery struggles on large codebases: in our experience, a Cosmos L1 chain consumed over an hour of compute and produced 3 unrelated findings, with knowledge graph construction as the bottleneck. Hound supports an explicit mode where pre-built graphs can be fed directly — motivating external graph builders for domains where agent-driven discovery is impractical.

**Grimoire** ([Honig, 2026](https://github.com/JoranHonig/grimoire)) is a multi-agent Claude Code plugin (alpha v0.1.0, solo developer) that provides an opinionated audit workflow: initialization (`/summon` maps the codebase and spawns detection agents), cartography (documenting code flows for context rebuilding), hypothesis-driven hunting (sigil agents that investigate one vulnerability vector each), finding documentation, PoC generation (~90% one-shot success), and — critically — Scribe distillation, which transforms confirmed findings into reusable detection modules called "checks." These checks (structured markdown: YAML metadata + grep-able patterns + assessment instructions) accumulate in a personal grimoire (`~/.grimoire/`, git-tracked), creating a per-researcher knowledge base that grows across audits. If the grimoire repo is shared across a team, this becomes organizational infrastructure — though no team has deployed it this way.

**LISA** ([Sun et al., 2025](https://arxiv.org/html/2509.24698)) is an academic smart contract auditing framework that builds a knowledge base from historical audit reports (primarily from Code4rena competitive audits) containing vulnerability templates, severity gradings, code examples, and human reasoning annotations. A scheduler dispatches specialized agents (rule-pattern, logic vulnerability, and project-specific) based on KB similarity to the target code. On real audits, LISA detects medium-severity issues competitively but fails on high-severity bugs requiring deep semantic reasoning (0/3 high-severity on Size Meta Vault). No follow-up work has been published.

**Codex Security** ([OpenAI, 2026](https://openai.com/index/codex-security-now-in-research-preview/)) builds a project-specific threat model (entry points, trust boundaries, risky components), then scans commits using the threat model as context. Findings are validated in sandboxed containers. Over a 30-day beta: 1.2M commits scanned, 792 critical findings, <6% false positive rate.

**Claude Code Security** ([Anthropic, 2026](https://www.anthropic.com/news/claude-code-security)) reasons about code through three phases: repository context research, comparative analysis against established patterns, and vulnerability assessment with multi-stage self-verification. Each finding is treated as a hypothesis; only those with >0.8 confidence survive. Found 500+ vulnerabilities in production codebases that had gone undetected for years.

### 1.1 The Gap

These five tools occupy different layers of the audit stack. Understanding what each already provides — and what it doesn't — reveals the actual gap.

**Grimoire provides a promising framework for organizational audit knowledge** — but it's early-stage (alpha v0.1.0, solo developer, 7 weeks old, no production usage). The design is sound: personal grimoire (`~/.grimoire/`) is git-tracked, sigils accumulate across audits, `/summon` runs the sigil swarm against new codebases, cartography loads context. If a team shared a grimoire repo and a team lead curated sigils after each audit, this could function as organizational knowledge infrastructure.

But Grimoire is a framework, not a working organizational system. Significant roadmap remains unbuilt (tomes, flows, scoping, reporting). No team has deployed it organizationally. And even at maturity, Grimoire's design has gaps that Penman addresses:

- **No automated ingestion.** Scribe distillation is manual — one confirmed finding → one sigil, by a human. Processing 500+ past engagements retroactively requires automation Grimoire doesn't have.
- **No numeric confidence.** Sigils are qualitative (`confidence: high/medium/low`). There's no confidence derived from "this pattern appeared in 8/12 lending audits." No base rates, no archetype-specific frequency data.
- **No archetype-aware prioritization.** The sigil swarm during `/summon` runs all checks equally. It doesn't know that for a lending protocol, oracle checks should run before reentrancy checks.
- **No autoresearch loops.** Sigils don't self-improve from external data (reports, advisories, CVEs).
- **No detection chain reconstruction.** Sigils capture what to look for but not the analytical process that led to the finding.

Penman is the concrete implementation of what Grimoire's framework designs for. It produces Grimoire-compatible output so the two can compose when Grimoire matures.

**Hound's beliefs die when the project ends.** The hypothesis that "oracle staleness appears in lending protocols" — confirmed with 0.9 confidence on project A — provides zero benefit to project B. Hound's belief system is the most sophisticated per-project hypothesis management available, but it doesn't persist across projects.

**LISA's KB entries are static templates.** They don't accumulate confidence from repeated observation, don't update from fix branch feedback, and use public data (Code4rena) rather than private organizational history.

**Codex Security and Claude Code Security build understanding from scratch** per engagement and retain nothing.

The actual gap is narrower than "no tool does organizational knowledge" — Grimoire can, if operated organizationally. The gap is: **no tool automates the ingestion of organizational audit history into confidence-scored, archetype-specific beliefs that improve detection prioritization.** Grimoire has the infrastructure (sigils, spellbook, cartography). What's missing is the automated pipeline that populates it from historical data and adds the quantitative layer (confidence, base rates, archetype associations).

### 1.2 Motivation: Personal Experimentation

The design of this system is informed by direct experimentation with Hound, Grimoire's framework concepts, and manual skill building.

**Hound on Cosmos L1 chains.** Hound's knowledge graph building is designed for moderately-sized EVM projects (ScaBench benchmarks average ~725 LOC per project). On a Cosmos L1 chain — a significantly larger codebase with different architecture — Hound ran for over an hour and produced 3 unrelated findings. Debugging revealed the bottleneck was graph construction: Hound's agent-driven graph discovery consumes excessive tokens on large, unfamiliar codebases. This motivated building the **Cosmos World Mapper** — a static analyzer that produces 3-layer knowledge graphs for Cosmos SDK chains: structural topology (modules, keepers, message routes), execution topology (transaction flows, state transitions), and security topology (trust boundaries, attack surfaces). Built using tree-sitter AST parsing with zero LLM speculation, producing interactive HTML visualization. Tested on Sei, Cosmos Hub, and Osmosis. Hound supports an explicit mode where pre-built knowledge graphs can be fed directly, bypassing its slow agent-driven discovery. Feeding World Mapper's security topology into Hound qualitatively improved its hypotheses, though no measurable results were obtained — both tools are still in the experimentation stage.

**Manual skill building.** The **IBC Vulnerability Scanner** (17 patterns from 50+ public findings) was built by manually gathering reports from Zellic, Informal Systems, Oak Security, and ibc-go GHSAs → summarizing → finding patterns → aggregating into a Claude Code skill. 50% false positive rate, but surfaces real issues quickly. This manual process — gather, summarize, pattern, aggregate — is exactly what Penman automates.

**The manual workflow.** World Mapper produces an HTML visualization of the codebase's security topology. This visualization helps craft specific investigation prompts for Hound, which then investigates independently. The IBC Scanner provides domain-specific detection patterns that could seed Hound's hypothesis system. All of this is currently manual — no MCP integration, no automated pipeline. Grimoire's framework (sigil distillation, `/summon` workflow, systematic cartography) provides the systematic structure for what was being done ad hoc.

**The insight.** If structural security posture (from World Mapper's topologies) is joined with organizational beliefs (from historical findings), it combines the power of both: structural understanding of the current codebase + organizational knowledge of what typically goes wrong in this archetype. Neither alone is sufficient. Hound without organizational beliefs starts blind. Beliefs without structural understanding can't be grounded in the specific codebase. The combination is the full picture.

### 1.3 Contribution

We propose a **concrete implementation of organizational audit knowledge infrastructure** — the system that Grimoire's framework designs for but doesn't yet build. Penman processes historical audit data into confidence-scored detection modules, with output in Grimoire-compatible format. It adds what no existing tool provides: automated bulk ingestion, numeric confidence from cross-engagement frequency, archetype-specific prioritization, and autoresearch loops for module self-improvement.

The core abstraction is the **belief object** — an extension of Grimoire's sigil with Hound's confidence model:

```
Belief {
  hypothesis:       "Oracle staleness in lending protocols with Chainlink dependency"
  archetype:        "lending-oracle"
  confidence:       0.67          // found in 8/12 matching engagements
  evidence:         [ENG-042, ENG-057, ENG-071, ...]
  detection_chain:  ["identify oracle source", "check staleness threshold", ...]
  fix_pattern:      "add heartbeat check on updatedAt"
  status:           confirmed     // Hound's lifecycle: proposed→investigating→supported→confirmed
}
```

Beliefs aggregate across engagements. Confidence is the frequency of occurrence across matching-archetype audits. Evidence chains link back to specific engagements. Fix branches update beliefs: a fix confirms the hypothesis; no fix (when the finding was valid) leaves it unchanged; a false-positive determination reduces confidence.

The system produces two concrete outputs:
1. **Detection modules** in Grimoire-compatible check format (YAML + grep patterns + assessment, 30-line limit) — directly loadable into Grimoire's spellbook
2. **Hypothesis seeds** for Hound's Strategist — pre-populated belief sets with organizational confidence, enabling informed Coverage→Intuition planning from step 1

---

## 2. Background: What Each Tool Provides

### 2.1 Hound's Belief System

Hound's key contribution is treating findings as first-class belief objects with explicit lifecycle management. A hypothesis is a tuple:

```
h = (title, type, severity, confidence ∈ [0,1], status, evidence ⊆ C, reasoning)
```

Status transitions: `proposed → investigating → supported → refuted → confirmed | rejected`. Confidence updates as evidence accrues. The QA Finalizer issues verdicts (`confirmed | rejected | uncertain`) over full source context.

Notably, Hound's QA Finalizer → human review → confirmed finding (confidence=1.0) pipeline is structurally similar to Grimoire's Scribe distillation → human verification → reusable sigil pipeline. Both tools converge on the same pattern: machine-generated hypothesis → human verification → concrete artifact. This convergence validates the pattern Penman builds on.

**What we borrow:** The belief lifecycle and confidence model. **What we extend:** Beliefs persist across projects. Confidence aggregates from multiple engagements. Evidence chains span organizational history.

### 2.2 Grimoire's Organizational Infrastructure

Grimoire provides the most complete framework design for organizational audit knowledge among the five tools — though it remains alpha-stage with no organizational deployment. Key components:

**Sigil system:** Scribe distills confirmed findings → reusable detection modules (sigils). Sigils accumulate in `~/.grimoire/sigils/` (git-tracked). `/summon` runs the sigil swarm against every new codebase. If the grimoire is shared across a team, this is organizational detection infrastructure.

**Cartography:** Maps and documents code flows, loading context from sigils and knowledge artifacts. For an organizational grimoire, this means every new audit starts with the organization's accumulated detection patterns.

**Check format:**

```yaml
---
name: oracle-staleness
description: Check for missing freshness validation on oracle price feeds
languages: [solidity]
severity-default: high
confidence: high
tags: [oracle, defi, lending]
---
## Patterns
- `latestRoundData` without subsequent `updatedAt` check
- `getPrice` consuming raw oracle output without staleness validation

## Assessment
Verify that the oracle consumer checks the `updatedAt` timestamp against a staleness threshold...
```

**What Grimoire designs for (but hasn't built):** Organizational knowledge accumulation via shared grimoire, detection module format (checks), sigil swarm execution, cartography context loading, manual distillation pipeline. The framework is promising but alpha-stage — no organizational deployment exists.

**What Penman concretely implements:** Automated bulk ingestion from historical audit data (Grimoire requires manual distillation per finding). Confidence as a numeric score derived from cross-engagement frequency (Grimoire checks have `confidence: high/medium/low` — qualitative, not data-derived). Archetype-specific prioritization for detection modules. Autoresearch loops for module self-improvement from external data. Output in Grimoire-compatible format so the two compose when Grimoire matures.

### 2.3 LISA's Knowledge Base Ingestion

LISA extracts structured information from audit reports: vulnerability templates, severity gradings, code examples, human reasoning annotations. The KB supports metadata (project size, complexity, language version) for context-aware agent dispatch.

**What we borrow:** The ingestion pipeline concept and structured extraction. **What we extend:** Private organizational data instead of public Code4rena. Belief aggregation instead of static templates. Fix branch feedback.

### 2.4 Codex Security's Threat Model

Codex Security generates a project-specific threat model capturing entry points, trust boundaries, auth assumptions, and risky components. The threat model contextualizes all subsequent scanning.

**What we borrow:** The concept of enriching per-project analysis with structured context. **What we extend:** Threat models seeded with archetype-specific beliefs from organizational history, not built from repository analysis alone.

### 2.5 Claude Code Security's Confidence Thresholds

Claude Code Security filters findings by confidence (>0.8 threshold). Each finding is treated as a hypothesis that must survive self-verification. 18 hard exclusions automatically filter noise.

**What we borrow:** Confidence-based filtering. **What we extend:** Confidence derived from cross-engagement frequency, not single-pass model estimation.

---

## 3. System Architecture

### 3.1 Ingestion

Two data sources per engagement:

- **Audit reports (PDF):** Final findings with confirmed severity, descriptions, code references.
- **Code review discussions (GitHub PRs):** Individual auditor findings, severity debates, false positive flags, consensus process.

Reconciliation fills gaps: GitHub has discussion context; PDFs have final severity. Fix branches add resolution status.

Extracted per finding:
- Finding description + code snippet with expanded context
- Severity (initial, final, escalation rationale)
- Fix status (fixed / not fixed / client-found)
- Discussion summary (FP flags, severity arguments)
- Protocol characteristics (type, dependencies, architecture)

### 3.2 Detection Chain Reconstruction

The analytical process that produced each finding was never recorded. We use LLM-based reconstruction:

**Input:** finding type + code context + protocol characteristics
**Output:** plausible detection chain (the analytical steps an auditor would follow)

This is inference, not recovery. Validated by domain expert review on a sample. Reconstruction quality improves as auditors voluntarily annotate their actual process during live audits.

### 3.3 Belief Creation and Aggregation

Each finding, once extracted and reconstructed, becomes a belief object. Beliefs with matching hypotheses across engagements are merged:

```
merge(belief_A, belief_B):
  confidence = (A.evidence_count + B.evidence_count) / total_matching_archetype_audits
  evidence = A.evidence ∪ B.evidence
  detection_chain = longest_common_subsequence(A.chain, B.chain) + unique_steps
  fix_pattern = most_common(A.fix, B.fix)
```

Confidence is a simple frequency: how often this hypothesis was confirmed across matching-archetype engagements. No Bayesian inference, no learned weights — just counting. This is intentional: transparency over sophistication.

### 3.4 Fix Branch Feedback

When a fix branch is reviewed:
- **Finding fixed:** Belief confidence unchanged (confirms the hypothesis was valid)
- **Finding not fixed (client disagreed):** Belief gets a `disputed` evidence entry — reduces confidence if pattern repeats
- **Client found the issue independently:** Belief gets a `client_confirmed` evidence entry — increases confidence
- **Finding was FP:** Belief confidence decreases. If confidence drops below threshold, belief status → `refuted`

### 3.5 Detection Module Generation

Each belief with confidence above a threshold (default: 0.3, configurable) generates a Grimoire-compatible check:

```
belief_to_check(belief):
  patterns = extract_grep_patterns(belief.detection_chain)
  assessment = generate_assessment(belief.hypothesis, belief.evidence, belief.fix_pattern)
  return Check(
    name = slugify(belief.hypothesis),
    description = belief.hypothesis,
    severity_default = most_common_severity(belief.evidence),
    confidence = belief.confidence,
    tags = [belief.archetype] + extract_tags(belief.detection_chain),
    patterns = patterns,
    assessment = assessment
  )
```

Each generated check runs an autoresearch loop: gather additional public data (reports, advisories, CVEs) → find new matching patterns → refine detection instructions. Same process used to build tools like IBC vulnerability scanners manually, now automated per check.

### 3.6 Belief Seeding for Per-Project Tools

**For Hound:** Export beliefs matching the target project's archetype as a pre-seeded hypothesis set. Hound's Strategist receives:
```
seed_beliefs = filter(belief_store, archetype=project_archetype)
# Each seed becomes a Hound hypothesis with:
# - title, type, severity from belief
# - confidence from organizational base rate
# - status = "proposed" (needs per-project verification)
# - reasoning = "organizational base rate from N engagements"
```

The Strategist's planning shifts: instead of blind Coverage sweep, it begins with archetype-informed Intuition guided by organizational priors. Coverage sweep still runs for completeness, but high-confidence beliefs get investigated first.

**For Grimoire:** Export beliefs as archetype-ordered sigil set for `/summon`. Instead of running all checks with equal priority, the sigil swarm runs high-confidence checks first for the detected archetype.

**For generic agents:** Export as domain-specific hypothesis trees — the planning layer for any agent framework that accepts structured input.

### 3.7 Structural Topology + Organizational Beliefs (Future Integration)

As described in Section 1.2, the full picture combines structural security posture (WHERE to look in this codebase — from static topology analysis like the Cosmos World Mapper's 3-layer knowledge graphs) with organizational beliefs (WHAT to look for — from Penman's cross-engagement belief store). Preliminary experimentation shows qualitative improvement when feeding World Mapper's security topology into Hound's explicit knowledge graph mode, though results are not yet measurable.

This integration is post-MVP. The MVP focuses on the organizational belief pipeline. Structural topology seeding would be formalized in a follow-on stage.

---

## 4. Evaluation Framework

### 4.1 Belief Calibration (MVP)

**Question:** Does belief confidence correlate with actual finding frequency?

**Method:** Hold out 20% of engagements. Build beliefs from 80%. For each held-out engagement, check: do high-confidence beliefs (>0.6) for the matching archetype correspond to actual findings? Measure calibration: predicted probability vs. observed frequency.

### 4.2 Belief-Seeded Hound (Post-MVP)

**Question:** Does seeding Hound with organizational beliefs improve recall?

**Method:** Run Hound on held-out projects in two conditions:
- **Unseeded:** Standard Hound (blind Coverage→Intuition)
- **Seeded:** Hound with pre-populated beliefs from organizational store

Measure: recall, precision, F1, time-to-first-finding. Hound's existing ScaBench methodology provides the evaluation framework.

### 4.3 Archetype-Ordered Grimoire Swarm (Post-MVP)

**Question:** Does archetype-aware check ordering improve efficiency?

**Method:** Run Grimoire's sigil swarm on held-out projects in two conditions:
- **Random order:** All checks run with equal priority
- **Archetype-ordered:** High-confidence checks for detected archetype run first

Measure: time to first valid finding, total valid findings within fixed time budget.

### 4.4 Structural Topology + Beliefs (Post-MVP)

**Question:** Does combining structural topology seeding with organizational belief seeding produce better results than either alone?

**Method (when World Mapper integration is ready):** Run Hound on held-out projects in four conditions:
- **Baseline:** Unseeded Hound
- **Topology only:** Hound with World Mapper's structural topology
- **Beliefs only:** Hound with organizational belief seeds
- **Combined:** Hound with both topology and beliefs

Measure: recall, precision, time-to-first-finding across conditions.

### 4.5 Detection Module Quality (MVP)

**Question:** Do auto-generated checks match manually-crafted check quality?

**Method:** Compare auto-generated checks against manually-crafted equivalents (e.g., IBC Vulnerability Scanner's 17 patterns) on pattern coverage, false positive rate, and detection accuracy on held-out code.

---

## 5. Relationship to Existing Work

This system builds explicitly on the shoulders of five tools. We do not claim novelty for any individual component:

| Component | Source | What we borrow | What we add |
|-----------|--------|----------------|-------------|
| Organizational knowledge framework | Grimoire | Framework design: shared grimoire, sigil swarm, cartography, check format, Scribe distillation. Alpha-stage, no organizational deployment. | Concrete implementation. Automated bulk ingestion from historical data. Grimoire designs for this — Penman builds it. Output is Grimoire-compatible. |
| Belief lifecycle | Hound | proposed→confirmed lifecycle, confidence scoring | Cross-engagement persistence. Hound's beliefs die with the project — we make them organizational. |
| Numeric confidence | Hound + Claude Code Security | Confidence as a number (Hound: 0-1, Claude: >0.8 threshold) | Confidence derived from cross-engagement frequency, not per-project estimation. Extends Grimoire's qualitative `confidence: high/medium/low` to data-derived numeric scores. |
| KB ingestion | LISA | Structured extraction from audit reports | Private organizational data (not public Code4rena). Fix branch feedback. Detection chain reconstruction. |
| Threat model seeding | Codex Security | Project-specific context enrichment | Archetype-specific beliefs seed per-project threat models. |
| Archetype-specific prioritization | None (new) | N/A | Grimoire's sigil swarm runs all checks equally. We add archetype-based ordering: for lending protocols, run oracle checks first because base rate is highest. |
| Autoresearch loops | None (new) | N/A | Each sigil autonomously gathers additional patterns from external data (reports, advisories, CVEs) and refines itself. |

**The contribution is the concrete implementation of what Grimoire's framework designs for.** Grimoire provides the framework (sigils, swarm, cartography, distillation) but is alpha-stage with no organizational deployment. Penman builds the automated pipeline that makes organizational audit knowledge real: historical data processing, confidence scoring, archetype prioritization, autoresearch loops. Output is Grimoire-compatible — when Grimoire matures, the two compose.

**What we cannot claim:**
- That automated distillation matches manual Scribe distillation quality (evaluation needed)
- That numeric confidence is better than Grimoire's qualitative assessment (evaluation needed)
- That archetype-ordered swarms outperform equal-priority swarms (evaluation needed)
- That organizational beliefs are better than per-project discovery (Hound may find things beliefs miss)
- That this approach generalizes beyond the specific organization's data

---

## 6. Limitations

1. **Grimoire could eventually provide this.** If Grimoire matures beyond alpha, adds numeric confidence to its check format, and a team builds the discipline to manually curate a shared grimoire across hundreds of engagements, it could provide organizational knowledge without Penman. Penman's value proposition depends on automation being significantly more practical than manual curation at scale — which is likely for 500+ engagements but unproven.

2. **Detection chain reconstruction is inference.** The actual auditor process was never recorded. Reconstructed chains are plausible approximations, not ground truth.

3. **Automated distillation may produce worse sigils than manual Scribe distillation.** Grimoire's manual process has a human expert verifying each sigil. Automated bulk generation may trade quality for quantity. The 30-line check format was designed for human authoring — automated generation within this constraint requires evaluation.

4. **Belief confidence is frequency-based.** A belief confirmed in 8/12 lending audits gets 0.67 confidence. But if the 12 audits were all similar lending protocols, this may overfit. Archetype granularity matters. It's also possible that Grimoire's qualitative `confidence: high/medium/low` is more useful in practice than numeric scores.

5. **Organizational data creates organizational bias.** Beliefs reflect what the organization has found historically. Vulnerability classes the organization has never encountered will have no beliefs. Grimoire's Librarian already addresses this partially by searching external knowledge (Solodit, Context7) — our system needs similar external augmentation.

6. **Evaluation requires held-out data.** With 70+ engagements, holding out 20% leaves ~14 for evaluation. Statistical power is limited.

7. **Archetype-ordered swarm may not outperform random.** Grimoire's equal-priority sigil swarm may be optimal if the agent parallelizes well. Prioritization only helps if there's a meaningful time constraint where checking high-confidence patterns first saves effort.

8. **Integration with Hound is theoretical.** We describe how beliefs seed Hound's Strategist, but Hound's planning model may not benefit from external seeds in the way we predict. Hound's strength is per-project graph-based discovery — organizational priors might constrain rather than guide its exploration.

---

## 7. Conclusion

We propose a concrete implementation of organizational audit knowledge infrastructure — the system that Grimoire's alpha-stage framework designs for but doesn't yet build. Penman processes historical audit data into confidence-scored detection modules with archetype-specific prioritization and autoresearch self-improvement. The belief model borrows from Hound (lifecycle, confidence), the ingestion approach from LISA (structured extraction from past reports), the detection module format from Grimoire (checks), and the quality framework from Claude Code Security (confidence thresholds) and Codex Security (threat model enrichment). Output is Grimoire-compatible, enabling composition when Grimoire matures.

Grimoire provides the framework. Hound provides the per-project analysis. LISA validates the learn-from-past-audits approach. Penman provides the concrete, automated pipeline that turns organizational audit history into the structured knowledge these tools need.

---

## References

1. B. Mueller. "Hound: Relation-First Knowledge Graphs for Complex-System Reasoning in Security Audits." September 2025. https://github.com/scabench-org/hound
2. J. Honig. "Grimoire: A Security Research Toolkit That Learns." February 2026. https://github.com/JoranHonig/grimoire
3. I. Sun, D. Tan, A. Deng. "LISA: An Agentic Framework for Smart Contract Auditing." September 2025. https://arxiv.org/html/2509.24698
4. OpenAI. "Codex Security: Now in Research Preview." March 2026. https://openai.com/index/codex-security-now-in-research-preview/
5. Anthropic. "Making Frontier Cybersecurity Capabilities Available to Everyone." February 2026. https://www.anthropic.com/news/claude-code-security
6. OpenAI, Paradigm. "Introducing EVMbench." February 2026. https://openai.com/index/introducing-evmbench/
7. Trail of Bits. "How We Made Trail of Bits AI-Native (So Far)." March 2026. https://blog.trailofbits.com/2026/03/31/how-we-made-trail-of-bits-ai-native-so-far/
8. Trail of Bits. "246 Findings from Our Smart Contract Audits: An Executive Summary." August 2019. https://blog.trailofbits.com/2019/08/08/246-findings-from-our-smart-contract-audits-an-executive-summary/
9. Anthropic. "Claude Code Security Review." 2026. https://github.com/anthropics/claude-code-security-review
10. I. Nonaka, H. Takeuchi. "The Knowledge-Creating Company." Oxford University Press, 1995.
