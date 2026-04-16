---
name: backpack-distill
description: >
  This skill should be used when the user asks to "distill a graph",
  "check for signals", "what should I pay attention to", "what matters
  right now", "run signal detection", "find issues in my graph", or
  wants to surface the most important actionable insights from their
  knowledge. Also use when the user mentions "distill", "signals",
  "signal detection", or "what's noteworthy".
metadata:
  version: "0.1.0"
---

# Backpack Distill

Extract the essential signals from your knowledge. The final step in the knowledge
pipeline: `backpack-mine` extracts raw ore, `backpack-refine` processes it into
documents, `backpack-distill` concentrates it down to the purest actionable insights.

Distilling runs structural detectors across all graphs and KB documents, then enriches
the raw findings with contextual descriptions. The output: a ranked list of signals —
the 5–20 most important things worth paying attention to right now.

## When to use this skill

- "Distill my backpack"
- "What signals are in the architecture graph?"
- "What should I pay attention to?"
- "Check my graphs for issues"
- "Run signal detection"
- "What's noteworthy across my knowledge?"
- After mining and refining, as the final pipeline step
- Periodically, to catch changes or drift in growing graphs

## When NOT to use

- The user wants to explore the graph interactively — use `backpack-guide`
- The user wants to mine new content — use `backpack-mine`
- The user wants KB documents — use `backpack-refine`
- The backpack is empty — there's nothing to distill

## What Signals Are

Signals are **derived salience** — computed from graph structure and KB content, not
raw data. They answer: "What matters right now and why?"

Signals are:
- **Deterministic** — same graph state always produces the same signals
- **Regenerable** — can always be recomputed from current state
- **Dismissible** — users acknowledge signals without deleting them
- **Enrichable** — raw detector output gets contextual descriptions from the LLM

## Signal Kinds

### Structural Detectors (8 kinds — no LLM, deterministic)

| Kind | What it detects | Example |
|---|---|---|
| `type_ratio_imbalance` | Problems without solutions, lopsided type distributions | "12 Risk nodes but 0 Mitigation nodes" |
| `missing_relationships` | Type pairs with no edges between them | "Person and System nodes exist but none are connected" |
| `property_completeness` | Nodes missing properties common to their type | "4 of 9 Vendor nodes are missing a 'contract_expiry' property" |
| `underconnected_important` | Important nodes with few connections | "Budget Q3 has cost properties but only 1 connection" |
| `disconnected_island` | Clusters separated from the main graph | "3 nodes form an island disconnected from the main graph" |
| `cross_graph_entity` | Same entity name appearing in multiple graphs | "'PostgreSQL' appears in both architecture and dependencies graphs" |
| `kb_graph_gap` | KB documents not linked to matching graphs | "Doc 'Microbiome Summary' has tags matching gut-microbiome but no sourceGraphs link" |
| `coverage_asymmetry` | Type coverage imbalance across related graphs | "architecture graph has 8 Service nodes but related infrastructure graph has none" |

### Semantic Detectors (future — LLM-based)

- `kb_contradiction` — conflicting claims across KB documents
- `buried_actionable` — actionable items buried in document prose
- `cross_graph_pattern` — patterns spanning multiple graphs

## The Protocol

### Step 1: Detect

Run signal detection across the active backpack.

1. `backpack_active` — confirm which backpack is active
2. `backpack_signal_detect` — runs all enabled detectors against all graphs and KB documents

This returns up to 15 top signals ranked by score, with evidence context (node IDs,
labels, types, properties).

Tell the user what was scanned:
`Distilling signals across <N> graphs and <M> KB documents.`

### Step 2: Enrich

Raw detector output is mechanical — "type_ratio_imbalance detected between Problem (5)
and Solution (0)." That's useful but not actionable. Enrichment adds context.

For each signal returned by detection:

1. **Read the evidence.** Use `backpack_get_node` on evidence node IDs to understand
   what the entities actually are.
2. **Write a contextual description.** Translate the structural finding into domain
   language. Not "5 Problem nodes have no Solution edges" but "The microbiome graph
   surfaces 5 bacterial species linked to inflammation but no documented therapeutic
   interventions or probiotic counterparts."
3. **Save the enrichment.** Use `backpack_signal_enrich` with the enriched descriptions.

### Step 3: Present

Present the enriched signals to the user, grouped by severity:

```
SIGNALS — <backpack name> — <date>

CRITICAL (act now)
  [signal title — enriched description — evidence]

HIGH (address soon)
  [signal title — enriched description — evidence]

MEDIUM (worth noting)
  [signal title — enriched description — evidence]

LOW (awareness)
  [signal title — enriched description — evidence]
```

For each signal:
- **Title** — one line, what matters
- **Description** — the enriched contextual explanation
- **Evidence** — which nodes/documents support the signal
- **Graph** — which graph it came from (with viewer link)

### Step 4: Triage

After presenting, help the user act on signals:

1. **Dismiss noise.** If a signal isn't relevant, use `backpack_signal_dismiss` to
   remove it. Dismissed signals won't reappear until the graph state changes.
2. **Investigate.** For actionable signals, offer to drill into the evidence —
   `backpack_get_node`, `backpack_get_neighbors`, `backpack_explain_path` on the
   evidence nodes.
3. **Fix.** If a signal reveals a graph quality issue (missing edges, orphan nodes,
   incomplete properties), offer to fix it directly.
4. **Tune.** If too many or too few signals, adjust sensitivity with
   `backpack_signal_configure`. Range is 0.0 (only critical) to 1.0 (everything),
   default 0.5.

## Sensitivity

Sensitivity controls detector thresholds. It's a single number from 0.0 to 1.0:

- **0.0** — Only critical signals. Tight filters. For mature, well-maintained graphs.
- **0.5** — Default. Balanced detection.
- **1.0** — Everything. Loose filters. For new graphs where you want maximum visibility.

Adjust with `backpack_signal_configure`. The setting persists across runs.

When the user says "too noisy" → lower sensitivity. "Missing things" → raise it.

## Enrichment Discipline

### Domain language, not graph language

Bad: "Node n_species_abc has degree 1 which is below the average of 4.2 for type Bacterium."
Good: "Bacteroides fragilis is connected to only 1 other entity despite producing 3 known
metabolites — this suggests the graph is missing relationships to the metabolic pathways,
host interactions, or gut regions it colonizes."

### Actionable, not descriptive

Bad: "There is a type ratio imbalance between Problem and Solution nodes."
Good: "The graph documents 5 operational problems but no solutions or mitigations. Consider
adding action plans, vendor alternatives, or process improvements for each flagged issue."

### Concise

Each enriched description should be 1–3 sentences. The signal title carries the headline;
the description adds "so what?" and "now what?"

## Integration with Other Skills

The knowledge pipeline: **mine → refine → distill**

| Workflow | Skill chain |
|---|---|
| Full pipeline | `backpack-mine` → `backpack-refine` → `backpack-distill` |
| Periodic health check | `backpack-distill` alone (re-run on existing graphs) |
| After mining more content | `backpack-distill` to catch new structural issues |
| Deep analysis | Full pipeline → use critical/high signals as input to decisions |
| Graph quality maintenance | `backpack-distill` → fix flagged issues → `backpack-distill` again to verify |

Each step is independently useful. A user can distill without mining or refining — any
graph with content can produce signals.

## Re-running

Signals are regenerable. Re-running distill on the same backpack:
- Recomputes all signals from current state
- Preserves dismissed signals (they only reappear if the underlying graph state changes)
- Updates enriched descriptions

Run distill periodically on growing graphs to catch drift, new gaps, or emerging patterns.

## Anti-Patterns

**Presenting raw detector output without enrichment.** The whole point of the LLM step
is turning mechanical findings into contextual, actionable language. Don't skip it.

**Enriching signals you haven't read the evidence for.** Always `backpack_get_node` on
evidence IDs before writing descriptions. Generic enrichments ("this could be a problem")
are worse than no enrichment.

**Dismissing signals without telling the user.** Dismissal is a user decision. Present
the signal, let them decide. You can recommend dismissal ("this looks like a false
positive because...") but don't auto-dismiss.

**Running distill on an empty backpack.** If there are no graphs or all graphs are empty,
stop. Tell the user to mine first.

**Over-tuning sensitivity.** If the user keeps toggling sensitivity, the real issue is
probably graph quality, not detector thresholds. Suggest fixing the underlying graph
issues instead.

## Example

User: "distill my backpack"

You:
1. `backpack_active` → personal backpack, 3 graphs
2. `backpack_signal_detect` → 8 signals detected (1 critical, 2 high, 3 medium, 2 low)
3. Read evidence nodes for top signals
4. Enrich:
   - CRITICAL: "The gut-microbiome graph documents 5 inflammation-linked bacteria but
     zero therapeutic interventions or probiotic counterparts. The problem side of the
     graph is growing without documented responses."
   - HIGH: "Bacteroides fragilis appears in both gut-microbiome and dietary-impacts
     graphs with different property sets — the dietary-impacts version has metabolite
     data that gut-microbiome is missing."
5. `backpack_signal_enrich` with contextual descriptions
6. Present grouped by severity with viewer links
7. Offer to investigate, dismiss, or fix

Summary: "Distilled 8 signals across 3 graphs. 1 critical: unaddressed inflammation
pathways in gut-microbiome. 2 high: cross-graph entity inconsistency and missing
metabolite relationships."
