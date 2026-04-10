---
name: backpack-guide
description: >
  This skill should be used when the user asks to "create a learning graph",
  "store knowledge", "build a learning graph", "search the backpack",
  "remember this", "add to the graph", "visualize the graph",
  "show me the graph", "set up auto-capture", "enable backpack hooks",
  "sync my backpack", "sync to cloud", "upload to cloud", "move to cloud",
  or wants persistent structured memory across sessions. Also use when
  the user mentions "backpack", "ontology", "learning graph", "nodes and
  edges", "graph traversal", or "sync".
metadata:
  version: "0.4.0"
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
- `backpack_audit` — connectivity audit: orphans, weak nodes, sparse types

### Layer 3: Inspect

- `backpack_get_node` — full node with all properties and connected edge summaries
- `backpack_get_neighbors` — graph traversal from a node (max depth 3)

### Layer 4: Mutate

- `backpack_synthesize` — **preferred for any session adding more than ~3 nodes** — builds connected subgraphs in one call
- `backpack_import_nodes` — bulk-add nodes and edges atomically
- `backpack_connect` — bulk-add edges between existing nodes
- `backpack_add_node` — single-node addition (avoid in normal flows)
- `backpack_update_node` — merge new properties into an existing node
- `backpack_remove_node` — remove a node and cascade-delete its edges
- `backpack_add_edge` / `backpack_remove_edge` — single-edge ops

### Layer 5: Audit and consolidate

- `backpack_audit_roles` — **scan for content that violates the three-role rule** — flags nodes that look procedural (should be a skill) or briefing-like (should be in CLAUDE.md). Run periodically.
- `backpack_audit` — connectivity audit

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

1. `backpack_audit` → connectivity issues (orphans, weak nodes, sparse types)
2. `backpack_audit_roles` → three-role rule violations
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

## Hooks

Backpack installs a lightweight PostToolUse hook that confirms when the backpack has been updated. Hooks are installed automatically when the MCP server starts. Disable by removing them from `.claude/settings.json`.

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
