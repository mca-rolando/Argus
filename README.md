# ARGUS

ARGUS is an open-source, centralized observability platform designed for
multi-site UniFi environments.

It provides a clear operational view of UniFi consoles, network devices,
applications, WAN connections, VPN tunnels, incidents, and configuration
snapshots across approximately 100 sites.

ARGUS is designed for Network Operations Center displays, desktop browsers,
and Raspberry Pi kiosk deployments running at resolutions of 1920×1080 or
lower.

## Project Status

ARGUS is currently in its initial implementation phase.

The architecture and functional baseline has been completed and is available
in both PDF and DOCX formats:

- [Architecture and Functional Specification v1.0 — PDF](docs/architecture/v1.0/specification/ARGUS_Architecture_Functional_Specification_v1.0.pdf)
- [Architecture and Functional Specification v1.0 — DOCX](docs/architecture/v1.0/specification/ARGUS_Architecture_Functional_Specification_v1.0.docx)
- [Architecture diagrams](docs/architecture/v1.0/diagrams/)
- [UI storyboards](docs/architecture/v1.0/storyboards/)

## Core Principles

- Read-only inspection of UniFi environments.
- Outbound-only HTTPS communication from agents to the ARGUS server.
- No inbound connectivity required to monitored consoles.
- Separation between device discovery and active monitoring.
- Site-centered operational model.
- Current-state monitoring with historical events and configuration snapshots.
- Clear health hierarchy for sites, consoles, applications, devices, WANs,
  and VPNs.
- Kiosk-first interface optimized for clarity and rapid incident recognition.
- No dependency on browser automation, GUI scraping, or direct modification
  of UniFi configuration.

## Planned Components

| Component | Purpose | Planned technology |
|---|---|---|
| `argus-api` | REST API, enrollment, inventory, health and administration | Python / FastAPI |
| `argus-web` | Administration, NOC and kiosk user interfaces | React / TypeScript |
| `argus-worker` | Health evaluation, correlation, notifications and retention | Python |
| `argus-agent` | Local read-only inspection of UDM, UCG and UDR consoles | Python |
| PostgreSQL | Persistent operational and historical data | PostgreSQL |
| Redis | Optional queues, caching and transient coordination | Redis |
| NGINX | TLS termination, reverse proxy and static UI delivery | NGINX |

## Repository Structure

```text
apps/
├── api/                    ARGUS REST API
├── web/                    React/TypeScript web interface
└── worker/                 Background processing service

agent/                      UniFi console agent
packages/                   Shared libraries
deployment/                 Docker, NGINX and systemd resources
docs/                       Architecture, operations and security documentation
scripts/                    Development and operational utilities
tests/                      Unit, integration and end-to-end tests
.github/workflows/          Continuous integration workflows
```

## Initial Scope

ARGUS v1 will monitor and report:

- UniFi consoles and installed applications.
- Network switches and wireless access points.
- UniFi Protect cameras and related devices.
- WAN availability and quality.
- Site-to-site and remote-access VPN health.
- Device adoption, connectivity and operational state.
- Network configuration summaries and historical snapshots.
- Incidents, events and recovery transitions.
- Agent enrollment, approval, heartbeat and update status.

ARGUS v1 will not modify UniFi configuration or replace UniFi Site Manager.

## Security Model

Agents use locally stored credentials with the minimum permissions necessary
for read-only inspection. Credentials remain on the monitored console and are
never transmitted to the ARGUS server.

Agents initiate outbound HTTPS connections to report normalized telemetry.
The central server does not initiate inbound connections to monitored sites.

Secrets, API keys, passwords, private keys, production configuration files,
and environment files must never be committed to this repository.

## Documentation

The formal v1.0 baseline contains:

- 18 architecture and operational diagrams.
- 22 conceptual UI storyboards.
- System context and deployment topology.
- Server and agent component models.
- Enrollment, discovery and health sequences.
- Data model and incident lifecycle.
- VPN health and security boundaries.
- Retention, update and offline-recovery models.
- NOC, administration, site and kiosk interface concepts.

## License

ARGUS is released under the MIT License.

Copyright © 2026 Rolando Blanco.
