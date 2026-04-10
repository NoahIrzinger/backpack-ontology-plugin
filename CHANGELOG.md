# Changelog

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
