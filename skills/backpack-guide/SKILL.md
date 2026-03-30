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
  version: "0.3.0"
---

# Backpack Guide

Backpack is the user's persistent knowledge base — a single backpack they carry across every conversation. Inside it are **learning graphs**, each one about a different topic (clients, processes, architecture, etc.).

## Core Concept

There is one backpack. Inside it are learning graphs. Each learning graph is a typed graph: **nodes** (things) connected by **edges** (relationships). There are no enforced schemas — decide what structure fits the domain. Nodes have a freeform `type` and arbitrary `properties`. Edges have a `type` connecting a source to a target.

## Progressive Discovery Pattern

Always follow this layered approach to keep the context window lean. Start broad, drill down only as needed.

### Layer 1: Discover

Start here. Understand what learning graphs exist before doing anything else.

- `backpack_list` — list all learning graphs with names, descriptions, and summary counts
- `backpack_create` — create a new empty learning graph
- `backpack_describe` — inspect a graph's structure: node types, edge types, counts

### Layer 2: Browse

Once you know which learning graph to work with, explore its contents.

- `backpack_node_types` — list distinct node types with counts
- `backpack_list_nodes` — paginated node summaries, optionally filtered by type
- `backpack_search` — case-insensitive text search across all node properties
- `backpack_audit` — analyze graph quality: orphans, weak nodes, sparse types, and improvement suggestions

### Layer 3: Inspect

Pull full details only for specific nodes of interest.

- `backpack_get_node` — full node with all properties and connected edge summaries
- `backpack_get_neighbors` — BFS graph traversal from a node (max depth 3)

### Layer 4: Mutate

Create or modify data only after understanding the existing structure.

- `backpack_add_node` — add a node with a freeform type and properties
- `backpack_update_node` — merge new properties into an existing node
- `backpack_remove_node` — remove a node and cascade-delete its edges
- `backpack_add_edge` — create a typed relationship between two nodes
- `backpack_remove_edge` — remove a relationship
- `backpack_import_nodes` — bulk-add nodes **and edges** in a single call (preferred for building connected subgraphs)
- `backpack_connect` — bulk-add edges between existing nodes (preferred for improving connectivity)
- `backpack_expand` — expand a node with related entities in a direction (loads context, you add the knowledge)
- `backpack_explain_path` — find the shortest path between two nodes and explain the semantic connection
- `backpack_enrich` — deepen a node with additional properties and connections
- `backpack_synthesize` — build a graph from multiple sources in one workflow

## Best Practices

1. **Always call `backpack_list` first** to see what already exists. Never create a duplicate learning graph.
2. **Use `backpack_describe` before mutating** to understand the existing schema — match existing node types and edge types rather than inventing new ones that overlap.
3. **Use `backpack_search` before adding nodes** to avoid duplicates.
4. **Keep node types consistent** — use singular PascalCase (e.g., `Person`, `Module`, `Recipe`).
5. **Keep edge types consistent** — use SCREAMING_SNAKE_CASE verbs (e.g., `WORKS_ON`, `DEPENDS_ON`, `CONTAINS`).
6. **Use meaningful properties** — include enough context in node properties that the node is useful on its own without traversing edges.
7. **Always include edges when importing nodes** — use `backpack_import_nodes` with the `edges` array to create nodes and relationships in one call. Reference new nodes by their array index (0, 1, 2...) and existing nodes by their ID string. Never import nodes without connecting them.

## Common Patterns

### Building a learning graph from scratch
1. `backpack_create` with a descriptive name and description
2. Use `backpack_import_nodes` with both `nodes` and `edges` to add connected subgraphs in a single call. Reference nodes within the import by index (0, 1, 2...) and existing anchor nodes by their ID.
3. Use `backpack_describe` to verify the structure looks right

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

Indices 0, 1, 2 refer to the Recipe, garlic, and olive oil nodes in order.
To connect a new node to an existing node, use its ID string: `"target": "n_abc123def456"`.

### Answering questions from a learning graph
1. `backpack_list` to find the relevant graph
2. `backpack_search` with keywords from the user's question
3. `backpack_get_node` on the most relevant result
4. `backpack_get_neighbors` if the answer requires traversing relationships

### Updating existing knowledge
1. `backpack_search` to find the node to update
2. `backpack_get_node` to see current properties
3. `backpack_update_node` to merge in new properties (existing properties are preserved)

### Improving an existing learning graph
1. `backpack_audit` to get a prioritized improvement report — orphans, weak nodes, sparse types, disconnected type pairs, and actionable suggestions
2. Fix orphans first — use `backpack_connect` to add edges for nodes with zero connections
3. Address weak nodes — nodes with far fewer connections than their type average
4. Check disconnected type pairs — if two types logically relate but have no edges between them, add relationships
5. Use `backpack_describe` to verify improvements (check `stats` for updated metrics)

### Curiosity flow: exploring and expanding
When the user is exploring a graph and wants to go deeper:
1. `backpack_expand` on a node of interest — adds related entities in the direction the user cares about
2. `backpack_explain_path` when the user asks "why are these connected?" — finds the shortest path and you explain the semantic meaning
3. `backpack_enrich` to deepen a specific node with more properties and connections
4. After expanding, suggest: "Open the viewer to see the new connections: [link]"

### Multi-source synthesis
When the user wants to combine knowledge from multiple sources (databases, code, APIs, docs, other projects):
1. Gather data from each source (read files, query MCP tools, etc.)
2. `backpack_synthesize` with all sources and a focus area
3. After building, `backpack_audit` to check for gaps
4. Suggest: "Open the viewer to explore connections: [link]"
5. Ask: "Which areas should I expand or investigate further?"

## Hooks

Backpack installs a lightweight PostToolUse hook that confirms when the backpack has been updated. Hooks are installed automatically when the MCP server starts.

### Disabling hooks
To disable, remove the backpack hooks from `.claude/settings.json`. Running `npx -p backpack-ontology@latest backpack-init` will reinstall them.

## Visualization

Backpack includes a web-based graph visualizer with force-directed layout and live reload.

### Launching the viewer

When the user asks to "visualize", "show the graph", "see the learning graph", or "open the viewer":

1. Run `npx backpack-viewer` via Bash to start the viewer server on localhost
2. The viewer starts on `http://localhost:5173` by default
3. In Cowork: open the URL in the browser so the user can see the graph immediately
4. In Claude Code: inform the user the viewer is running and they can open `http://localhost:5173` in their browser

The viewer reads data directly from the same storage location (`~/.local/share/backpack/ontologies/`) and supports live reload — changes made via MCP tools appear in the visualization automatically.

### When to suggest visualization
- After building or significantly modifying a learning graph
- After auto-capture updates a graph
- When the user is exploring a complex graph with many relationships
- When the user asks about the overall structure or shape of their data

## Syncing to Backpack App

**IMPORTANT**: When the user says anything about syncing their backpack — "sync my backpack", "sync to cloud", "upload to cloud", "move to cloud", "sync to remote", "push to cloud" — IMMEDIATELY run the sync command. Do NOT ask clarifying questions. Do NOT try to use MCP tools. Do NOT check what's in the backpack first. Just run the command:

```bash
npx -p backpack-ontology@latest backpack-sync
```

The sync tool handles everything automatically:
- Reads local learning graphs from disk
- Authenticates with Backpack App (reuses cached credentials or opens browser)
- Uploads all learning graphs to the cloud
- Reports what was created/updated

After syncing, tell the user their learning graphs are now in Backpack App and they can switch to cloud mode by updating their MCP config to use `backpack-app` instead of `backpack`.
