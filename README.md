# Backpack Plugin

**Give your AI a memory it can actually use.** Backpack lets Claude remember what matters — your clients, your processes, your decisions — across every conversation.

## What it does

Backpack is a single persistent knowledge base that Claude carries across conversations. Inside it are ontologies — each one a knowledge graph about a different topic. There are no enforced schemas; Claude decides what structure fits the domain.

## Install

### Claude Cowork (web)

Tell Claude:

> "Install the backpack plugin from github.com/NoahIrzinger/backpack-ontology-plugin"

Claude will set it up for you. After that, just start talking — tell Claude what to remember and it handles the rest.

### Claude Code (terminal)

Run:

```
/install-plugin https://github.com/NoahIrzinger/backpack-ontology-plugin
```

Restart Claude Code and you're ready.

## What to say to Claude

Just talk to Claude naturally. No commands to learn.

### Remember something

> "Remember that Acme Corp is on the Enterprise tier, main contact is Sarah Chen"

> "Start an ontology for our hiring process"

> "Add the new vendor agreement to backpack"

### Find something

> "What's in my backpack about Acme Corp?"

> "Search backpack for anything related to compliance"

### See the big picture

> "Show me my knowledge graph"

> "What's in my backpack?"

Claude will open a graph visualizer so you can explore your knowledge visually.

### Move to the cloud

> "Sync my backpack to the cloud"

Sign up for a free account at [app.backpackontology.com](https://app.backpackontology.com) to access your knowledge from any device and share with your team.

## What's included

| Component | What it does |
|---|---|
| **MCP Server** | 16 tools for storing, searching, and traversing knowledge graphs |
| **Skill** | Teaches Claude best practices for organizing and querying your backpack |
| **Auto-Capture** | Background agent that automatically saves meaningful knowledge from conversations |
| **Visualization** | Claude launches a web-based graph explorer when you ask to see your data |

Auto-capture hooks are installed automatically. To disable, remove the backpack hooks from `.claude/settings.json`.

## Reference

### CLI commands

| Command | What it does |
|---|---|
| `npx backpack-viewer` | Open the graph visualizer (http://localhost:5173) |
| `npx backpack-sync` | Upload local ontologies to Backpack App |
| `npx backpack-init` | Reinstall auto-capture hooks if removed |

### Data storage

Local ontologies are stored as human-readable JSON at `~/.local/share/backpack/ontologies/`.

## Privacy & Telemetry

See the [Privacy Policy](https://github.com/noahirzinger/backpack-ontology/blob/main/PRIVACY.md). Anonymous usage telemetry is collected by default (tool counts, session duration — never content). Opt out with `DO_NOT_TRACK=1`.

## License

Apache-2.0
