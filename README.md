# Backpack Ontology Plugin

Persistent knowledge graph memory for Claude. Store, search, and traverse structured data that persists across sessions.

## What it does

Backpack gives Claude structured, searchable memory organized as typed graphs — nodes (entities) connected by edges (relationships). There are no enforced schemas; Claude decides what structure fits the domain.

Examples:
- "Create an ontology about our codebase architecture"
- "Search the backpack for anything related to authentication"
- "Add the deployment pipeline to the infrastructure ontology"
- "Show me the graph"
- "Set up auto-capture so backpack learns as we work"

## Components

### MCP Server: backpack-ontology

Provides 16 tools organized in progressive discovery layers:

- **Discover**: list, create, describe, delete ontologies
- **Browse**: list nodes, node types, search
- **Inspect**: get full node data, graph traversal
- **Mutate**: add/update/remove nodes and edges, bulk import

### Skill: backpack-guide

Teaches Claude how to use the ontology tools effectively — progressive discovery pattern, naming conventions, common workflows, auto-capture setup, and graph visualization via backpack-viewer.

### Auto-Capture (Hooks)

Opt-in automation that builds knowledge graphs from conversations. Run `npx backpack-init` in your project to enable. A background agent reviews each conversation and captures meaningful knowledge — business relationships, technical decisions, domain concepts, processes — without needing to call tools manually.

### Visualization: backpack-viewer

The skill instructs Claude to launch a web-based graph visualizer (`npx backpack-viewer`) when the user asks to see their ontology. Force-directed layout with live reload — changes made via MCP tools appear automatically.

## Installation

Requires Node.js 18+.

### 1. Install the plugin

In Claude Code, run:

```
/install-plugin https://github.com/NoahIrzinger/backpack-ontology-plugin
```

This installs the skill and MCP server. Claude will now have access to 16 backpack tools and know how to use them effectively.

### 2. Enable auto-capture (optional)

To have backpack automatically build knowledge graphs from your conversations:

```bash
npx backpack-init
```

This writes hook configuration to `.claude/settings.json` in your project. A background agent will review conversations and capture meaningful knowledge without you needing to call tools manually.

### 3. Visualize your knowledge graph

After building an ontology, view it in the browser:

```bash
npx backpack-viewer
```

Opens a force-directed graph visualization at http://localhost:5173 with live reload — changes made via MCP tools appear automatically.

## Data Storage

Ontologies are stored as human-readable JSON at `~/.local/share/backpack/ontologies/`.

Override with environment variables:
- `XDG_DATA_HOME` — override data location
- `BACKPACK_DIR` — override both config and data directories

## Privacy & Telemetry

See the [Backpack Ontology Privacy Policy](https://github.com/noahirzinger/backpack-ontology/blob/main/PRIVACY.md). Anonymous usage telemetry is collected by default (tool counts, session duration — never content). Opt out with `DO_NOT_TRACK=1` or `BACKPACK_TELEMETRY_DISABLED=1`.

## License

Apache-2.0
