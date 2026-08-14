# ARGUS Documentation

This directory contains the technical, architectural, operational, and
security documentation for ARGUS.

## Current Baseline

The current formal baseline is ARGUS Architecture and Functional Specification
v1.0.

- [Architecture documentation](architecture/README.md)
- [Specification v1.0 — PDF](architecture/v1.0/specification/ARGUS_Architecture_Functional_Specification_v1.0.pdf)
- [Specification v1.0 — DOCX](architecture/v1.0/specification/ARGUS_Architecture_Functional_Specification_v1.0.docx)
- [Architecture diagrams](architecture/v1.0/diagrams/)
- [UI storyboards](architecture/v1.0/storyboards/)

## Documentation Areas

| Directory | Purpose | Status |
|---|---|---|
| `architecture/` | Architecture, functional specification, diagrams and storyboards | Baseline v1.0 available |
| `operations/` | Installation, administration, backup, recovery and troubleshooting | Planned |
| `security/` | Threat model, hardening, credential handling and security operations | Planned |

## Documentation Principles

- Documentation must reflect implemented behavior.
- Architectural changes must update the relevant specification or decision
  record.
- Operational procedures must include prerequisites, validation, rollback and
  recovery considerations.
- Examples must never contain production credentials, tokens, private keys,
  customer information, or sensitive infrastructure data.
- Screens and diagrams should remain readable at 1920×1080 or lower whenever
  they represent the NOC or kiosk interface.

## Versioned Architecture

Architecture baselines are stored in versioned directories under
`docs/architecture/` so that implementation decisions can be traced to the
specification that introduced them.

The current version is `v1.0`.
