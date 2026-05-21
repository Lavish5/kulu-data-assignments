# CDC Lakehouse Implementation Plan

**Status**: Planning Phase (Dummy PR)  
**Domain**: E-Commerce Order Management System  

---

## 1. Requirements Summary

### Core Assignment Goals
✓ Design and implement a **CDC pipeline** that keeps a data lake and warehouse synchronized  
✓ Support **schema change detection** with safe stop on incompatible changes  
✓ Preserve **full history** for point-in-time recovery and restore  
✓ Mirror **source validations** in warehouse models  
✓ Provide **catalog** for data discovery and access  
✓ Ensure **reproducibility** and **operational safety**

### Evaluation Criteria
- Correctness of CDC logic
- Schema and entity modeling soundness
- Handling of schema drift and validation parity
- Historical recovery capability
- Code clarity and documentation
- Reproducibility (ability to rerun end-to-end)

---

## 2. Solution Architecture

### 2.1 Source Data Model (7 Tables)

**Strong Entities** (independent lifecycle):
- `customers` — PK: customer_id
- `products` — PK: product_id  
- `orders` — PK: order_id, FK: customer_id

**Weak Entities** (lifecycle tied to parent):
- `order_items` — PK: order_item_id, FK: order_id, product_id
- `inventory_logs` — PK: log_id, FK: product_id
- `shipments` — PK: shipment_id, FK: order_id
- `returns` — PK: return_id, FK: order_item_id


### 2.2 Three-Layer Pipeline Architecture

```
SOURCE → CDC CAPTURE → LAKE → WAREHOUSE → CATALOG
         (simulated)  (events)  (snapshots) (metadata)
```

#### Lake Layer (Immutable Event Log)
- **Table**: `lake_cdc_events`
- **Purpose**: Full history preservation
- **Properties**: 
  - Append-only (no updates/deletes)
  - Monotonic sequence IDs for ordering and checkpointing

#### Warehouse Layer (Current-State + SCD2 History)
- **Current-State Tables**: 7 tables matching source schema
- **Additional Columns**:
  - `_cdc_seq`: Latest sequence number for this row
  - `_deleted`: Soft-delete flag (0=active, 1=deleted)
- **SCD2 History** (optional, for time-travel):
  - `_valid_from`: Timestamp when row became current
  - `_valid_to`: Timestamp when row was superseded

#### CDC Capture (Simulated)
- **Mechanism**: Python-based simulation of WAL-style CDC
- **Input**: Source table changes (INSERT/UPDATE/DELETE)
- **Output**: CDCRecord objects with sequence, operation, table_name, PK, data
- **Checkpointing**: Offset tracking for replay capability

### 2.3 Schema Change Detection

**Strategy**: Schema Contract Validation
- **Schema Contract**: Python dict defining expected columns per table
- **Validation Logic**:
  - Before ingestion: verify source has all expected columns
  - On mismatch: raise exception, stop pipeline, emit warning
  - Never continue silently under broken assumptions

**Incompatible Changes Detected**:
- Dropped columns (expected but not found)
- Type mismatches (e.g., DECIMAL → INT)
- Nullability changes where constraint violated
- Foreign key removals

---

## 3. Implementation Phases

### Phase 1: Foundation ✓ (Already Started)
- Source schema design
- **TODO**: Ensure all source tables created with constraints

### Phase 2: CDC & Lake (Core)
- Implement `CDCCapture` class (`pipeline/cdc.py`)
  - Simulate changes: INSERT/UPDATE/DELETE operations
  - Generate CDCRecord objects with monotonic sequence
  - Support replay from checkpoint
- Implement lake table and append logic (`pipeline/lake.py`)
  - Create `lake_cdc_events` table
  - Append CDCRecords with idempotency
  - Log all operations for audit trail

### Phase 3: Warehouse
- Create warehouse tables (`pipeline/warehouse.py`)
  - 7 tables with current-state + metadata columns
  - Create SCD2 history tables
- Implement CDC application logic
  - Upsert logic on INSERT/UPDATE
  - Soft-delete on DELETE (set `_deleted = 1`)

### Phase 4: Validation & Schema Safety
- Implement schema contract validation
  - Define SCHEMA_CONTRACT
  - Validate before ingestion
  - Fail loudly on mismatch
- Implement business validation tests 
  - Order total == SUM(order_items)
  - Stock qty ≥ 0
  - Status transitions valid
  - Referential integrity checks
  - Parity check: source constraints == warehouse constraints

### Phase 5: Catalog & Discovery
- Create catalog metadata
  - Document all tables, columns, types
  - Mark lake vs warehouse tables

### Phase 6: Testing & Documentation
- Unit tests for each module
  - CDC capture and replay
  - Lake append idempotency (may use test_cdc)
  - Upsert, soft-delete, time-travel logic
  - Business validations
  - Schema change detection

---

## 4. Key Design Decisions

### 4.1 Simulated CDC vs. Real WAL
**Decision**: Simulated CDC via Python  
**Rationale**:
- Assignment scope
- DuckDB doesn't have Debezium connector
- Production equivalent: Postgres WAL reader / Debezium

### 4.2 Lake Structure: Single vs. Multi-Table
**Decision**: Single `lake_cdc_events` table with `table_name` column  
**Rationale**:
- Simplified idempotency (single append target)
- Easy replay from checkpoint
- Unified audit trail
- Query flexibility (can filter by table_name)

### 4.3 Warehouse History: SCD2 vs. Event Sourcing
**Decision**: Dual approach
- **Current-State**: Fast queries for analytics
- **SCD2 History** (optional): Time-travel capability
  
**Rationale**:
- Current-state fast for real-time dashboards
- SCD2 history enables point-in-time queries
- Supports "what was the order total on 2025-01-15?"

### 4.4 Schema Change Handling: Validation vs. Automatic Migration
**Decision**: Fail-fast validation (no auto-migration)  
**Rationale**:
- Assignment goal: safe stop on incompatible changes
- Prevents silent data corruption
---