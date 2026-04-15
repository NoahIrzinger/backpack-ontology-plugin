---
name: backpack-kb
description: >
  This skill should be used when the user asks to "save a document",
  "create a KB article", "search my documents", "mount a folder",
  "add a KB mount", "ingest a document", "synthesize into a document",
  or wants to work with the knowledge base layer. Also use when the user
  mentions "knowledge base", "kb", "documents", "mounts", or "collections".
metadata:
  version: "0.1.0"
---

# Backpack KB

The knowledge base layer adds markdown documents with metadata alongside learning graphs.
Documents live in **mounts** — filesystem directories that Backpack indexes. Obsidian vaults,
shared drives, team folders, and any local directory can be mounted.

## What is the KB?

A collection of markdown documents organized into mounts. Each document has:

- **Content** — markdown body
- **Metadata** — title, tags, collection, creation/modification dates
- **Mount** — which directory it belongs to

Documents and learning graphs are complementary: documents hold prose, graphs hold structured
relational facts. They feed each other through the document-to-graph pipeline.

## Mounts

A mount is a filesystem directory that Backpack watches for documents.

- `backpack_kb_mount` — add a mount (provide a path) or remove an existing one
- `backpack_kb_mounts` — list all registered mounts with their paths and document counts

Any directory works: an Obsidian vault (`~/Documents/obsidian-vault`), a shared Google Drive
folder, a team OneDrive directory, or a plain local folder. Backpack indexes markdown files
within the mount.

## Document tools

- `backpack_kb_list` — list documents (filter by mount, tags, collection)
- `backpack_kb_read` — read a document's full content and metadata
- `backpack_kb_search` — full-text search across all mounted documents
- `backpack_kb_save` — write a new document or update an existing one
- `backpack_kb_ingest` — read document content for processing (extraction, mining)
- `backpack_kb_delete` — remove a document

## Common patterns

### Saving a synthesis from a graph

When the user has a graph and wants a written summary:

1. Query the graph (search, traverse, get neighbors) to gather relevant nodes
2. `backpack_synthesize` the gathered content into structured prose
3. `backpack_kb_save` the synthesis as a document with title, tags, and collection

### Ingesting external documents into a graph

When the user has documents and wants to build a graph from them:

1. `backpack_kb_list` or `backpack_kb_search` to find relevant documents
2. `backpack_kb_ingest` to read the document content
3. `backpack_extract` entities and relationships from the content
4. `backpack_import_nodes` to add the extracted data to a learning graph

### Searching across mounts

`backpack_kb_search` searches across all mounted directories. Use it when the user asks
"do I have anything about X?" or "find my notes on Y." Results include the mount, document
path, and matching excerpts.

### Organizing with collections and tags

When saving documents, use `tags` for cross-cutting concerns (topic keywords) and
`collection` for grouping (project name, client name, domain). These are filterable
in `backpack_kb_list`.

## The graph-document cycle

Documents and graphs feed each other:

```
Documents → backpack_kb_ingest → backpack_extract → Graph
Graph → backpack_synthesize → backpack_kb_save → Documents
```

- **Documents inform graphs:** ingest a research paper, extract entities, build a graph
- **Graphs produce documents:** synthesize a graph region into a readable summary, save it

This cycle means the KB is both a source of raw material for graphs and a destination for
graph-derived insights.

## Syncing KB to cloud

KB documents sync alongside graphs through the viewer's sync controls. Push packages both
graphs and documents into the encrypted BPAK archive. Pull restores them into the local
cache backpack. No separate sync step is needed for KB content.
