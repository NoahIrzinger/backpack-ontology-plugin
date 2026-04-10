---
name: backpack-guide
description: >
  This skill should be used when the user asks to "create a learning graph",
  "store knowledge", "build a learning graph", "search the backpack",
  "remember this", "add to the graph", "visualize the graph",
  "show me the graph", "audit my backpack", "normalize my graph",
  "check graph health", "snapshot this", "branch this graph",
  "sync my backpack", "sync to cloud", "upload to cloud", "move to cloud",
  or wants persistent structured memory across sessions. Also use when
  the user mentions "backpack", "ontology", "learning graph", "nodes and
  edges", "graph traversal", or "sync". For autonomous mining of a topic
  ("mine X", "research X into backpack", "grow this graph from sources"),
  use the separate `backpack-mine` skill instead.
metadata:
  version: "0.5.0"
---

# Backpack Guide

Backpack is the user's persistent knowledge graph — a typed graph store the LLM can query without loading the whole thing into context. Inside the backpack are **learning graphs**, each one about a different topic.

## The Three-Role Rule (read this before doing anything)

There are three places an LLM can read knowledge from. They do different jobs and they should never overlap.

| Mechanism | What it holds | When it's read | Who writes |
|---|---|---|---|
| **CLAUDE.md** | The LLM's *briefing* — facts about the project that are true today and need to be in every session | Auto-loaded at session start | Human (occasionally LLM-suggested) |
| **Skills** | The LLM's *playbook* — how to perform a task as a sequence of steps | Loaded on demand when triggered | Human (occasionally LLM-generated) |
| **Backpack learning graphs** | The LLM's *discovered relational knowledge* — typed entities and the relationships between them, accumulated across sessions | Queried via MCP when needed | LLM (via conversation), human-curated |

**These are mutually exclusive.** Before adding anything to a learning graph, ask: does this belong in CLAUDE.md or a skill instead?

### What belongs in a learning graph

- "The auth service depends on the user service" → typed edge
- "GPT-4 was developed by OpenAI in 2023" → typed nodes connected by typed edges
- "Bob owns the billing module; the billing module integrates with Stripe" → ownership and integration relationships
- "Mitochondria contain DNA; mitochondria produce ATP; ATP powers muscle contraction" → biology entities and reactions
- Anything that is **a discrete fact connecting two entities by a named relationship**

### What does NOT belong in a learning graph (use CLAUDE.md instead)

- "This project uses Go on stdlib net/http" → environmental briefing → CLAUDE.md
- "The user prefers terse responses with no preamble" → user preference → CLAUDE.md
- "Production runs on Kubernetes" → infrastructure fact loaded every session → CLAUDE.md
- "Always use parameterized SQL queries" → coding convention → CLAUDE.md
- Anything that is **a fact you'd want to inject into every conversation regardless of topic**

### What does NOT belong in a learning graph (use a skill instead)

- "To deploy: run `make deploy`, then check the logs" → procedural → skill
- "When the user asks to commit, run `git status` and propose a message" → workflow → skill
- "Step 1: open file. Step 2: change line. Step 3: run tests." → sequential steps → skill
- Anything that is **a sequence of actions performed when triggered**

### When in doubt

Use this rule of thumb based on access pattern:

- **Loaded into every session?** → CLAUDE.md
- **Loaded when a task triggers it?** → skill
- **Looked up on demand to answer a question?** → learning graph

Only the third case justifies a learning graph. The first two have better homes.

## What learning graphs are good for

Learning graphs store **discrete, atomic, composable facts** with **explicit typed relationships**, queryable by structure rather than by full-text search. The killer property: an LLM can answer questions about a 50,000-node graph by loading the 500 most relevant tokens, not the whole graph.

The right shape for a graph is:
- Many entities (nodes) of similar types
- Explicit named relationships (edges) between them
- Queryable structure: "what's connected to X within 2 hops by edge type Y"
- Mergeable from multiple sources without rewriting prose
- Updatable per-fact without rewriting paragraphs

The wrong shape is a single long paragraph of prose, a workflow with steps, or a list of conventions. Those have other homes.

## Progressive Discovery Pattern

When working with a learning graph, never load the whole thing. Start broad, drill down only as needed.

### Layer 1: Discover

- `backpack_list` — list all learning graphs
- `backpack_create` — create a new empty learning graph
- `backpack_describe` — inspect structure: types and counts (no actual data)

### Layer 2: Browse

- `backpack_node_types` — distinct node types with counts
- `backpack_list_nodes` — paginated node summaries, optionally filtered by type
- `backpack_search` — text search across all node properties

### Layer 3: Inspect

- `backpack_get_node` — full node with all properties and connected edge summaries
- `backpack_get_neighbors` — graph traversal from a node (max depth 3)

### Layer 4: Mutate

- `backpack_import_nodes` — **preferred for any session adding more than ~3 nodes**. Validates the batch automatically: errors block the commit, warnings (type drift, duplicates, three-role rule violations) are surfaced in the response. Pass `dryRun: true` to review the validation without committing.
- `backpack_synthesize` — combine multiple text sources (databases, code, docs) into structured nodes
- `backpack_connect` — bulk-add edges between existing nodes
- `backpack_add_node` — single-node addition (**avoid** in normal flows; prefer import_nodes)
- `backpack_update_node` — merge new properties into an existing node
- `backpack_remove_node` — remove a node and cascade-delete its edges
- `backpack_add_edge` / `backpack_remove_edge` — single-edge ops

### How import_nodes validates your batch

The validator runs automatically on every `backpack_import_nodes` call. It catches:

- **Errors (block commit):** broken edges, self-loops, invalid property shapes (nested objects, etc.)
- **Warnings (committed anyway, but reported):**
  - **Type drift** — you used `service` but the graph already has `Service`
  - **Duplicate nodes** — your new node has the same type+label as an existing one
  - **Three-role violations** — your node looks procedural (should be a skill) or briefing-like (should be in CLAUDE.md)

The agent workflow for safe bulk adds:

1. (Optional but recommended for big batches): call `backpack_import_nodes` with `dryRun: true` to review warnings
2. Fix any flagged issues (rename to canonical types, deduplicate, move procedural content to skills)
3. Call again without `dryRun` (or `dryRun: false`) to commit

Errors always block the commit regardless of `dryRun`. Don't wrap dry-run in a loop hoping for a different result — fix the underlying issue.

### Layer 5: Audit and consolidate

- `backpack_health` — **one-call complete picture** — runs all the audits below in a single call and returns a unified report (tokens, connectivity, role violations, type drift plan, active editor). Start here when you want to know how a graph is doing.
- `backpack_audit_roles` — **scan for content that violates the three-role rule** — flags nodes that look procedural (should be a skill) or briefing-like (should be in CLAUDE.md). Run periodically.
- `backpack_normalize` — **consolidate type drift** — collapses near-miss type variants ("service" / "Service" / "SERVICE") onto the dominant one, both for nodes and edges. **Defaults to dry-run** for safety: review the plan, then call again with `dryRun: false` to apply. Type renames preserve node IDs and all edges, so this is safe to run on a connected graph.
- `backpack_audit` — connectivity audit
- `backpack_lock_status` — read the current edit heartbeat for a graph (who else is actively editing, if anyone)

## Best Practices

1. **Always check the three-role rule before adding to a graph.** If the content is procedural or briefing-shaped, redirect to a skill or CLAUDE.md.
2. **Always call `backpack_list` first** to see what already exists. Never create a duplicate learning graph.
3. **Use `backpack_describe` before mutating** to understand the existing types — match existing node types and edge types rather than inventing overlapping new ones.
4. **Use `backpack_synthesize` or `backpack_import_nodes` for new content**, not individual `add_node` calls. Per-call overhead adds up fast.
5. **Use `backpack_search` before adding nodes** to avoid duplicates.
6. **Keep node types consistent** — singular PascalCase (`Person`, `Module`, `Recipe`).
7. **Keep edge types consistent** — SCREAMING_SNAKE_CASE verbs (`WORKS_ON`, `DEPENDS_ON`, `CONTAINS`).
8. **Always include edges when importing nodes** — never import unconnected nodes. Reference new nodes by array index (0, 1, 2...) and existing nodes by ID string.
9. **Run `backpack_audit_roles` periodically** on graphs that have grown — drift toward procedural or briefing content is the primary failure mode of freeform graphs.

## Common Patterns

### Building a learning graph from scratch

1. `backpack_list` to confirm it doesn't exist
2. `backpack_create` with a descriptive name and description
3. `backpack_synthesize` or `backpack_import_nodes` to add connected subgraphs in one call
4. `backpack_describe` to verify the structure looks right
5. `backpack_audit_roles` to check nothing drifted into procedural or briefing shapes

### Example: importing a connected subgraph

```json
{
  "ontology": "cooking",
  "nodes": [
    { "type": "Recipe", "properties": { "name": "Aglio e Olio" } },
    { "type": "Ingredient", "properties": { "name": "garlic" } },
    { "type": "Ingredient", "properties": { "name": "olive oil" } }
  ],
  "edges": [
    { "type": "USES", "source": 0, "target": 1 },
    { "type": "USES", "source": 0, "target": 2 },
    { "type": "PAIRS_WITH", "source": 1, "target": 2 }
  ]
}
```

Indices 0, 1, 2 refer to new nodes in order. Use existing node IDs as strings to anchor: `"target": "n_abc123def456"`.

### Answering questions from a learning graph

1. `backpack_list` → find the relevant graph
2. `backpack_search` with keywords from the question
3. `backpack_get_node` on the most relevant result
4. `backpack_get_neighbors` if the answer requires traversing relationships

### Updating existing knowledge

1. `backpack_search` → find the node
2. `backpack_get_node` → see current properties
3. `backpack_update_node` → merge in new properties (existing properties preserved)

### Improving an existing learning graph

1. `backpack_health` → unified report (faster than running each tool separately)
2. `backpack_normalize` with `dryRun: true` → preview type drift consolidation, then apply if the plan looks right
3. `backpack_connect` → fix orphans by adding edges
4. Move flagged procedural nodes into a skill, briefing nodes into CLAUDE.md, then `backpack_remove_node` from the graph

### Curiosity flow: exploring and expanding

1. `backpack_expand` on a node of interest
2. `backpack_explain_path` when the user asks "why are these connected?"
3. `backpack_enrich` to deepen a specific node
4. After expanding, suggest opening the viewer

### Multi-source synthesis

1. Gather data from each source (read files, query other tools)
2. `backpack_synthesize` with all sources and a focus area
3. `backpack_audit_roles` to check the synthesized content matches the rule
4. `backpack_audit` for connectivity gaps
5. Suggest opening the viewer

## Collaboration and conflicts

When a learning graph is shared (e.g. via OneDrive, Dropbox, or a network filesystem), multiple agents may try to write at the same time. Backpack handles this with optimistic concurrency:

- Every read records the current version of the graph.
- Every write must match that version. If someone else wrote in between, the second writer gets a `ConcurrencyError` and **no partial state is committed**.
- On conflict, retry: the local cache is invalidated automatically, so the next read pulls fresh state. Re-apply your change against the new state and try again.

Backpack also writes a lightweight heartbeat (`graphs/<name>/.lock`) on every successful write, with the author and timestamp. Treat this as an awareness signal — if you're about to make a big change and the heartbeat shows another author was active in the last few minutes, consider coordinating before clobbering their work.

## Versioning: branches, snapshots, rollback

Every learning graph has full version history. Use it as a safety net before risky operations.

- `backpack_snapshot <graph> "<label>"` — capture a labeled snapshot of the current state. Returns a version number. Cheap.
- `backpack_versions <graph>` — list all snapshots with labels and versions, newest first.
- `backpack_rollback <graph> <version>` — revert the graph to a previous snapshot. Destructive but safe (the snapshot is preserved as a version you can roll back to again).
- `backpack_diff <graph> <version>` — compare current state with a snapshot.
- `backpack_branch_create <graph> <branch>` / `backpack_branch_switch` / `backpack_branch_list` / `backpack_branch_delete` — fork a graph into a named branch for isolated experimentation. Branches are independent event logs.

**When to snapshot:**

- Before any bulk import or normalization run
- Before deleting nodes
- Before running an autonomous mining loop (the `backpack-mine` skill does this automatically)
- Anytime the user says "save where this is" or "checkpoint this"

## Autonomous mining

For "mine a topic" / "build a learning graph from sources" / "research X into the backpack" requests, hand off to the `backpack-mine` skill. It runs an iteration loop that finds sources, extracts entities and relationships, validates them through `backpack_import_nodes`, and stops on convergence or a node target. Don't try to replicate that loop here — invoke the dedicated skill.

## Visualization

Backpack includes a web-based graph visualizer with force-directed layout and live reload.

When the user asks to "visualize", "show the graph", "see the learning graph", or "open the viewer":

1. Run `npx backpack-viewer` via Bash to start the viewer server
2. The viewer starts on `http://localhost:5173` by default
3. Open the URL or tell the user to open it

The viewer reads directly from the same storage and supports live reload — changes via MCP appear automatically.

## Syncing to Backpack App

When the user says anything about syncing — "sync my backpack", "sync to cloud", "upload to cloud", "push to cloud" — IMMEDIATELY run:

```bash
npx -p backpack-ontology@latest backpack-sync
```

Do NOT ask clarifying questions. Do NOT use MCP tools first. Just run the command. The sync tool handles auth, upload, and reporting automatically.
