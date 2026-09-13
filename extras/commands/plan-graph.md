Break down the project scope you describe into a **DAG-shaped agent workflow graph** — nodes, dependencies, what can run in parallel vs. what must run sequentially, merge/verify nodes — rendered as a LangGraph-style visual flowchart, with notes on which nodes can be dispatched to parallel subagents in the same round.

**General-purpose command, not tied to any specific project or domain** — this is a planning-stage thinking aid, usable for any kind of project (not just software).

---

## Step 0: Scope the request

If the description is specific enough, start breaking it down right away. 🚫 **Don't ask clarifying questions** — for anything ambiguous, pick the most reasonable scope yourself and note in your report: "I interpreted the scope as XXX — let me know if that's wrong."

---

## Step 1: Break into nodes (think it through before drawing)

1. List every subtask in the project.
2. Determine each subtask's input dependencies: a task that **only reads shared state and doesn't depend on another node's output → can run in parallel**; a task that needs another node's output first → mark the dependency direction.
3. Identify **merge (join) points**: nodes that must wait for multiple branches to finish before proceeding.
4. Decide whether to add **verify nodes**: does this project have an objective, checkable acceptance condition (a number, a rule, a checklist)?
   - Yes → add a verify node + a repair loop (a dashed line back to the failed node, labeled with the **specific fix needed**, not "redo everything").
   - No → don't force one in. Most projects are just a linear path to the finish line — don't fabricate a fake verification step just to have something to draw.
5. 🚨 **Be honest**: if the project has almost no nodes that can run in parallel, just tell the requester "this one's faster done linearly, a graph won't add much" — don't draw a graph for its own sake. This also applies to tasks that need holistic judgment, creative continuity, or would lose coherence if split apart (e.g., writing an SOP, life planning) — it's fine to say a breakdown isn't recommended.

---

## Step 2: Draw the graph (Artifact + Mermaid)

Publish a flowchart via `Artifact` (`graph LR` or `graph TD`, pick horizontal or vertical based on node count — whichever lays out better). This graph is meant to help the requester think, so it's worth making it look good — **load the `artifact-design` skill before publishing** to get the design sense right; don't hand over a bare-bones flowchart.

Drawing conventions:
- Label each node with one line describing what it actually does.
- Color-code: **green = can run in parallel**, **orange/yellow = sequential dependency (waiting on something)**, **red = verify node or a point that needs a decision from the requester**.
- Merge/verify nodes get a double border or bold outline to distinguish them from regular task nodes.
- If there's a repair loop, draw it as a dashed line back, labeled with the specific fix amount (e.g. "over budget by $500", "-100 per night in the same zone" — concrete, not just "retry").

---

## Step 3: Propose the dispatch plan

Once the graph is done, spell out:

- **How many subagents can be dispatched in parallel this round** (list the corresponding nodes) — this is the map for dispatching work.
- **Which nodes must be done sequentially**, and why (what output they're waiting on).
- If the requester then says "go," dispatch directly against this map using the `Agent` tool — no need to re-plan from scratch.

---

## Pitfalls to avoid

- A dispatch map is not auto-execution. Finishing the graph doesn't start any subagents — wait for the requester to say "go" or "do it this way" before acting.
- If the scope changes mid-project, redraw the graph — don't force the old graph to still apply.
- This graph is a planning aid, not a document the requester is expected to maintain — unless they ask, there's no need to save it anywhere permanent.
