## Summary
Describe what you implemented.

## Source Schema Design
- Domain chosen:
- Strong entities:
- Weak entities:
- Keys, relationships, and indexes:
- Source validation rules:

## CDC Strategy
- How changes are captured:
- How inserts, updates, and deletes are handled:
- How replay/restart works:
- How duplicates are handled:

## Lake and Warehouse Modeling
- How the lake captures every change:
- How the warehouse maintains latest snapshot:
- How time travel / restore is supported:

## Schema Change Safety
- How incompatible changes are detected:
- How ingestion is stopped:
- How warnings/failures are surfaced:

## Validation Parity
- Which source system validations were mirrored downstream:
- How failures are checked and reported:

## Catalog Exposure
Explain how lake and warehouse datasets are published and discovered.

## Validation
List the commands or workflows you ran to validate the solution.

## Known Limitations / Next Steps
List tradeoffs or improvements you would make with more time.

## Responsible AI Usage
- Did you use AI tools?
- Where did they help?
- What did you personally verify or correct?

## Author Checklist
- [ ] Linting passes
- [ ] Tests pass
- [ ] Model validation/tests pass
- [ ] Schema compatibility checks pass
- [ ] Data quality validations pass
- [ ] Catalog metadata validation passes
- [ ] README was tested from a clean setup
- [ ] End-to-end CDC flow was validated locally
