# Backpack Ontology Plugin

Persistent knowledge graph memory for Claude. Store, search, and traverse structured data that persists across sessions.

## What it does

Backpack gives Claude structured, searchable memory organized as typed graphs — nodes (entities) connected by edges (relationships). There are no enforced schemas; Claude decides what structure fits the domain.

Examples:
- "Create an ontology about our codebase architecture"
- "Search the backpack for anything related to authentication"
- "Add the deployment pipeline to the infrastructure ontology"
- "Show me the graph"

## Components

### MCP Server: backpack-ontology

Provides 15 tools organized in progressive discovery layers:

- **Discover**: list, create, describe, delete ontologies
- **Browse**: list nodes, node types, search
- **Inspect**: get full node data, graph traversal
- **Mutate**: add/update/remove nodes and edges, bulk import

### Skill: backpack-guide

Teaches Claude how to use the ontology tools effectively — progressive discovery pattern, naming conventions, common workflows, and graph visualization via backpack-viewer.

### Visualization: backpack-viewer

The skill instructs Claude to launch a web-based graph visualizer (`npx backpack-viewer`) when the user asks to see their ontology. Force-directed layout with live reload — changes made via MCP tools appear automatically.

## Setup

Requires Node.js 18+. The MCP server and viewer both run via npx — no additional configuration needed.

## Data Storage

Ontologies are stored as human-readable JSON at `~/.local/share/backpack/ontologies/`.

Override with environment variables:
- `XDG_DATA_HOME` — override data location
- `BACKPACK_DIR` — override both config and data directories

## Privacy & Telemetry

See the [Backpack Ontology Privacy Policy](https://github.com/noahirzinger/backpack-ontology/blob/main/PRIVACY.md). Anonymous usage telemetry is collected by default (tool counts, session duration — never content). Opt out with `DO_NOT_TRACK=1` or `BACKPACK_TELEMETRY_DISABLED=1`.

## License

Apache-2.0
