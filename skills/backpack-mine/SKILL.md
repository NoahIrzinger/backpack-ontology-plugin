---
name: backpack-mine
description: >
  This skill should be used when the user asks to "mine a topic", "build a learning graph
  from scratch", "research a topic into the backpack", "expand a graph automatically",
  "ralph wiggum loop", "autonomous mining", or wants to autonomously grow a learning graph
  by iterating over external sources. Also use when the user mentions "mining", "knowledge
  mining", "auto-research", or asks Claude to "go grow this graph for me".
metadata:
  version: "0.2.0"
---

# Backpack Mine

Autonomous knowledge graph builder. Take a topic, run an iteration loop that finds sources,
extracts relational facts, validates them, and writes them to a learning graph. Stop when
one of the termination conditions is hit.

This skill is the **driver loop**. The LLM (you) is the worker. Backpack tools handle
persistence and validation. Source discovery uses your built-in WebSearch / WebFetch / Read
tools — no separate retrieval system.

## When to use this skill

- "Mine spanish wines"
- "Build a learning graph about the gut microbiome"
- "Research the history of postgres into the backpack"
- "Expand my llm-research graph with 30 more nodes from web sources"
- Any time the user wants to grow a graph autonomously instead of typing entities by hand.

## Read the three-role rule first

This skill builds learning graphs. Before adding ANY content, ask: does this fact belong in
a graph? See `backpack-guide` for the full rule. The short version:

- A discrete fact connecting two entities by a named relationship → graph
- A procedural step ("first do X then Y") → skill, not graph. **Skip.**
- An environmental briefing ("postgres is a popular database") → CLAUDE.md, not graph. **Skip.**

The validator catches some role violations as warnings, but you are the first line of
defense. When you extract from a source, filter out anything that isn't a discrete relational
fact.

## Defaults

Unless the user overrides them, use these:

| Parameter | Default | What it means |
|---|---|---|
| `iterations` | 10 | Hard cap on loop iterations |
| `targetNodes` | 50 | Stop when graph reaches this total node count |
| `convergenceThreshold` | 2 | Stop if last 2 iterations each added fewer than this many new nodes |
| `minConnections` | 1 | Skip a node if it would end up with fewer than this many edges |
| `sourceMode` | hybrid | `seeded` (user-provided URLs/files only), `web` (search only), `hybrid` (seeded first then web) |

The user can override any of these in their request. Read the request carefully — if they
say "mine 30 nodes about X", that's `targetNodes: 30`. If they say "20 iterations max",
that's `iterations: 20`.

## Setup (run once at the start)

1. **Confirm the active backpack.** Call `backpack_active` to see which backpack is active.
   Mining writes to the active backpack's graphs directory — if the user wanted to mine into
   a different backpack, switch first with `backpack_switch <name>`. Surface the active
   backpack name in your first message: "Mining into your **work** backpack." Never
   silently mine into the wrong backpack. If the user's request implies a specific backpack
   ("mine this into my shared work folder"), switch before starting; if ambiguous, ask.

2. **Confirm or create the graph.** `backpack_list` first. If a graph for the topic already
   exists, ask the user whether to expand it or start fresh in a new graph. If new:
   `backpack_create <name> "<one-line topic description>"`. Pick a stable kebab-case name
   like `gut-microbiome` or `spanish-wines`.

3. **Snapshot for rollback.** `backpack_snapshot <graph> "before mining: <topic>"`. Capture
   the returned version number.

4. **Learn the existing vocabulary.** `backpack_describe <graph>`. Note existing node types
   and edge types — match them in subsequent extractions to prevent type drift. If the graph
   is empty, you're inventing the vocabulary: pick consistent names (PascalCase for nodes,
   SCREAMING_SNAKE_CASE for edges) and use them throughout the run.

5. **Queue seed sources, if any.** If the user gave URLs, file paths, or documents in their
   request, queue those as the starting sources for iteration 1. Otherwise iteration 1 starts
   with WebSearch on the topic.

6. **Tell the user the plan.** One terse line, naming the active backpack:
   `Mining "<topic>" into <backpack>/<graph> for up to <iterations> iterations or <targetNodes> nodes. Snapshot v<N> captured. Starting iteration 1.`

## The iteration loop

For each iteration `i` from 1 to `iterations`:

### 1. Pick the target

What to mine this iteration. Use this priority order:

1. **Iteration 1:** the user's seed topic (or first queued seed source).
2. **Iterations 2+:** the most under-connected node in the graph that hasn't been expanded
   yet. Use `backpack_audit` (orphans, weak nodes) or `backpack_stats` (degree table sorted
   ascending) to find candidates. Prefer "frontier" nodes — recently added, low degree.
3. **Fallback:** the most under-represented node type — pick a type with few instances and
   look for more entities of that type in the next source.

### 2. Find sources

Based on `sourceMode`:

- **seeded:** read the next queued seed source (URL via WebFetch, file via Read).
- **web:** WebSearch for the target. Pick the top 1–3 results that look authoritative
  (Wikipedia, primary documentation, well-known domain sites). Fetch with WebFetch.
- **hybrid (default):** drain queued seeds first, then fall back to web search.

Reject sources you can't actually read (paywalls, JS-only pages, dead links). Move on to
the next candidate. If two consecutive web searches return nothing usable, stop the run
and tell the user the topic is too narrow.

### 3. Extract

Read the source. Identify entities and relationships that are:

- **Relational** — a thing connected to another thing by a named relationship
- **Discrete** — atomic facts, not paragraphs of prose
- **Composable** — they make sense as nodes that other entities can reference

Skip:

- Procedural content ("to install postgres, run X then Y") → not graph material
- Briefings ("postgres is open source") → too vague, no relationship
- Anything you can't anchor to a named relationship between two entities
- Anything that would be a duplicate of an existing node (you already saw the vocabulary
  in step 3 of setup — use it)

For every extracted node, attach two properties:

- `source` — the URL or file path the entity came from
- `minedAt` — ISO timestamp of this iteration

For every extracted edge that came from a specific source (rather than being inferred),
attach the same `source` property.

### 4. Validate (dry run)

Call `backpack_import_nodes` with `dryRun: true`. The validator returns:

- **Errors** (broken edges, invalid property shapes, self-loops) — fix by editing your
  proposed batch and re-running the dry run. Errors block the commit.
- **Warnings** (type drift, duplicates, role violations) — handle each:
  - **Type drift:** rename to the canonical variant the validator suggested.
  - **Duplicates:** drop your version. If you have new properties for the existing node, use
    `backpack_update_node` on the existing ID separately.
  - **Role violations:** drop the offending node entirely. It doesn't belong in a graph.

### 5. Commit

When the dry run is clean (or has only acceptable warnings you've addressed), call
`backpack_import_nodes` again **without** `dryRun` to commit. Capture the actual node and
edge counts added.

### 6. Log progress

One terse line, no JSON dump:

`Iteration <i>/<iterations>: added <N> nodes, <E> edges. <total> total.`

### 7. Check termination

After every iteration, check in this order:

1. **Hit the cap?** `i >= iterations` → stop. Reason: "iteration cap reached."
2. **Hit the node target?** `total nodes >= targetNodes` → stop. Reason: "node target reached."
3. **Converged?** Both of the last two iterations added fewer than `convergenceThreshold`
   new nodes → stop. Reason: "converged — sources exhausted."
4. **Otherwise:** continue to the next iteration.

If you stop, jump to End of Run.

## End of run

1. **Normalize.** `backpack_normalize <graph>` (defaults to dryRun=true). Review the plan.
   If it's small and obviously correct (case fixes, separator fixes), apply it with
   `dryRun: false`. If it's large or surprising, leave it for the user to review.

2. **Health check.** `backpack_health <graph>`. The full report.

3. **Final summary.** One paragraph:
   ```
   Mining complete: added <N> nodes, <E> edges across <i> iteration(s).
   Reason: <termination reason>.
   Snapshot v<N> is your rollback point.
   View: http://localhost:5173#<graph>
   ```

4. **Suggest next steps** based on what `backpack_health` returned:
   - "I noticed <X> orphan nodes — want me to add edges?"
   - "<Y> nodes have role violations — want me to clean them up?"
   - "Want to run another <K> iterations to deepen specific areas?"
   - "Type drift needs another normalize pass — want me to apply it?"

## Resuming a run

If the user asks to "continue mining <topic>" or the previous run was interrupted, just run
this skill again on the same graph. The loop is stateless — it picks up from whatever the
graph currently looks like by re-reading types and weak nodes via `backpack_describe` and
`backpack_audit`. The iteration cap restarts; the node target carries forward (since it's
a total, not an increment).

## Anti-patterns

- **One source per iteration when one source has 50 entities.** If a Wikipedia article is
  rich, mine the whole article in a single iteration. An "iteration" is a unit of decision-
  making, not a unit of data volume.
- **Writing nodes without sources.** Every mined node needs a `source` property. Without it,
  the graph becomes untraceable LLM hallucination.
- **Ignoring validator warnings.** Type drift compounds — drop bad nodes and rename to the
  canonical type before committing.
- **Looping forever on a thin topic.** If web search returns nothing relevant twice in a
  row, stop and tell the user the topic is too narrow.
- **Mining procedural / briefing content.** The three-role rule still applies. Skip anything
  that belongs in a skill or CLAUDE.md.
- **Bypassing the dry run.** Always preview before committing.
- **Adding nodes that would be orphans.** Respect `minConnections` — if a candidate node
  has no plausible relationship to anything in the proposed batch or in the existing graph,
  don't add it.

## Example invocation

User: "mine the gut microbiome into a learning graph"

You:
1. `backpack_list` → no `gut-microbiome` graph yet
2. `backpack_create gut-microbiome "Bacterial species, metabolites, and host interactions in the human gut microbiome"`
3. `backpack_snapshot gut-microbiome "before mining: gut microbiome"` → version 1
4. `backpack_describe gut-microbiome` → empty graph; you'll invent the vocabulary
5. Tell user: `Mining "gut microbiome" into gut-microbiome for up to 10 iterations or 50 nodes. Snapshot v1 captured. Starting iteration 1.`
6. **Iteration 1:** WebSearch "human gut microbiome overview" → fetch top result (Wikipedia
   article) → extract ~15 nodes (Bacteroides, Firmicutes, butyrate, SCFAs, host immune
   modulation, etc.) and ~25 edges (PRODUCES, METABOLIZES, COLONIZES, MODULATES) → import_nodes
   with dryRun=true → no warnings → import_nodes commit → log
   `Iteration 1/10: added 15 nodes, 25 edges. 15 total.`
7. **Iteration 2:** `backpack_audit` → Bacteroides has only 3 connections, frontier candidate.
   WebSearch "Bacteroides species human gut" → fetch → extract 6 species of Bacteroides and
   their characteristic metabolites → dryRun → 1 type drift warning ("bacteria" should be
   "Bacterium") → fix → commit. Log.
8. **Iterations 3–6:** continue picking weak nodes, fetching, extracting.
9. **Iteration 7:** added 1 node. **Iteration 8:** added 0 nodes. Convergence triggered.
10. **End of run:** normalize (clean), health check (4 orphans surfaced), final summary,
    viewer link, suggest connecting orphans.

## What this skill does NOT do

- It does not call out to LLM APIs directly. The LLM is you, in the user's existing session.
- It does not run headlessly or on a schedule. For scheduled mining, use a separate driver.
- It does not bypass the validator. Every commit goes through `backpack_import_nodes`.
- It does not invent its own MCP tools. Everything it needs already exists in backpack.
