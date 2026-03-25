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

Run:

```
/install-plugin https://github.com/NoahIrzinger/backpack-ontology-plugin
```

Restart Claude Code and you're ready.

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
| **MCP Server** | 16 tools for storing, searching, and traversing learning graphs |
| **Skill** | Teaches Claude best practices for organizing and querying your backpack |
| **Update Notifications** | Confirms when your backpack has been updated |
| **Visualization** | Claude launches a web-based graph explorer when you ask to see your data |

Auto-capture hooks install automatically. To disable, remove the backpack hooks from `.claude/settings.json`.

## Reference

### CLI commands

| Command | What it does |
|---|---|
| `npx backpack-viewer` | Open the graph visualizer (http://localhost:5173) |
| `npx -p backpack-ontology@latest backpack-sync` | Upload local learning graphs to Backpack App |
| `npx -p backpack-ontology@latest backpack-init` | Reinstall auto-capture hooks if removed |

### Data storage

Local learning graphs are stored as human-readable JSON at `~/.local/share/backpack/ontologies/`.

## Privacy & Telemetry

See the [Privacy Policy](https://github.com/noahirzinger/backpack-ontology/blob/main/PRIVACY.md). Anonymous usage telemetry is collected by default (tool counts, session duration, never content). Opt out with `DO_NOT_TRACK=1`.

## License

Apache-2.0
