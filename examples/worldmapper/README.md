# Cosmos World Mapper — Security Topology Visualizations

Static analysis-generated security topology views of Cosmos SDK chains. **100% deterministic — zero LLM involvement.**

## How these files were generated

1. **Go analyzer** (`cosmos-chain-analyzer`) parses the chain's source code using `go/ast`, `golang.org/x/tools/go/packages`, and proto file parsing. No regex, no LLM. Every data point traces to a specific `file:line:column` in the source.

2. **Output:** A JSON file containing structural topology (modules, keepers, state stores, message routes), execution topology (ABCI hooks, ante handler chain, block execution order), and security-relevant metadata (keeper dependencies, store key prefixes, entry points).

3. **Visualizer** (`security-viz.py`) reads the JSON and renders a self-contained HTML file with interactive dependency graphs (vis.js) and security-focused module cards.

## What the visualization shows

- **Keeper dependency graph** — which modules can access which keepers (orange = custom modules, gray = SDK, purple = overrides)
- **ABCI execution order** — BeginBlock/EndBlock sequence (consensus-critical ordering)
- **Ante handler chain** — transaction validation pipeline
- **Custom module attack surface** — for each custom module:
  - Entry points (messages with field names and source locations)
  - State keys (store prefixes with byte values)
  - ABCI hooks (consensus-critical code paths)
  - Keeper dependencies (cross-module access)

## Files

- `babylon-security.html` — [Babylon chain](https://babylonlabs.io/) security topology ([view](https://imxm.github.io/penman/examples/worldmapper/babylon-security.html))
