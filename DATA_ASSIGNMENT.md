# CDC Lakehouse Reliability Assignment — Data Engineering

## Overview

Design and implement a small **change data capture (CDC) pipeline** that keeps a data lake and a warehouse synchronized from a relational source system.

This assignment is intended to evaluate senior data engineering candidates on the kind of problems that matter in real systems:

- schema and entity modeling
- CDC correctness
- durability and replayability
- near real-time propagation
- handling schema drift and incompatible source changes
- reproducibility and operational safety
- consistency between source validations and downstream models
- historical recovery and point-in-time restore

This is intentionally **not** a toy ETL exercise. The goal is to assess whether you can design a data platform workflow that behaves safely when data changes, schemas evolve, and downstream consumers require both **full history** and **current-state analytics**.

Focus on **clarity, correctness, and robustness** over breadth.

---

## Problem Statement

You are given a transactional source system with **3 to 10 tables**. You should design the source schema yourself, but it must contain:

- a mix of strong and weak entities
- relationships between tables
- indexes
- a mix of data types, including at least:
  - currency / decimal
  - dates / timestamps
  - enum-like fields
  - identifiers / foreign keys
  - nullable fields where appropriate

Your task is to build a CDC-based pipeline that:

1. captures all source changes
2. writes every change to a **lake**
3. keeps a **warehouse** updated for near real-time availability
4. stops ingestion and warns users if source schema changes incompatibly
5. preserves enough history to move backward in time and restore data via the warehouse
6. mirrors source system and business validations in the warehouse models
7. exposes both the lake and warehouse datasets in a **catalog** for access

You may choose the domain, but it should resemble a realistic transactional system rather than a flat reporting dataset.

Examples of suitable domains:

- wallet / payments / transfers
- e-commerce orders and line items
- ride-hailing trips and settlements
- subscriptions and invoices
- ticketing and reservations

---

## Time Expectation

Please spend **3–5 hours** on this assignment.

We do not expect a production-complete data platform. We care more about:

- correct reasoning
- sensible modeling
- reliable CDC design
- explicit assumptions and tradeoffs
- reproducibility
- clear documentation

A smaller, well-reasoned solution is better than a broad but shallow one.

---

## Core Requirements

### 1) Source Data Model

Design a source relational model with **3 to 10 tables**.

Your model must include:

- strong entities
- weak entities
- one-to-many or many-to-one relationships
- appropriate primary keys and foreign keys
- indexes where lookup or change-capture performance would reasonably matter
- a mix of data types such as:
  - amount / currency
  - date / timestamp
  - enum or status
  - nullable optional attributes

Examples of strong entities:
- customer
- order
- wallet
- merchant

Examples of weak entities:
- order_item
- wallet_balance_history
- payment_attempt
- reservation_hold_event

You should document:

- why the entities were chosen
- entity relationships
- what makes a table strong vs weak in your model
- important invariants
- expected change patterns

### 2) CDC Pipeline

Build a CDC flow from the source into both a **lake** and a **warehouse**.

Required behavior:

- all inserts, updates, and deletes in the source must be captured
- the **lake** must retain **every data change**
- the **warehouse** must reflect the **latest snapshot**
- the warehouse should become available in **near real time**
- the design must support replay and recovery

You may choose the exact implementation approach, for example:

- database CDC log / WAL-based ingestion
- timestamp/version-based incremental extraction
- simulated CDC if necessary for local scope

If you simulate CDC, document what is simulated and what would change in a production setup.

### 3) Schema Change Detection and Safe Stop

Your pipeline must explicitly detect source schema changes.

Assume there is **no backward compatibility guarantee** from source systems.

Required behavior:

- if an incompatible source schema change is detected, ingestion must stop
- the system must emit a clear warning / failure signal
- the lake and warehouse should not continue ingesting silently under broken assumptions
- the detection strategy must be documented clearly

Examples of incompatible changes:

- dropped column
- renamed column
- changed data type
- changed enum domain
- nullable to non-nullable change where current logic breaks
- changed primary/foreign-key assumptions

You do not need to solve automatic migration of breaking changes. We care that you fail safely and observably.

### 4) Historical Recovery and Time Travel

We need the ability to move backward in time and restore data via the warehouse.

Your design should explain and, where practical, demonstrate:

- how historical states are preserved
- how a prior point in time can be reconstructed
- how restore / rollback would work operationally
- what data model in the warehouse enables this

A strong solution will distinguish clearly between:

- event/change history in the lake
- current-state analytical snapshot in the warehouse
- historical reconstruction support via versioned or temporal modeling

### 5) Validation Parity

All important source validations should exist in the warehouse as well.

This includes both:

- **system validations**
  - primary key uniqueness
  - referential integrity
  - not-null rules
  - valid types / domains

- **business validations**
  - allowed status transitions
  - non-negative monetary amounts where required
  - line item totals matching order totals, if modeled
  - settlement dates not preceding creation dates, if modeled

Document:

- which validations exist in the source
- how the same validations are asserted or tested in the warehouse
- how validation failures are surfaced

### 6) Catalog Exposure

Expose both the lake and warehouse datasets in a **catalog**.

You may implement this minimally for the scope of the assignment, but the design should make access paths clear.

At minimum, document:

- dataset names / schemas
- which datasets are lake vs warehouse
- who they are intended for
- how a consumer discovers and queries them
- what metadata is published

If you implement a lightweight local catalog approach, explain the production analogue.

---

## Data Platform Expectations

We expect a clean separation of concerns across your data platform design.

A strong submission will usually make the following distinctions clear:

### Source Layer
- transactional source schema
- authoritative system of record
- source validations and constraints

### Ingestion / CDC Layer
- change capture logic
- schema change detection
- replay / checkpointing strategy
- failure handling

### Lake Layer
- immutable or append-oriented change history
- raw or normalized change records
- enough detail to reconstruct every change

### Warehouse Layer
- curated latest-state snapshot
- analytical tables or models
- versioned history strategy if used for restore / time travel

### Quality / Validation Layer
- tests, constraints, or assertions
- parity checks with source assumptions
- data contract or schema checks

### Catalog / Access Layer
- discoverability
- published metadata
- access boundaries or intended consumers

---

## Documentation-First Workflow

Before writing implementation code, document the behavior of your system.

At minimum, include:

- source schema
- CDC contracts
- assumptions about change capture
- lake data model
- warehouse data model
- schema evolution policy
- stop-the-line behavior for incompatible changes
- time-travel / restore strategy
- validation strategy
- catalog exposure strategy

We are intentionally looking for candidates who can make the non-happy-path behavior of a data platform explicit before building it.

---

## Functional Expectations

Your submission should include the following artifacts or equivalents.

### A) Source Schema Definition
Provide DDL, migration files, model definitions, or equivalent.

Should include:
- keys
- constraints
- indexes
- representative enum/status fields
- sample data if helpful

### B) CDC Design / Implementation
Provide code, SQL, pipeline definitions, orchestration, or scripts that show how:

- source changes are detected
- changes are written to the lake
- latest-state data is published to the warehouse
- checkpoints / offsets / extraction boundaries are managed
- schema changes are detected and handled safely

### C) Lake Output
The lake should preserve **every change**.

Examples of acceptable representations:
- append-only change table
- bronze/raw CDC records
- JSON or parquet change files
- event log with operation type and timestamps

The important point is that change history is durable and queryable.

### D) Warehouse Output
The warehouse should represent the **latest snapshot** for downstream use.

A strong solution may also include:
- SCD2 or temporal history support
- current-state dimensions/facts
- versioned snapshot tables
- restore-oriented modeling

### E) Validation Layer
Include tests, assertions, expectations, or SQL checks that demonstrate:

- system constraint parity
- business rule parity
- freshness / completeness checks where relevant
- schema compatibility checks

### F) Catalog Metadata
Provide a minimal catalog artifact, metadata file, registry entry, or documented equivalent that exposes:

- lake datasets
- warehouse datasets
- owners / intended users
- schema and description
- update cadence

---

## Reliability Requirements

Your design should explicitly account for the following:

- duplicate CDC events
- retries after partial failure
- out-of-order change arrival where relevant
- ingestion restart after checkpointed progress
- incompatible schema changes
- deletes in the source
- late-arriving updates
- recovery from historical data

We are not testing for a perfect enterprise platform. We are testing whether you can reason carefully about correctness and safe operations.

---

## Architecture Expectations

Use a clean, layered approach.

We expect to see something close to the following, adapted to your chosen stack:

### 1) Source Modeling Layer
- DDL / source entities / seed data
- explicit constraints and indexes

### 2) CDC / Ingestion Layer
- extraction or log consumption logic
- checkpointing / offsets
- schema compatibility checks

### 3) Lake Writing Layer
- durable append-only change persistence
- full change retention

### 4) Warehouse Modeling Layer
- current-state snapshot logic
- historical restore strategy
- downstream-friendly schema

### 5) Validation / Quality Layer
- data tests
- rule assertions
- schema checks

### 6) Catalog Layer
- dataset registration / metadata exposure

You do not need an elaborate platform framework, but the flow should be easy to reason about and maintain.

---

## Testing Requirements

We expect tests or validation steps that prove platform behavior, not only happy-path execution.

### Required Testing Categories

#### 1) Modeling and Constraint Tests
Cover:
- key constraints
- relationship integrity
- index assumptions where relevant
- source validation rules

#### 2) CDC Correctness Tests
Cover:
- insert capture
- update capture
- delete capture
- duplicate or replay safety
- restart/recovery behavior

#### 3) Schema Change Safety Tests
Cover:
- incompatible schema change detection
- ingestion stop behavior
- warning / failure signal emitted

#### 4) Warehouse Correctness Tests
Cover:
- latest snapshot correctness
- validation parity with source
- historical reconstruction or restore logic

#### 5) Catalog Tests or Validation
Cover:
- catalog entries exist for lake and warehouse datasets
- schema / metadata exposure is correct

### Red, Blue, Green Discipline

Please follow a **Red, Blue, Green** workflow:

- **Red**: write a failing test or failing assertion for a required behavior
- **Blue**: implement the minimum necessary logic to make it pass
- **Green**: refactor for clarity, reliability, and maintainability while preserving behavior

You do not need to submit every intermediate step, but your solution should reflect disciplined iteration.

---

## Technology Choices

You may use any language, framework, orchestration tool, or local data stack you are comfortable with.

Examples include:
- Python / SQL
- dbt
- Spark
- Airflow / Dagster / Prefect
- PostgreSQL / DuckDB / ClickHouse / BigQuery-like local analogues
- Delta / Iceberg / Hudi-like local analogues
- simple scripts with well-structured SQL models

Please choose a stack that makes your correctness story easy to understand.

If you make simplifying assumptions, document them.

---

## Non-Goals

You do **not** need to build:

- a full production deployment platform
- complex IAM or security controls
- a polished UI
- enterprise-scale orchestration
- exhaustive BI dashboards

Keep the scope tight. Depth is more important than breadth.

---

## Deliverables

Please submit your solution as a **Pull Request**.

Your PR should include:

- source schema definitions
- CDC implementation or simulation
- lake and warehouse outputs/models
- validation tests/checks
- catalog metadata or equivalent
- setup instructions
- how to run validations/tests
- a short architecture/design explanation
- assumptions, tradeoffs, and limitations

### PR Description Requirements

Your PR description must explicitly explain:

1. **Source schema design**
   - tables
   - strong vs weak entities
   - keys
   - relationships
   - indexes
   - validation rules

2. **CDC strategy**
   - how changes are captured
   - how replay/restart works
   - how duplicates are handled
   - how deletes are handled

3. **Lake and warehouse modeling**
   - how the lake captures every change
   - how the warehouse maintains latest snapshot
   - how time travel / restore is supported

4. **Schema change safety**
   - how incompatible changes are detected
   - how ingestion is stopped
   - how warnings/failures are surfaced

5. **Validation parity**
   - how source validations are represented downstream
   - how failures are checked and reported

6. **Catalog exposure**
   - how lake and warehouse datasets are published/discovered

7. **Responsible AI usage**
   - whether you used AI tools
   - where they helped
   - what you personally reviewed, validated, or corrected

Please be candid. AI usage is allowed, but we care about engineering judgment, not generated volume.

---

## What We Are Optimizing For

A strong submission is one that is:

- correct under replay, restart, and schema change
- explicit about invariants and tradeoffs
- easy to reason about
- layered cleanly
- backed by meaningful tests and validations

A smaller but robust solution is preferred over a broader but fragile one.
