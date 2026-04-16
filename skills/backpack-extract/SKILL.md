---
name: backpack-extract
description: >
  This skill should be used when the user asks to "extract a graph",
  "generate docs from a graph", "document this graph", "create KB files
  from a graph", "write up what's in this graph", "summarize this graph
  into documents", or wants to produce a reusable set of knowledge base
  documents from a mined learning graph. Also use when the user mentions
  "extract", "graph documentation", or "graph to KB".
metadata:
  version: "0.1.0"
---

# Backpack Extract

Generate a core set of knowledge base documents from any learning graph. The second
step in the knowledge pipeline: `backpack-mine` digs up raw ore, `backpack-extract`
pulls out the usable material into readable, reusable prose documents.

Mining builds raw relational knowledge. Extracting turns it into documents that humans
and LLMs can read without querying the graph node-by-node.

## When to use this skill

- "Extract the spanish-wines graph"
- "Generate docs from my gut-microbiome graph"
- "Document what's in the architecture graph"
- "Write up the key findings from this graph"
- "Create KB articles from my research graph"
- After any mining run when the user wants lasting prose artifacts
- When handing off a graph to someone who hasn't explored it yet

## When NOT to use

- The graph is empty or has fewer than 5 nodes — there's nothing to refine
- The user wants to query the graph interactively — use `backpack-guide` patterns
- The user wants to mine new content into the graph — use `backpack-mine`
- The user wants a one-off synthesis answer — just use `backpack_synthesize_structured`
  or answer from the graph directly

## The Six Documents

Every graph, regardless of domain, benefits from these six documents. They work whether
the graph is about wines, biology, business operations, software architecture, or
anything else.

### 1. Overview

**What it answers:** "What is this graph about? What's in it? How big is it?"

The README for the graph. Anyone encountering it for the first time reads this.

**Content:**
- Graph name and description
- Total node count, edge count, type count
- The top 5–10 most connected entities (the hubs) with one-line summaries
- A prose paragraph describing what the graph covers and its scope
- When it was last updated (from the most recent `source_date` across nodes)

**Data needed:** `backpack_describe`, `backpack_stats` (sort by connectivity descending,
take top 10), `backpack_get_node` on each hub for summaries.

---

### 2. Taxonomy

**What it answers:** "What vocabulary does this graph use? What do the types mean?"

The schema documentation. Essential for anyone adding to the graph later — prevents
type drift by documenting what already exists.

**Content:**
- Each node type: count, what it represents in this domain, 2–3 example instances
- Each edge type: count, what relationship it represents, what node types it connects
- The meta-graph: a prose description of which types connect to which via which edge
  types (e.g., "Person → MANAGES → Organization → USES → System")

**Data needed:** `backpack_describe` for types and counts, `backpack_list_nodes` filtered
by type (limit 3) for examples, edge type analysis from `backpack_stats`.

---

### 3. Key Entities

**What it answers:** "Who or what are the most important things in this graph?"

The hubs — the centers of gravity that everything else connects to. In a business
graph these might be key people or critical systems. In a biology graph, major
organisms or pathways. In a wine graph, dominant regions or grape varieties.

**Content:**
- The top entities by connectivity (typically 10–20, scaled to graph size)
- For each: name, type, summary (from the node's summary property), connection count,
  and the 3–5 most important relationships
- Grouped by type for readability

**Data needed:** `backpack_stats` sorted by total connections descending,
`backpack_get_node` on each for properties, `backpack_get_neighbors` (depth 1) for
key relationships.

**Scaling rule:** For graphs under 50 nodes, cover all non-trivial entities (3+ connections).
For graphs over 50 nodes, cover the top 20 by connectivity. For graphs over 200 nodes,
cover the top 30 and note how many entities exist below the cutoff.

---

### 4. Relationship Patterns

**What it answers:** "How do things connect at a structural level? What's the shape?"

The meta-structure — not individual edges, but the patterns of how types relate to
each other. This is the document that reveals the graph's architecture.

**Content:**
- Type-level connection patterns: which node types connect to which, via which edge
  types, and how frequently (e.g., "78% of Person nodes connect to at least one
  Organization via WORKS_AT")
- Clusters: groups of tightly connected nodes that form natural communities
- Bridge entities: nodes that connect otherwise separate clusters
- Structural observations: is the graph hub-and-spoke? Densely connected? Fragmented
  into islands?

**Data needed:** `backpack_analyze_patterns` (dependency, frequency),
`backpack_stats` for per-type connectivity, `backpack_get_neighbors` on bridge
candidates.

---

### 5. Findings

**What it answers:** "What non-obvious things does the structure reveal?"

The analytical payoff. What you learn from looking at the graph as a whole that you
wouldn't see from any individual node. This is where the graph earns its keep over
flat documents.

**Content:**
- Pattern analysis results: frequency outliers, dependency risks, cost drivers,
  governance gaps, contract/reality mismatches
- Surprising connections: entities linked through unexpected paths
- Questions the graph raises but doesn't answer (gaps worth investigating)
- Structural anomalies: nodes with unusually high/low connectivity for their type

**Data needed:** `backpack_analyze_patterns` (all pattern types),
`backpack_synthesize_structured` for the universal intelligence questions (these
work on any domain), `backpack_health` for gap identification.

**Minimum graph size:** Skip this document if the graph has fewer than 15 nodes —
there isn't enough structure for meaningful pattern detection. Tell the user why
it was skipped and that it'll be generated on the next refine run after more mining.

---

### 6. Source Provenance

**What it answers:** "Where did this knowledge come from? How fresh is it?"

The audit trail. Essential for trust, staleness detection, and knowing which sources
have been covered vs. which haven't.

**Content:**
- All unique sources across node `source` properties, grouped by `source_type`
- Date range per source type (earliest to latest `source_date`)
- Count of nodes per source
- Coverage assessment: are there source types with only 1–2 nodes (under-mined)?
- Staleness flag: any sources older than 6 months get a note

**Data needed:** `backpack_list_nodes` (paginated, all nodes), extract `source`,
`source_type`, `source_date` properties from each. Aggregate.

**Skip condition:** If fewer than 20% of nodes have source metadata, skip this
document and note that the graph lacks provenance data. Manually-built graphs
often won't have source metadata — that's fine.

---

## The Protocol

### Step 1: Survey

Before generating any documents, understand what you're working with.

1. `backpack_active` — confirm which backpack is active
2. `backpack_describe <graph>` — types, counts, structure
3. `backpack_stats <graph>` — full connectivity analysis
4. `backpack_health <graph>` — overall health report

From the survey, determine:
- **Graph size class:** small (< 30 nodes), medium (30–200), large (200+)
- **Which documents to generate:** all six by default, but skip Findings if < 15 nodes,
  skip Source Provenance if < 20% of nodes have source metadata
- **The hubs:** top entities by connectivity (needed for Overview and Key Entities)

Tell the user the plan in one line:
`Extracting <graph> (<N> nodes, <E> edges) into <K> KB documents.`

### Step 2: Generate

For each document, in order (Overview first — it's the entry point):

1. **Query the graph** for the data that document needs (listed above per document)
2. **Write the prose** — clear, specific, domain-appropriate language. No graph jargon
   ("nodes", "edges", "connectivity") in the body text — translate to domain terms
   ("entities", "relationships", "connections"). The reader may not know what a
   learning graph is.
3. **Save to KB** via `backpack_kb_save` with:
   - `title`: `"<Graph Name>: <Document Type>"` (e.g., "Spanish Wines: Overview")
   - `id`: `"<graph>-<type>"` (e.g., `"spanish-wines-overview"`) — makes re-runs idempotent
   - `tags`: `["graph-docs", "<graph>", "<document-type>"]`
   - `sourceGraphs`: `["<graph>"]`
   - `sourceNodeIds`: IDs of the specific nodes referenced in the document (for Key Entities
     and Findings especially)
4. **Log progress:** `Saved: <title>`

### Step 3: Summary

After all documents are generated:

1. List what was created (with document IDs for reference)
2. Note what was skipped and why
3. Tell the user how to find the documents: `backpack_kb_list` filtered by tag `graph-docs`
   and graph name

## Writing Discipline

### Domain-appropriate language

The documents should read naturally for the graph's domain. A biology graph's Overview
shouldn't say "this graph has 47 nodes" — it should say "this knowledge base covers
47 biological entities including 12 bacterial species, 8 metabolites, and 15 host
interactions." Translate graph structure into domain meaning.

### Specific over general

Bad: "There are several important entities in this graph."
Good: "Bacteroides fragilis is the most connected organism (14 relationships), linking
to 4 metabolic pathways and 3 host immune responses."

### Consistent voice

All six documents should read as if written by the same knowledgeable author. Don't
shift between formal and casual, or between expert and introductory, across documents.
Match the apparent complexity of the domain — a wine graph can be warmer, a compliance
graph should be precise.

### Right-sized

Scale document length to graph size:
- Small graphs (< 30 nodes): each document 200–400 words. Don't pad.
- Medium graphs (30–200 nodes): each document 400–800 words.
- Large graphs (200+ nodes): each document 800–1500 words. Use sections and headers.

## Idempotency

The skill uses predictable document IDs (`<graph>-overview`, `<graph>-taxonomy`, etc.).
Running refine on the same graph twice **updates** the existing documents rather than
creating duplicates. This means:

- After mining more content, re-run refine to refresh the docs
- After normalizing or auditing, re-run refine to capture the changes
- The documents always reflect the current state of the graph

Tell the user when documents are being updated vs. created for the first time.

## Re-running After Changes

When the user has modified the graph since the last refine (more mining, manual edits,
normalization), re-running refine is the right call. The skill detects existing documents
by ID and updates them. Call this out:

`Updating <graph> documents (last generated <date from existing doc>). <N> nodes added since last refine.`

## Integration with Other Skills

The knowledge pipeline: **mine → extract → refine**

| Workflow | Skill chain |
|---|---|
| Full pipeline | `backpack-mine` → `backpack-extract` → `backpack-refine` |
| Research a topic end-to-end | `backpack-mine` → `backpack-extract` |
| Refresh docs after expanding | `backpack-mine` (continue) → `backpack-extract` (re-run) |
| Understand an unfamiliar graph | `backpack-extract` → read the Overview first |
| Hand off a graph to a collaborator | `backpack-extract` → share the KB docs alongside the graph |
| Deep analysis | `backpack-mine` → `backpack-extract` → `backpack-refine` → use signals as input to decisions |

## End of Run

After generating all documents, suggest the next step in the pipeline:

`Documents saved. Want me to refine the graph for signals? (/refine)`

This completes the **mine → extract → refine** pipeline. Each step is independently
useful — the user can run any skill on its own — but together they form the full
knowledge processing pipeline: extract → document → signal.

## Anti-Patterns

**Generating docs for an empty graph.** If `backpack_describe` shows 0 nodes, stop.
Tell the user to mine first.

**Writing graph jargon in documents.** The reader of these documents may never open the
graph viewer. "Node n_abc123 has 7 edges" means nothing to them. "Bacteroides fragilis
connects to 7 other entities" is better. "Bacteroides fragilis produces 3 short-chain
fatty acids and colonizes 4 gut regions" is best.

**Generating all six documents for a tiny graph.** A 10-node graph doesn't need
Findings or detailed Relationship Patterns. Scale to what the graph actually contains.

**Padding thin documents.** If a document type would be 2 sentences, either skip it
with a note or merge it into the Overview. Don't stretch 2 sentences into 2 paragraphs
of filler.

**Ignoring source metadata.** If nodes have `source`, `source_type`, and `source_date`
properties, the Source Provenance document should use them. Don't skip provenance just
because it requires aggregation work.

**Not linking sourceGraphs.** Every saved document must include the `sourceGraphs` field
pointing back to the origin graph. This is how the KB-to-graph cycle stays connected.

## Example

User: "refine the spanish-wines graph"

You:
1. `backpack_describe spanish-wines` → 82 nodes (Region: 15, GrapeVariety: 22,
   Wine: 30, Winery: 15), 140 edges, 6 types
2. `backpack_stats spanish-wines` → Rioja (18 connections), Tempranillo (15),
   Garnacha (12)...
3. `backpack_health spanish-wines` → healthy, 2 orphans
4. Plan: "Extracting spanish-wines (82 nodes, 140 edges) into 6 KB documents."
5. Generate Overview: "This knowledge base covers 82 entities across Spain's wine
   regions, grape varieties, wineries, and their wines..."
6. Generate Taxonomy: "The graph uses 4 entity types: Region (15 — wine-producing
   areas like Rioja, Ribera del Duero...), GrapeVariety (22 — both indigenous
   varieties like Tempranillo and international plantings like Cabernet Sauvignon...)..."
7. Generate Key Entities: "Rioja is the most connected region (18 relationships)..."
8. Generate Relationship Patterns: "Regions contain wineries (LOCATED_IN), wineries
   produce wines (PRODUCES), wines feature grape varieties (MADE_FROM)..."
9. Generate Findings: "Tempranillo appears in 68% of wines across all regions,
   making it the dominant variety. However, Priorat wines show no Tempranillo —
   they rely entirely on Garnacha and Cariñena..."
10. Generate Source Provenance: "Knowledge sourced from 4 web articles (Wikipedia,
    Decanter, Wine Spectator) and 2 user-provided documents, spanning March–April 2026."
11. Summary: "Generated 6 documents for spanish-wines. Find them with
    `backpack_kb_list` filtered by tag `graph-docs`."
