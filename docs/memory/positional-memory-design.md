# OpenClaw and 3D positional memory design

## Objectives and non goals

**Objectives**

- Provide non destructive memory compaction by archiving full fidelity records and indexing summaries.
- Keep memory inspectable and auditable with stable hash references.
- Enforce consent aware recall using privacy and policy gates.
- Keep the feature optional and off by default with minimal infrastructure.

**Non goals**

- Replacing full fidelity archives with summaries.
- Auto mutating personality capsules or long term persona.
- Introducing required services or new runtime dependencies for core installs.

**Intent**

- Summaries are indexes, not replacements.
- Hash pointers are the durable reference for all recall paths.
- Retrieval always respects consent policy and channel boundaries.

## Scope overview

OpenClaw can add a non destructive memory scaffold by combining a hash addressed archive with a positional index. Compaction writes immutable memory documents, stores summary plus coordinates in SQL, and replaces in session context with a summary block that contains the hash pointer. The model sees summaries by default and requests full documents through a tool only when policy allows it.

## Architecture

### Archive layer

- Immutable memory documents stored by content hash.
- Stored as files or a document store.
- Hash computed over canonical bytes before compression.

### Index layer

- SQL table that stores summary, hash, coordinates, and consent flags.
- Acts as a card catalog over the immutable archive.

### Retrieval layer

- Tool that resolves a hash to a document.
- Optional search over summaries and coordinates.

### Policy layer

- Defines what can be archived, summarized, and recalled.
- Enforces consent boundaries and channel rules.

## Memory document format

A memory document is full fidelity and immutable.

- `hash_algo`
- `hash`
- `created_at`
- `source`
- `messages`
- `attachments`
- `tags`
- `consent_scope`

## SQL index sketch

`memory_index`

- `hash` primary key
- `summary`
- `t_x`
- `y_privacy`
- `z_abstraction`
- `cluster_id`
- `decay_score`
- `reinforcement_count`
- `last_accessed_at`
- `consent_flags`
- `source_session`
- `source_channel`
- `created_at`

## Memoria Ontology

### Axes

- **X axis time**: relative day offset.
  - `x = 0` means today.
  - `x = -1` means yesterday.
  - `x = +1` means tomorrow.
- **Y axis public to private**: range from `-15` to `+15`.
- **Z axis abstract to physical**: range from `-15` to `+15`.

### Coordinate usage

Summaries and hash pointers are the only nodes stored in positional memory. Full fidelity documents remain in the archive and are retrieved by hash when policy allows.

## Compaction flow

1. Select a compaction range.
2. Create a memory document for the range.
3. Canonicalize and hash the document.
4. Store the document under the hash in the archive.
5. Generate a summary.
6. Insert summary, hash, and coordinates into the SQL index.
7. Replace the compacted range with a summary block and hash pointer.

## Retrieval policy

- Default behavior returns summaries only.
- Full documents require explicit tool calls and consent checks.
- Optional search returns top summaries without dereferencing.

## Configuration and opt in

This integration is optional and off by default. Example configuration keys:

- `memory.archive.enabled`
- `memory.archive.path`
- `memory.index.sql.enabled`
- `memory.index.sql.dsn`
- `memory.positional.enabled`
- `memory.positional.provider`
- `memory.compaction.mode`

## Storage lifecycle

- Hot: per hash files for fast access.
- Warm: per hash compression.
- Cold: bundled archives with a manifest mapping hash to storage location.

## Integration surface

Minimal core changes:

- Call `archive_and_index()` during compaction before summary replacement.
- Add `memory.get(hash)` and optional `memory.search(...)` tools.
- Guard behavior behind config so default behavior is unchanged.
