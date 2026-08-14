# ARGUS Architecture Documentation

This directory contains the formal architecture and functional-design
materials for ARGUS.

## Current Baseline

The current implementation baseline is version 1.0, dated August 12, 2026.

- [Architecture and Functional Specification v1.0 — PDF](v1.0/specification/ARGUS_Architecture_Functional_Specification_v1.0.pdf)
- [Architecture and Functional Specification v1.0 — DOCX](v1.0/specification/ARGUS_Architecture_Functional_Specification_v1.0.docx)
- [Architecture and operational diagrams](v1.0/diagrams/)
- [Conceptual UI storyboards](v1.0/storyboards/)

## Baseline Contents

Version 1.0 contains:

- 18 architecture and operational diagrams.
- 22 conceptual user-interface storyboards.
- Central server and UniFi agent component models.
- Enrollment, discovery, health and recovery sequences.
- Operational data model and incident lifecycle.
- WAN and VPN health models.
- Security boundaries and credential-handling principles.
- Retention, agent-update and configuration-snapshot models.
- Administration, NOC, site-detail and kiosk interface concepts.

## Directory Structure

```text
architecture/
├── README.md
└── v1.0/
    ├── README.txt
    ├── specification/
    │   ├── ARGUS_Architecture_Functional_Specification_v1.0.docx
    │   └── ARGUS_Architecture_Functional_Specification_v1.0.pdf
    ├── diagrams/
    │   └── D01 through D18
    └── storyboards/
        └── S01 through S22
```

## Change Management

Do not modify a published architecture baseline in a way that changes its
meaning without recording the change.

Material architectural changes should be handled by one of the following:

- A corrected revision with an explicit change record.
- A new architecture baseline version.
- An Architecture Decision Record when the change is narrower than a complete
  baseline revision.

Implementation code and operational documentation should reference the
architecture version on which they are based.
