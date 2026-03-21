---
name: backpack-guide
description: >
  This skill should be used when the user asks to "create an ontology",
  "store knowledge", "build a knowledge graph", "search the backpack",
  "remember this", "add to the ontology", "visualize the graph",
  "show me the ontology", or wants persistent structured memory across
  sessions. Also use when the user mentions "backpack", "ontology",
  "nodes and edges", or "graph traversal".
metadata:
  version: "0.1.0"
---

# Backpack Ontology Guide

Backpack provides persistent knowledge graph storage via MCP tools. Use this guide to interact with ontologies effectively.

## Core Concept

Ontologies are typed graphs: **nodes** (entities) connected by **edges** (relationships). There are no enforced schemas — decide what structure fits the domain. Nodes have a freeform `type` and arbitrary `properties`. Edges have a `type` connecting a source node to a target node.

## Progressive Discovery Pattern

Always follow this layered approach to keep the context window lean. Start broad, drill down only as needed.

### Layer 1: Discover

Start here. Understand what ontologies exist before doing anything else.

- `backpack_list` — list all ontologies with names, descriptions, and summary counts
- `backpack_create` — create a new empty ontology
- `backpack_describe` — inspect an ontology's structure: node types, edge types, counts

### Layer 2: Browse

Once you know which ontology to work with, explore its contents.

- `backpack_node_types` — list distinct node types with counts
- `backpack_list_nodes` — paginated node summaries, optionally filtered by type
- `backpack_search` — case-insensitive text search across all node properties

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
- `backpack_import_nodes` — bulk-add multiple nodes in a single operation

## Best Practices

1. **Always call `backpack_list` first** to see what already exists. Never create a duplicate ontology.
2. **Use `backpack_describe` before mutating** to understand the existing schema — match existing node types and edge types rather than inventing new ones that overlap.
3. **Use `backpack_search` before adding nodes** to avoid duplicates.
4. **Keep node types consistent** — use singular PascalCase (e.g., `Person`, `Module`, `Recipe`).
5. **Keep edge types consistent** — use SCREAMING_SNAKE_CASE verbs (e.g., `WORKS_ON`, `DEPENDS_ON`, `CONTAINS`).
6. **Use meaningful properties** — include enough context in node properties that the node is useful on its own without traversing edges.
7. **Prefer `backpack_import_nodes` for bulk operations** — when adding 3+ nodes, batch them in a single call.

## Common Patterns

### Building an ontology from scratch
1. `backpack_create` with a descriptive name and description
2. Add foundational nodes first (the "anchors" other nodes relate to)
3. Add detail nodes and connect them with edges
4. Use `backpack_describe` to verify the structure looks right

### Answering questions from an ontology
1. `backpack_list` to find the relevant ontology
2. `backpack_search` with keywords from the user's question
3. `backpack_get_node` on the most relevant result
4. `backpack_get_neighbors` if the answer requires traversing relationships

### Updating existing knowledge
1. `backpack_search` to find the node to update
2. `backpack_get_node` to see current properties
3. `backpack_update_node` to merge in new properties (existing properties are preserved)

## Visualization

Backpack includes a web-based graph visualizer with force-directed layout and live reload.

### Launching the viewer

When the user asks to "visualize", "show the graph", "see the ontology", or "open the viewer":

1. Run `npx backpack-viewer` via Bash to start the viewer server on localhost
2. The viewer starts on `http://localhost:5173` by default
3. In Cowork: open the URL in the browser so the user can see the graph immediately
4. In Claude Code: inform the user the viewer is running and they can open `http://localhost:5173` in their browser

The viewer reads ontology data directly from the same storage location (`~/.local/share/backpack/ontologies/`) and supports live reload — changes made via MCP tools appear in the visualization automatically.

### When to suggest visualization
- After building or significantly modifying an ontology
- When the user is exploring a complex graph with many relationships
- When the user asks about the overall structure or shape of their data
