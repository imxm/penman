# Penman

Penman builds organizational knowledge infrastructure for security auditing — an automated pipeline that processes audit history (PDF reports, GitHub PR discussions, fix branches) into confidence-scored detection modules and structured knowledge. It automates and scales what auditors do manually: gather findings from past engagements, find patterns, and aggregate them into reusable detection tools. Each module carries numeric confidence derived from cross-engagement frequency and self-improves via autoresearch loops.

## Documents

- **[Proposal](proposal.md)** — Grant proposal for Oak Security's Solidified Research Programme ($5K seed, 3-4 weeks)
- **[Whitepaper](paper.md)** — "Findings as Beliefs: Cross-Engagement Hypothesis Infrastructure for Automated Security Auditing" — technical architecture, competitive analysis, evaluation framework

## Context

The design is informed by personal experimentation with existing tools:

- **[Hound](https://github.com/scabench-org/hound)** — Per-project knowledge graphs + belief system. Powerful on small codebases (31.2% recall on ScaBench), but knowledge graph construction struggles on large L1 chains. Motivating the need for pre-built structural topology and organizational belief seeding.
- **[Grimoire](https://github.com/JoranHonig/grimoire)** — Multi-agent audit toolkit (alpha v0.1.0) with the most promising framework design for organizational audit knowledge (sigils, Scribe distillation, shared grimoire via git). Penman builds the automated pipeline that Grimoire's framework designs for.
- **[LISA](https://arxiv.org/html/2509.24698)** — Academic framework that learns from past Code4rena audit reports. Validates the learn-from-past-audits approach. Penman uses private organizational data instead of public contest data.

Two private experimental tools informed the design:
- **IBC Vulnerability Scanner** — 17 detection patterns from 50+ public IBC findings, built manually using the gather→summarize→pattern→aggregate process that Penman automates
- **Cosmos World Mapper** — 3-layer knowledge graph builder for Cosmos SDK chains (structural → execution → security topologies), built because Hound's graph construction is impractical on L1 codebases

## Example Output

See [examples/](examples/) for a sample detection module showing what Penman produces — a Grimoire-compatible check with numeric confidence and provenance.

## Research

- [Competition analysis](research/competition-analysis.md) — Grounded analysis of Hound, Grimoire, LISA, Codex Security, Claude Code Security, and 6+ additional tools
- [25-dimension comparison matrix](research/comparison-matrix.md) — Structured comparison across all five primary tools
