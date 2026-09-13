# Extras

Standalone Claude Code commands that have nothing to do with the memory
architecture in `template/`. They live here, separately, so the main pitch of
this repo — the four-layer memory system — stays a single idea. Install only
the ones you want:

```bash
cp extras/commands/plan-graph.md ~/.claude/commands/
```

| Command | What it does |
|---|---|
| [`/plan-graph`](commands/plan-graph.md) | Breaks a described project scope into a DAG-shaped agent workflow graph — nodes, dependencies, parallel vs. sequential, merge/verify points — rendered as a Mermaid flowchart via `Artifact`, with a dispatch plan for which nodes can run as parallel subagents. |
