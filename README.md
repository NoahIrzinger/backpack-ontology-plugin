# Backpack Plugin

**Carry your knowledge forward.** Backpack lets Claude remember what matters: your clients, your processes, your decisions. Knowledge that travels with you.

## What it does

Backpack is a persistent knowledge base that Claude carries across conversations. Inside it are learning graphs, each about a different topic. There are no enforced schemas. Claude decides what structure fits the domain.

## Install

### Claude Cowork (web)

Tell Claude:

> "Install the backpack plugin from github.com/NoahIrzinger/backpack-ontology-plugin"

Claude will set it up for you. After that, just start talking.

### Claude Code (terminal)

Run these two commands in Claude Code:

```
/plugin marketplace add NoahIrzinger/backpack-ontology-plugin
/plugin install backpack-ontology@NoahIrzinger-backpack-ontology-plugin
```

The first command registers this repo as a marketplace. The second installs the plugin from it. Restart Claude Code (or run `/reload-plugins`) and you're ready.

For local development against a cloned repo, start Claude Code with:

```
claude --plugin-dir ./backpack-ontology-plugin
```

### Connect to the cloud (free)

Sign up for a free account at [app.backpackontology.com](https://app.backpackontology.com) to sync across devices and share with your team:

```
claude mcp add backpack-app -s user --transport sse https://app.backpackontology.com/mcp/sse
```

## What to say to Claude

No commands to learn. Just talk naturally.

### Remember something

> "Remember that Acme Corp is on the Enterprise tier, main contact is Sarah Chen"

> "Start a learning graph for our hiring process"

> "Add the new vendor agreement to backpack"

### Mine a topic autonomously

> "Mine the gut microbiome into a learning graph"

> "Build a learning graph about Spanish wines, 30 nodes max"

> "Research the history of postgres into the backpack"

Claude will run an iteration loop: find sources, extract entities and relationships,
validate against the existing graph, and commit. It stops when it hits the iteration cap,
the node target, or when it stops finding new entities. Every node carries the source it
came from, so the graph stays traceable.

### Find something

> "What's in my backpack about Acme Corp?"

> "Search backpack for anything related to compliance"

### See the big picture

> "Show me my learning graph"

> "What's in my backpack?"

Claude will open a graph visualizer so you can explore your knowledge visually.

### Move to the cloud

> "Sync my backpack to the cloud"

Sign up for a free account at [app.backpackontology.com](https://app.backpackontology.com) to access your knowledge from any device and share with your team.

## What's included

| Component | What it does |
|---|---|
| **MCP Server** | Tools for storing, searching, traversing, auditing, and normalizing learning graphs. Auto-migrates pre-0.3.0 graphs on first start. |
| **`backpack-guide` skill** | Teaches Claude best practices for organizing and querying your backpack |
| **`backpack-mine` skill** | Autonomous mining loop — point Claude at a topic and grow a graph from web sources |
| **Visualization** | Claude launches a web-based graph explorer when you ask to see your data |

## Reference

### CLI commands

| Command | What it does |
|---|---|
| `npx backpack-viewer` | Open the graph visualizer (http://localhost:5173) |
| `npx -p backpack-ontology@latest backpack-sync` | Upload local learning graphs to Backpack App |
| `npx -p backpack-ontology@latest backpack-init` | Remove any leftover Backpack hooks from `.claude/settings.json` (the MCP server also runs this cleanup automatically on startup) |

### Data storage

Local learning graphs are stored at `~/.local/share/backpack/graphs/<name>/`. Each graph has an append-only event log (`branches/<branch>/events.jsonl`) and a materialized snapshot cache (`branches/<branch>/snapshot.json`). Graphs from older versions are migrated automatically on first start.

## Privacy & Telemetry

See the [Privacy Policy](https://github.com/noahirzinger/backpack-ontology/blob/main/PRIVACY.md). Anonymous usage telemetry is collected by default (tool counts, session duration, never content). Opt out with `DO_NOT_TRACK=1`.

## License

Apache-2.0
