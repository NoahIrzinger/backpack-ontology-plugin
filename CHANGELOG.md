# Changelog

## 0.4.0 (2026-04-10)

Pairs with `backpack-ontology@0.5.0` and `backpack-viewer@0.5.0`.
Introduces **multiple backpacks** — users can register several graph
directories (personal, shared folders, project-specific, network
mounts, anywhere the filesystem presents a path) and switch between
them with a single command.

### Skills refreshed for multi-backpack
- **`backpack-guide` (v0.7.0)** opens with a "Multiple backpacks"
  section placed *before* the three-role rule. Teaches the agent:
  - Every `backpack_list` / `backpack_describe` response includes an
    `activeBackpack` field — always read it.
  - Name the backpack when reporting actions ("Added X to the **work**
    backpack's Y graph").
  - Suggest switching when the user's question implies a different
    backpack than the active one.
  - Never assume which backpack is active — check the `activeBackpack`
    field from the last list/describe.
  - Reference for the meta-tools with the **new simpler signatures**
    from ontology 0.5.0: `backpack_register <path>` (no name —
    derived from path tail), `backpack_switch <path-or-name>`,
    `backpack_active`, `backpack_registered`, `backpack_unregister <path-or-name>`.
- **`backpack-mine` (v0.2.0)** gains a first setup step that confirms
  the active backpack before mining starts. Surfaces the backpack
  name in the plan message (`Mining "X" into <backpack>/<graph>...`).
  Never silently mines into the wrong backpack — asks if ambiguous.

## 0.2.2 (2026-04-10)

### Fix
- **Added `.claude-plugin/marketplace.json`** — the plugin repo was missing
  the marketplace catalog file, so `/plugin marketplace add
  NoahIrzinger/backpack-ontology-plugin` failed with "marketplace file not
  found." The repo is now a valid Claude Code marketplace that lists itself
  as its single plugin.
- **Corrected install command in README.** The plugin's name (from
  `plugin.json`) is `backpack-ontology`, not `backpack-ontology-plugin`.
  The 0.2.1 README told users to run
  `/plugin install backpack-ontology-plugin@...` which would fail.
  Corrected to `/plugin install backpack-ontology@NoahIrzinger-backpack-ontology-plugin`.

## 0.2.1 (2026-04-10)

### Fix
- **README install instructions.** The previous README told users to run
  `/install-plugin <github-url>`, which is not a real Claude Code command —
  it fell through to "Unknown skill: install-plugin". Corrected to the actual
  marketplace-based install flow: `/plugin marketplace add` followed by
  `/plugin install`. Also documented the `--plugin-dir` path for local
  development against a cloned repo.

## 0.2.0 (2026-04-10)

Pairs with `backpack-ontology@0.3.0` and `backpack-viewer@0.3.0`. Existing graphs
from older versions are migrated automatically on first start — nothing to do.

### New skill: `backpack-mine`
- Autonomous mining loop driver. Point Claude at a topic, run an iteration loop
  that finds sources via WebSearch / WebFetch / Read, extracts relational facts,
  validates them through `backpack_import_nodes`, and stops on convergence or
  a node target.
- Configurable defaults: `iterations` (10), `targetNodes` (50),
  `convergenceThreshold` (2), `minConnections` (1), `sourceMode` (hybrid).
- Every mined node carries `source` and `minedAt` properties for traceability.
- Auto-snapshots before iteration 1 so the user can roll back the entire run
  with one call.
- Enforces the three-role rule on every extraction (skips procedural and
  briefing-shaped content).

### `backpack-guide` refreshed for 0.3.0
- `backpack_normalize` doc updated: now defaults to dry-run, opt-in to apply.
- New tools documented in Layer 5: `backpack_health`, `backpack_audit_roles`,
  `backpack_lock_status`.
- New section on branches, snapshots, rollback, and diff with concrete
  "when to snapshot" guidance.
- New section on collaboration and conflicts (optimistic concurrency + lock
  heartbeat).
- Cross-reference to `backpack-mine` for autonomous mining requests.
- Removed stale Hooks section — hooks auto-install was removed in `backpack-ontology@0.2.26`
  and a cleanup pass now runs automatically on MCP startup.
- Removed dead trigger phrases ("set up auto-capture", "enable backpack hooks")
  from skill description.
- Layer 2 / Layer 5 deduplication: `backpack_audit` now only listed under Layer 5.

### README updates
- Added "Mine a topic autonomously" usage examples.
- Fixed stale storage path: `~/.local/share/backpack/ontologies/` →
  `~/.local/share/backpack/graphs/<name>/branches/<branch>/{events.jsonl, snapshot.json}`.
- Fixed `backpack-init` description: it's now a cleanup command (was: installer).
- Updated "What's included" table to list both skills.

### Plugin metadata
- Version: 0.1.0 → 0.2.0
- Added `mining` keyword

## 0.1.0

Initial public release.
- Plugin manifest, MCP server registration, `backpack-guide` skill, README, PRIVACY.
