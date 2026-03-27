# Evaluation Guide — CDC Lakehouse Reliability Assignment

## Purpose of This Guide

This guide helps reviewers evaluate whether the candidate demonstrated senior-level data engineering judgment for a CDC-driven lakehouse workflow.

The assignment is intentionally not about building a wide analytics project. It is about whether the candidate can reason about:

- source modeling quality
- CDC correctness
- replay and recovery
- schema evolution safety
- validation parity
- historical reconstruction
- warehouse snapshot correctness
- dataset discoverability

Use this rubric to assess both the implementation and the pull request description.

---

## Reviewer Mindset

Look for:

- correctness before tooling preference
- explicit data contracts and invariants
- safe failure on schema incompatibility
- durable change retention
- clear distinction between lake history and warehouse snapshot
- thoughtful validation design
- easy-to-follow platform flow

Do not over-index on any specific tool choice if the reasoning is sound.

---

## Scoring Rubric

You may score each category on a 1–4 scale:

- **1 — Weak**
- **2 — Mixed / Partial**
- **3 — Strong**
- **4 — Exceptional**

A strong submission will usually score mostly 3s, with one or two 4s in areas such as CDC design, schema safety, or validation rigor.

---

## 1) Source Schema Design

### What Reviewers Should Look For

- 3 to 10 tables with realistic structure
- strong and weak entities
- sensible keys and relationships
- appropriate use of data types
- indexes that match likely access/change patterns
- clear constraints and invariants

### Strong Signals

- source schema resembles a realistic transactional domain
- strong vs weak entities are clearly identified and justified
- primary and foreign keys are sensible
- currency, dates, enums/statuses, and nullable fields are modeled intentionally
- indexes are not arbitrary; they support access patterns or CDC workflow assumptions
- important business rules are documented

### Weak Signals

- flat or unrealistic schema with little relational thinking
- tables exist mainly to satisfy the count requirement
- no clear strong/weak entity distinction
- missing keys, weak relationships, or under-modeled constraints
- indexes absent or added without rationale

### Reviewer Questions

- Does this look like a real transactional source system?
- Are relationships and entities modeled intentionally?
- Are constraints doing useful work, or merely storing fields?

---

## 2) CDC Strategy and Correctness

### What Reviewers Should Look For

- clear definition of how changes are captured
- inserts, updates, and deletes all handled
- durable replay/restart story
- duplicate/retry awareness
- lake and warehouse fed from a coherent CDC design

### Strong Signals

- candidate explains exactly what a “change” means in their design
- restart/replay behavior is explicit
- deletes are not ignored
- duplicate events or reprocessing are handled safely
- checkpoints, offsets, or extraction boundaries are defined
- design makes near real-time warehouse availability believable

### Weak Signals

- CDC is described vaguely as periodic refresh
- no delete handling
- no replay story
- duplicate events could corrupt history or snapshot
- near real-time claim unsupported by design

### Reviewer Questions

- If ingestion restarts mid-run, what happens?
- Can this design avoid losing or duplicating changes?
- Is the warehouse derived in a way that is consistent with the lake history?

---

## 3) Lake Modeling

### What Reviewers Should Look For

- every change captured in the lake
- append-oriented or immutable design
- enough metadata to reconstruct change history
- durable historical record

### Strong Signals

- operation type, timestamps, keys, and ordering/version metadata are preserved
- lake model supports full audit of source changes
- lake is clearly distinct from curated warehouse outputs
- append-only semantics are visible in both design and tests

### Weak Signals

- lake is effectively just another snapshot table
- updates overwrite history
- missing metadata makes reconstruction difficult
- lake and warehouse roles are blurred

### Reviewer Questions

- Can every source change be recovered from the lake?
- Is the lake truly historical, or just another transformed layer?
- What information would be needed for replay or audit, and is it present?

---

## 4) Warehouse Modeling and Time Travel

### What Reviewers Should Look For

- warehouse exposes latest snapshot
- design supports moving backward in time / restore
- historical reconstruction story is credible
- downstream model is understandable

### Strong Signals

- latest-state tables are clearly defined
- temporal, versioned, or SCD-style logic is used thoughtfully where helpful
- candidate explains how point-in-time restore works operationally
- warehouse logic preserves correctness under late or repeated changes

### Weak Signals

- warehouse only has current snapshot with no restore story
- time travel is hand-waved without data model support
- current-state logic could drift from CDC history
- update ordering assumptions are unclear

### Reviewer Questions

- Can I reconstruct prior state from this design?
- Is the warehouse really usable for restore / time-based recovery?
- Does the warehouse reflect latest state consistently?

---

## 5) Schema Change Safety

### What Reviewers Should Look For

- explicit detection of incompatible source changes
- ingestion stop behavior
- clear warning/failure signaling
- safe handling of non-backward-compatible schemas

### Strong Signals

- candidate identifies concrete compatibility rules
- breaking change detection is automated or clearly validated
- ingestion fails closed rather than silently drifting
- warnings or alerts are surfaced in a reviewable way
- schema contract is documented clearly

### Weak Signals

- schema evolution is ignored
- pipeline keeps running after breaking changes with undefined behavior
- no clear definition of what counts as incompatible
- failure signaling is vague or absent

### Reviewer Questions

- Would this pipeline stop safely on a dropped or renamed column?
- Is schema drift visible quickly?
- Does the candidate think in terms of contracts, not just happy-path ingestion?

---

## 6) Validation Parity

### What Reviewers Should Look For

- source validations carried into warehouse checks/models
- both system and business rules considered
- failures surfaced clearly
- parity documented, not implied

### Strong Signals

- not-null, uniqueness, referential, and domain rules are checked
- business rules are restated downstream intentionally
- validation logic is automated through tests/assertions/checks
- failures are made actionable

### Weak Signals

- warehouse assumes source is always valid
- only superficial row-count checks exist
- business validations omitted entirely
- no traceability from source rule to warehouse assertion

### Reviewer Questions

- Are downstream consumers protected from invalid source states?
- Did the candidate mirror the important source assumptions?
- Could validation drift over time, or is it explicit?

---

## 7) Catalog Exposure and Discoverability

### What Reviewers Should Look For

- both lake and warehouse are exposed in a discoverable way
- metadata is useful
- ownership and intended use are clear
- schema descriptions or data contract information exist

### Strong Signals

- lake and warehouse datasets are named and described clearly
- metadata includes schema, purpose, and update cadence
- consumer access path is documented
- candidate distinguishes operational data exposure from curated analytics exposure

### Weak Signals

- catalog requirement acknowledged but not addressed meaningfully
- no metadata beyond file/table names
- unclear which datasets consumers should use
- lake and warehouse access expectations are muddled

### Reviewer Questions

- Could a downstream user discover the right dataset easily?
- Is there enough metadata to understand what each dataset is for?
- Is access/documentation aligned with the modeled layers?

---

## 8) Layering and Code Quality

### What Reviewers Should Look For

- clear separation of schema, ingestion, transformation, validation, and metadata concerns
- code/SQL/pipeline logic that is easy to follow
- maintainable repository structure
- minimal unnecessary complexity

### Strong Signals

- repository layout mirrors the data platform flow
- transformations and validations are separated clearly
- ingestion logic is understandable
- tests and configs live in obvious places
- naming makes the pipeline easy to reason about

### Weak Signals

- all logic mixed together in a few scripts
- no separation between raw, modeled, and validated data
- hard to see where invariants are enforced
- structure appears accidental rather than deliberate

### Reviewer Questions

- Can another engineer extend this without re-learning everything?
- Are responsibilities clearly separated?
- Is the flow from source to catalog easy to follow?

---

## 9) Testing and Validation Depth

### What Reviewers Should Look For

- tests for CDC correctness
- tests for schema change failure
- tests for latest snapshot correctness
- tests or assertions for historical reconstruction
- evidence of Red, Blue, Green discipline

### Strong Signals

- inserts, updates, deletes, duplicates, and replay are all tested
- incompatible schema changes are tested explicitly
- warehouse correctness is asserted against change history
- validations are executable, not merely described
- candidate explains important missing tests if time constrained

### Weak Signals

- only happy-path pipeline run tested
- no schema change tests despite assignment emphasis
- no delete/history validation
- little confidence in correctness under failure or replay

### Reviewer Questions

- Do the tests prove the important platform behaviors?
- Is there confidence in correctness after restart or schema drift?
- Are the most failure-prone paths explicitly checked?

---

## Suggested Overall Rating Bands

### Exceptional
The candidate clearly understands CDC reliability, schema contracts, and temporal data design.  
Lake, warehouse, validation, and schema safety reinforce one another well.

### Strong
The solution is coherent, pragmatic, and well-structured.  
There may be simplifications, but the correctness story is convincing.

### Mixed
There are solid ideas, but one or more critical areas are underdeveloped, such as schema safety, replay, or validation parity.

### Weak
The submission behaves more like a simple ETL refresh than a reliable CDC-based data platform.  
History, schema change safety, or correctness guarantees are weak.

---

## Common Failure Patterns

Reviewers should watch for these:

- lake stores only current state instead of every change
- warehouse snapshot logic is not clearly derived from CDC history
- deletes ignored
- replay/restart behavior undefined
- incompatible schema changes not detected or not blocking
- warehouse validations weaker than source assumptions
- no credible restore/time-travel story
- catalog requirement addressed only superficially

---

## What to Value Most

When in doubt, prioritize:

1. CDC correctness and replayability
2. safe handling of incompatible schema changes
3. full change retention in the lake
4. correct latest snapshot and restore logic in the warehouse
5. validation parity with source rules
6. maintainable layering
7. breadth of tooling last

A modest solution that is deeply correct should outrank a broader but fragile one.
