# AGENTS.md — ARGUS Engineering Rules

This file governs all automated and human-assisted engineering work in the
ARGUS repository. These rules apply repository-wide unless a more specific
nested `AGENTS.md` adds stricter requirements. A nested file may not weaken
security, isolation, appliance safety, read-only monitoring, testing, or
approval requirements defined here.

## 1. Project Mission

ARGUS is a centralized, read-only observability platform for distributed
UniFi environments.

ARGUS collects and presents normalized operational state for:

- UniFi consoles and installed applications.
- Gateways, switches, access points, cameras, and approved monitored devices.
- WAN interfaces and failover state.
- Site-to-site and remote-access VPN tunnels.
- Agent availability and version state.
- Non-secret network configuration summaries.
- Events, alerts, incidents, maintenance windows, and audit records.

ARGUS must help operators answer:

- Where is the problem?
- What is failing?
- Since when?
- What is affected?
- Is the information current or stale?

The primary operational unit is a Site within an Organization.

## 2. Non-Goals and Invariants

The following are non-negotiable:

- ARGUS telemetry operation is strictly read-only with respect to UniFi and the
  monitored appliance.
- ARGUS must never modify UniFi Network, Protect, console, device, WAN, VPN,
  firewall, WiFi, routing, user, credential, or other native configuration.
- Native UniFi routing, firewall, VPN, Network, Protect, and console operation
  always take priority over ARGUS telemetry.
- If ARGUS must choose between collecting telemetry and preserving normal
  console operation, ARGUS must stop, defer, shed, or reduce telemetry.
- ARGUS telemetry collection must not modify operating-system state.
- Explicitly authorized agent installation, update, repair, rollback, and
  removal operations may modify only narrowly scoped ARGUS-owned
  operating-system resources.
- Agent lifecycle privileges must never be available to or reused by telemetry
  collectors.
- ARGUS failure must never cause console instability or interfere with native
  UniFi services.
- Do not implement generic UniFi or operating-system command execution.
- Do not add configuration-write endpoints, browser automation, or GUI
  scraping.
- Do not ingest Protect video, recordings, or thumbnails in V1.
- Do not collect WiFi passwords, VPN PSKs, private keys, RADIUS secrets,
  administrative passwords, session cookies, or equivalent secrets.
- Site agents initiate outbound HTTPS connections only.
- The ARGUS server must not require inbound access to monitored sites.
- PostgreSQL is the authoritative persistent datastore.
- Redis, when used, is transient coordination infrastructure and must not
  become the sole durable source of operational truth.
- Unknown state must not be represented as confirmed offline state.
- Monitoring exclusion or administrative disablement must not be represented
  as a failure.
- ARGUS must never fill or materially endanger console storage.
- Telemetry history must be sacrificed according to an explicit overflow
  policy when necessary to preserve appliance operation.

Read-only local shell commands may be used by the ARGUS Agent for telemetry
collection when no appropriate API exists. Such commands must be documented,
strictly bounded, sanitized, allowlisted where practical, tested, executed
without a shell when practical, and proven not to modify UniFi or
operating-system state.

Configuration-changing, state-changing, unrestricted, or server-supplied shell
operations remain prohibited.

Any request that conflicts with these invariants must be stopped and raised to
the user.

## 3. Authority and Precedence

Architecture/product authority and engineering/process authority are separate.

### 3.1 Architecture and Product Precedence

When determining what ARGUS should do, use this precedence:

1. Explicit approved user requirement.
2. Accepted Architecture Decision Records.
3. Current versioned architecture specification.
4. Other repository documentation.
5. Existing implementation.

An explicit approved user requirement may supersede prior product
documentation. Material architectural changes must still be recorded in an ADR
or a new architecture baseline as appropriate.

Existing implementation is evidence of current behavior, not automatically the
correct architecture.

### 3.2 Engineering and Process Precedence

When determining how work may be performed, use this precedence:

1. Explicit user instruction.
2. Applicable `AGENTS.md` files.
3. Repository tooling and conventions.

User instructions may authorize a task or change its scope, but they do not
silently waive destructive-action safeguards, credential protections, tenant
isolation, appliance safety, the read-only UniFi invariant, or the prohibition
against force-pushing.

Material conflicts must be reported before implementation.

## 4. Authoritative Architecture

The authoritative design sources are:

- `README.md`
- `SECURITY.md`
- `docs/architecture/v1.0/specification/`
- `docs/architecture/v1.0/diagrams/`
- Accepted ADRs under `docs/architecture/decisions/`

The initial runtime architecture is:

- Python/FastAPI backend API.
- Python background worker.
- React/TypeScript frontend.
- Python ARGUS Agent.
- PostgreSQL persistent datastore.
- Optional Redis for queues, caching, scheduling, and transient coordination.
- NGINX for TLS termination, reverse proxying, security headers, and static
  frontend delivery.
- Docker Compose on Ubuntu Server 26.04 for the initial server deployment.
- A directly installed, appliance-safe ARGUS Agent on supported UniFi
  UDM, UDM Pro, UDM SE, UCG, UDR, and related console families.

Do not silently change a required technology, deployment model, or service
boundary. Propose an ADR first.

## 5. Repository Structure and Ownership

Expected ownership is:

- `apps/api/` — FastAPI application, agent and administrative APIs.
- `apps/worker/` — asynchronous processing, health evaluation, incident
  correlation, notifications, retention, and rollups.
- `apps/web/` — React/TypeScript administrative and NOC interfaces.
- `agent/` — locally installed UniFi console agent.
- `packages/python-common/` — deliberately shared Python contracts and
  utilities that do not create hidden service coupling.
- `deployment/docker/` — server container images and container resources.
- `deployment/nginx/` — reverse-proxy and TLS configuration.
- `deployment/systemd/` — approved service definitions where applicable.
- `docs/` — versioned architecture, ADRs, security, compatibility evidence,
  and operational guidance.
- `scripts/` — development and operational utilities.
- `tests/unit/` — isolated unit tests.
- `tests/integration/` — database, API, worker, and component integration tests.
- `tests/e2e/` — complete user and agent workflows.
- `.github/workflows/` — continuous-integration policy.

Database migrations must have one clearly documented owner, normally the API
package. Shared API schemas must also have one authoritative owner.

Do not duplicate domain models independently across services. Do not create a
generic shared package that couples unrelated runtime internals.

## 6. Backend Standards

Backend code must:

- Use supported Python and FastAPI versions pinned by repository tooling.
- Use complete type annotations for public functions, service boundaries, and
  data models.
- Use explicit request and response schemas.
- Validate all external input at the boundary.
- Separate transport, application, domain, and persistence concerns.
- Use dependency injection for database sessions, authentication, clocks, and
  external integrations where it improves testability.
- Use timezone-aware UTC timestamps internally.
- Avoid blocking I/O in asynchronous request handlers.
- Bound request bodies, batches, queries, retries, and pagination.
- Make agent ingestion endpoints idempotent where practical.
- Return stable machine-readable error codes in addition to safe messages.
- Avoid exposing stack traces, SQL details, secrets, or internal network
  information to clients.
- Include liveness and readiness checks with different semantics.
- Preserve database backpressure instead of acknowledging data that was not
  durably accepted.

Health evaluation and incident correlation must be deterministic and testable
without HTTP or a live worker.

Do not place significant domain rules directly in route handlers.

## 7. Frontend Standards

Frontend code must use React and TypeScript with strict type checking.

The frontend must:

- Consume documented API contracts rather than infer server payloads.
- Treat all server data as untrusted input.
- Distinguish loading, stale, empty, unavailable, unauthorized, and failed
  states.
- Display the age or freshness of operational data.
- Preserve the health ordering:
  `CRITICAL → WARNING → DEGRADED → UNKNOWN → HEALTHY → DISABLED`.
- Never use color as the only indicator of status.
- Pair health colors with text, icons, symbols, or another accessible cue.
- Ensure no important action or information depends exclusively on hover.
- Support keyboard operation and visible focus.
- Use semantic HTML and accessible names.
- Avoid rendering secrets, complete credentials, or sensitive token material.
- Avoid storing long-lived credentials in browser local storage unless an
  accepted ADR explicitly authorizes and mitigates it.

The `/noc` presentation is a first-class interface:

- It must fit comfortably at 1920×1080 without horizontal or vertical scrolling
  during normal wallboard operation.
- It must remain operational at 1600×900 and 1366×768 using responsive density.
- It must show persistent live/stale and last-updated information.
- It must not require hover.
- It must use restrained animation suitable for Raspberry Pi Chromium.
- It must be tested at all required viewport sizes.
- Critical status and primary site labels must remain readable at distance.

Administrative UI components may be shared with `/noc`, but `/noc` must not be
implemented as an overcrowded administrative page.

## 8. ARGUS Agent Standards

The ARGUS Agent runs directly on production UniFi network infrastructure,
including UDM, UDM Pro, UDM SE, UCG, UDR, and related supported console
families. It must follow the Appliance-Safe Agent principle.

### 8.1 Appliance-Safe Agent Principle

Native console operation always has priority over ARGUS.

The agent must:

- Yield, defer, throttle, shed, or stop telemetry under resource pressure.
- Fail independently without destabilizing the console.
- Never delay or interfere with routing, firewall, VPN, Network, Protect,
  console management, or other native services.
- Consume minimal and explicitly bounded CPU, RAM, process and thread count,
  disk space, disk-write rate, network bandwidth, and concurrent operations.
- Use resource budgets validated experimentally for each supported console
  family and relevant UniFi OS version.
- Never invent or apply one universal resource limit to every console family.
- Use jitter and staggered scheduling for recurring collectors.
- Use boot-time jitter before reconnecting or starting collection.
- Bound restart and watchdog behavior with backoff.
- Never enter an infinite or high-frequency crash/restart loop.
- Minimize disk writes to protect storage capacity and reduce unnecessary wear.
- Preserve normal console operation even if telemetry must be lost.

Resource exhaustion, repeated collector failure, spool pressure, low disk
space, native-service pressure, or an unstable restart cycle must cause
telemetry reduction or shutdown rather than aggressive recovery.

### 8.2 Core Agent Behavior

The agent must remain small, capability-driven, and self-supervising.

Agent code must:

- Operate without inbound listening ports.
- Initiate outbound HTTPS only.
- Verify the server hostname and certificate.
- Use a unique credential per agent.
- Store credentials with restrictive local filesystem permissions.
- Preserve identity and unacknowledged spool data across ordinary reboot and
  agent upgrade.
- Add jitter to all recurring work.
- Bound concurrency, retries, memory, disk consumption, and probe frequency.
- Apply exponential backoff with jitter when the server is unavailable.
- Detect supported UniFi OS, Network, and Protect capabilities.
- Disable unsupported collectors without failing the entire site.
- Keep discovery separate from approval and active monitoring.
- Collect only approved, non-secret data.
- Never execute server-supplied arbitrary shell commands.
- Never accept an unrestricted URL, host, path, command, or command argument
  as desired state.
- Validate and constrain all probe targets to prevent SSRF and network abuse.
- Report agent and schema versions with each applicable exchange.

Collectors must produce normalized domain records. They must not directly
control transport acknowledgements or delete spool records.

### 8.3 Reboot, Persistence, and UniFi OS Upgrades

The agent must:

- Recover automatically after ordinary supported console reboots.
- Start automatically after boot using an evidenced, platform-compatible
  mechanism.
- Use boot-time jitter to prevent synchronized reconnect storms when many sites
  reboot together.
- Preserve installation persistence across supported UniFi OS upgrades where
  technically possible.
- Detect when an upgrade removes, disables, or invalidates ARGUS components.
- Recover safely through a documented repair path.
- Report loss or degradation of persistence when it can do so safely.
- Preserve agent identity and unacknowledged spool state during ordinary
  reboots and supported agent upgrades.
- Keep persistent state separate from replaceable application code where the
  platform permits.

Never assume that a filesystem path, systemd unit, startup mechanism,
container mechanism, package location, or persistence technique survives a
UniFi OS upgrade.

Every persistence and recovery claim must satisfy the UniFi Evidence Rule for
the applicable platform and version.

If supported persistence cannot be established safely for a platform/version,
that combination must be labeled experimental or unsupported rather than
silently using an unverified mechanism.

### 8.4 Installation and Lifecycle Operations

Installation, update, repair, rollback, and removal are privileged lifecycle
operations distinct from telemetry collection.

The ARGUS Agent should be self-contained whenever technically practical.
Agent installation must not modify the UniFi OS package database, system
Python environment, global language runtimes, or unrelated system libraries
unless an accepted platform-specific ADR and compatibility evidence explicitly
require and authorize it. Prefer vendored, isolated, or ARGUS-owned runtime
dependencies over system-wide dependency installation.

Lifecycle operations may modify only documented ARGUS-owned resources after
explicit task authorization. They must:

- Be narrowly scoped.
- Be idempotent where practical.
- Avoid changing native UniFi configuration.
- Never restart, reload, reconfigure, disable, replace, or interfere with
  Network, Protect, routing, firewall, VPN, console management, or other native
  services.
- Never repurpose native UniFi service files or configuration.
- Never leave unrestricted credentials, permissions, startup hooks, files,
  processes, or network listeners.
- Use atomic or recoverable upgrade behavior.
- Support safe rollback.
- Preserve identity and unacknowledged spool state.
- Cleanly remove only ARGUS-owned resources during uninstall.
- Preserve evidence or state explicitly designated for backup or recovery.
- Verify successful startup without creating an unbounded retry loop.

Privileges or mechanisms available to lifecycle tooling must not be exposed to,
imported by, or reused by telemetry collectors.

### 8.5 Local Shell Telemetry Collection

A collector may use a local shell command only when:

- No suitable local UniFi or operating-system API exists.
- The telemetry requirement is documented.
- The exact command, executable, arguments, expected output, timeout, and
  supported platform are documented.
- Arguments are fixed or derived from strictly validated values.
- Input is not concatenated into a shell command string.
- Direct process execution without a shell is used whenever practical.
- Execution time, output size, frequency, permissions, process count, CPU,
  memory, and concurrency are bounded.
- Output parsing is defensive and tested with malformed output.
- Sensitive output is filtered before logging, spooling, or transmission.
- Tests prove that the command does not modify UniFi or operating-system state.
- Failure degrades the collector to unavailable or unknown without destabilizing
  the whole agent.

The agent must not use telemetry commands for:

- UniFi configuration changes.
- Package installation or removal.
- Service enablement, disablement, restart, or reload.
- File modification.
- Firewall, route, interface, VPN, user, permission, or credential changes.
- Arbitrary command execution requested by the server.
- Privilege escalation beyond an explicitly documented read-only requirement.

A future agent rewrite in another language requires an ADR.

## 9. Database and Migration Rules

PostgreSQL is the authoritative persistent datastore.

All schema changes must:

- Be represented by versioned migrations.
- Be reviewed with the code that consumes them.
- Include rollback or forward-recovery guidance.
- Avoid destructive changes without an explicit migration plan.
- Preserve tenant isolation and referential integrity.
- Define required indexes and uniqueness constraints.
- Use database transactions for enrollment, token consumption, identity
  issuance, state transitions, and other atomic workflows.
- Store UTC timestamps using timezone-aware types.
- Define retention behavior separately from transactional correctness.
- Avoid using Redis or application memory as the only durable record.

Migration rules:

- Never edit an already released migration to change its meaning.
- Create a new corrective migration.
- Never run migrations against a shared, staging, or production database
  without explicit user approval.
- Never drop a table, column, constraint, or retained data without explicit
  approval and a recovery plan.
- Test both clean-database migration and upgrade from the previous supported
  schema.
- Backfills must be resumable, observable, and bounded.

Migrations against disposable, task-local test databases are normal development
operations once the task and branch are authorized.

Polymorphic references such as `entity_type` plus `entity_id` require an
accepted ADR explaining integrity enforcement, indexing, deletion behavior,
and tenant validation.

Audit records described as immutable must have technical enforcement. A normal
mutable table alone does not satisfy this requirement.

## 10. API Contracts and Schema Versioning

All public, frontend-facing, and agent-facing APIs must be versioned.

Rules:

- Use `/api/v1/` for the initial HTTP API.
- Maintain an authoritative OpenAPI definition generated from or verified
  against implementation.
- Every agent payload must identify its schema version.
- Agent payloads must include stable entity identifiers, source timestamps,
  and sequence information where ordering matters.
- Define compatibility policy before supporting more than one agent schema.
- Reject unsupported schema versions with a stable error code and upgrade
  guidance.
- Additive fields must not change existing field meaning.
- Breaking semantic changes require a new API or schema version.
- IDs, timestamps, enums, nullability, pagination, errors, and idempotency must
  be explicitly specified.
- Frontend and agent clients must not depend on undocumented response fields.
- Contract fixtures and compatibility tests are required.

API versioning does not replace database migration versioning or agent release
versioning.

## 11. Authentication, Enrollment, and Credential Security

Credential classes must remain separate:

- Administrative user sessions.
- One-time agent enrollment tokens.
- Long-lived per-agent credentials.
- Integration API keys.
- Future mTLS certificates.
- Lifecycle installation or update authorization.

Do not reuse one credential class for another purpose.

Enrollment must:

- Use a cryptographically random, high-entropy token.
- Have a short, explicit expiration.
- Have an activation limit, normally one.
- Consume usage transactionally.
- Resist concurrent double use.
- Be rate limited.
- Be audited without logging token material.
- Place the new agent in a restricted pending state.
- Return a unique agent identity and credential only after valid enrollment.
- Define what data a pending agent may submit and what it may retrieve.

Credentials and API keys must:

- Be displayed in full only when initially issued when applicable.
- Be stored server-side as a secure hash when plaintext recovery is unnecessary.
- Support independent rotation and revocation.
- Have explicit scopes and tenant ownership.
- Never appear in URLs.
- Never be logged.
- Use constant-time verification where applicable.
- Have issuance, rotation, revocation, and failed-use audit events.

Administrative authentication, session handling, MFA, break-glass access,
password hashing, CSRF protection, and OIDC/SAML integration require an ADR
before that functionality is implemented.

Do not invent a fingerprint trust model for console ownership. Document and
approve the proof-of-possession model first.

## 12. Multi-Tenant Organization and Site Isolation

Organization is the top-level tenant boundary. Site is the primary operational
unit within an organization.

Every tenant-owned record must have an explicit, enforceable ownership path to
an organization.

Requirements:

- Derive authorization scope from authenticated identity, not request claims
  alone.
- Apply organization and site filtering in the service and persistence layers.
- Never trust a client-supplied `organization_id` to grant access.
- Validate related objects belong to the same organization before linking them.
- Include negative cross-tenant tests for every resource type.
- Background jobs, caches, exports, audit queries, notifications, and search
  must preserve tenant boundaries.
- Cache keys and queue messages must include tenant context where applicable.
- Globally unique identifiers do not replace authorization checks.
- Super-admin cross-tenant access must be explicit, minimal, and audited.

A new endpoint is incomplete until object-level authorization and cross-tenant
tests exist.

## 13. Logging and Secret Redaction

Use structured logging with stable event names and correlation identifiers.

Logs may include:

- Request or operation identifiers.
- Agent, organization, site, console, and entity IDs when policy permits.
- Safe state transitions.
- Durations and bounded counts.
- Safe error and resource-pressure categories.

Logs must not include:

- Passwords.
- Enrollment tokens.
- Agent credentials.
- API keys.
- Authorization or cookie headers.
- Private keys or certificate private material.
- WiFi or VPN secrets.
- RADIUS secrets.
- UniFi session cookies.
- Complete sensitive request or response bodies.
- Raw configuration exports.
- Database connection strings containing credentials.

Redaction must occur before serialization and output. Do not rely only on log
viewer masking.

Exception reporting, lifecycle diagnostics, local command diagnostics, and
telemetry parsing errors must use the same redaction rules. Redaction behavior
requires tests containing realistic secret-shaped values.

Audit records and diagnostic logs are different data classes. Do not substitute
one for the other.

## 14. UniFi Integration Rules

ARGUS must prefer official local UniFi APIs.

Collectors must:

- Be capability-driven and version-aware.
- Record relevant UniFi OS, Network, and Protect versions.
- Isolate vendor-specific payload parsing behind adapters.
- Normalize data before it reaches core health logic.
- Preserve fixture examples for each supported version.
- Treat absent or unsupported fields as unavailable, not as failure.
- Apply explicit timeouts, TLS policy, retries, and response-size bounds.
- Never scrape the GUI or automate a browser.
- Never use write endpoints.
- Never send configuration-changing methods or payloads.
- Never collect secrets merely because an endpoint or local command returns
  them.
- Redact sensitive fields before logging, spooling, or transmission.
- Use local shell telemetry collection only under Section 8.5.
- Respect appliance resource budgets and native-service priority.

Site Manager and SNMP are optional secondary sources and must not silently
become dependencies for primary local site health.

### 14.1 UniFi Evidence Rule

Never claim support for a UniFi version, model, endpoint, field, command,
filesystem path, startup mechanism, persistence mechanism, container behavior,
systemd behavior, upgrade behavior, resource budget, or API behavior without
at least one of:

- Official documentation applicable to that version or model.
- Approved lab evidence recorded in the repository.
- A repository compatibility fixture and test derived from approved evidence.

Behavior observed on one UniFi OS, Network, Protect, console, device, or API
version must not automatically be generalized to another version.

Each compatibility claim must identify its evidence and applicable version
range. Unverified behavior must be labeled experimental, unknown, or
unsupported.

Undocumented Protect interfaces require explicit risk documentation, isolated
adapters, approved lab evidence, compatibility fixtures, and an ADR before
production use.

Tests or mocks invented solely from assumptions are not evidence of actual
UniFi support.

## 15. Offline Spool and Replay

The agent spool must be durable, bounded, ordered, recoverable, and
appliance-safe.

Required properties:

- Sequence numbers are monotonic within a documented identity or epoch.
- Records are deleted only after authoritative server acknowledgement.
- Retransmission is safe and idempotent.
- Partial batch acceptance has defined semantics.
- Agent restart does not lose unacknowledged records.
- Ordinary console reboot does not lose agent identity or unacknowledged
  records.
- Supported agent upgrades preserve identity and unacknowledged records.
- Server restart does not require agent re-enrollment.
- Spool corruption has detectable and documented recovery behavior.
- Hard disk-space and time-retention bounds are configurable and enforced.
- Disk-write rates are minimized and explicitly bounded.
- Writes are batched or coalesced where safe.
- Overflow policy is explicit, deterministic, observable, and tested.
- Telemetry history is discarded according to that policy before console
  storage is endangered.
- Reserved free-space or low-disk safeguards are platform-specific and
  experimentally validated.
- Credentials and prohibited secrets must never enter the spool.
- Clock skew must not replace sequence-based ordering.
- Replay must not create duplicate events or reopen closed incidents
  incorrectly.
- Backpressure must not be reported as successful ingestion.
- Replay concurrency and bandwidth must be bounded.
- Reconnect replay must not starve current health reporting or native console
  workloads.

The agent must never fill console storage. If safe persistence cannot continue,
the agent must shed telemetry, record a bounded local diagnostic when safe, and
preserve console operation.

The spool format, ACK protocol, sequence reset behavior, resource bounds,
disk-write policy, overflow policy, corruption recovery, and agent identity
recovery require accepted ADRs before Phase 0 implementation.

## 16. Testing Requirements

Every behavioral change requires proportionate automated tests.

Minimum expectations:

- Unit tests for domain rules, parsing, redaction, health evaluation, and
  correlation.
- Integration tests for PostgreSQL transactions, migrations, API routes,
  authentication, authorization, worker processing, and replay.
- Contract tests for agent and frontend schemas.
- End-to-end tests for critical operator and enrollment workflows.
- Negative tests for malformed input, unauthorized access, expired or revoked
  credentials, replay, and cross-tenant access.
- Security regression tests for every corrected vulnerability.
- Deterministic UniFi fixtures or simulators; tests must not depend on customer
  consoles.
- Migration tests from the previous supported database version.
- Load tests for approximately 100 jittered agents before production rollout.
- NOC viewport tests at 1920×1080, 1600×900, and 1366×768.
- Accessibility tests ensuring status is not color-only and critical behavior
  is not hover-only.
- Offline and reconnect tests proving ordered, duplicate-safe replay.
- Tests proving excluded devices do not alert.
- Tests proving agent loss yields UNKNOWN for dependent state unless independent
  evidence confirms an outage.
- Tests proving maintenance suppresses notifications without deleting events.
- Tests for every allowed local shell collector, including timeouts, malformed
  output, output bounds, sanitization, and absence of state changes.

Appliance-safety testing must include:

- Experimentally validated CPU, RAM, process/thread, disk, disk-write,
  bandwidth, and concurrency budgets for every supported console family.
- Normal and stressed native-service conditions.
- Collector throttling, shedding, and shutdown under resource pressure.
- Low-disk and spool-overflow behavior.
- Repeated network failure and reconnect behavior.
- Bounded watchdog restart and crash-loop prevention.
- Boot-time jitter and fleet reconnect-storm simulation.
- Ordinary reboot and automatic-start recovery.
- Supported UniFi OS upgrade survival or documented repair recovery.
- Detection of removed or disabled ARGUS components.
- Atomic or recoverable agent upgrade and rollback.
- Identity and spool preservation across reboot and agent upgrade.
- Clean uninstall limited to ARGUS-owned resources.
- Proof that install, update, repair, rollback, and uninstall do not restart,
  reconfigure, or interfere with native UniFi services.
- Proof that lifecycle privileges are unavailable to telemetry collectors.
- Validation that agent dependencies are self-contained or ARGUS-owned and do
  not alter the UniFi OS package database, system Python, global runtimes, or
  unrelated system libraries unless an accepted platform-specific ADR permits
  the exception.
- Failure injection demonstrating that agent failure does not destabilize the
  console.

Resource tests must use approved lab hardware or evidence appropriate to the
specific console family and version. Desktop or generic Linux results do not
establish appliance safety.

Tests must not use real credentials or production data.

Never weaken authentication, TLS verification, authorization, validation,
redaction, tenant isolation, resource bounds, or another security or
appliance-safety control to make a test pass. Fix the test or implementation
correctly.

Do not claim tests passed unless they were actually run and their result was
observed.

## 17. Security Requirements

Use secure defaults and least privilege.

All changes must consider:

- Authentication and authorization.
- Organization and site isolation.
- Input validation and injection.
- Replay and idempotency.
- SSRF and probe-target abuse.
- Secret handling and redaction.
- Dependency and supply-chain risk.
- Rate limiting and resource bounds.
- Audit coverage.
- Update signing and rollback protection.
- Database and backup confidentiality.
- Browser security headers, CSRF, XSS, and session protection.
- Safe failure and error disclosure.
- Local process execution and command-output handling.
- Separation of lifecycle privileges from telemetry privileges.
- Appliance stability and native-service priority.

Additional rules:

- Do not disable TLS verification outside an explicitly isolated test fixture.
- Do not add permissive CORS as a troubleshooting shortcut.
- Do not use wildcard authorization scopes.
- Do not use shell invocation when a safe API or direct executable call exists.
- Do not interpolate external input into SQL, shell commands, paths, templates,
  or URLs without appropriate safe handling.
- Pin dependencies and commit lock files.
- Run relevant static analysis, dependency scanning, and secret scanning.
- Security-sensitive behavior requires audit events and negative tests.
- Agent updates must be signed or cryptographically verified, atomic or
  recoverable, and rollback-capable before automatic updating is enabled.
- Telemetry code must run with less privilege than lifecycle tooling whenever
  the platform permits.
- Lifecycle authorization, binaries, functions, and interfaces must not be
  callable through normal telemetry or server desired-state messages.

If a secure or appliance-safe implementation depends on an unresolved
architectural choice, stop and request an ADR rather than selecting a weaker
shortcut.

## 18. Docker and Deployment Requirements

The initial supported server deployment is Docker Compose on Ubuntu Server
26.04. This does not imply that Docker or another container mechanism is
supported or persistent on any UniFi console.

### 18.1 Server Containers

Server container requirements:

- Use pinned base-image versions or immutable digests according to project
  policy.
- Use minimal runtime images.
- Run as non-root whenever technically possible.
- Do not bake credentials or environment-specific configuration into images.
- Use explicit health checks.
- Use read-only filesystems and dropped Linux capabilities where practical.
- Separate build and runtime stages.
- Keep PostgreSQL data outside ephemeral container filesystems.
- Do not expose PostgreSQL or Redis publicly.
- Expose only required NGINX ports.
- Keep development conveniences out of production Compose profiles.
- Use restart behavior that does not hide persistent crash loops.
- Log safe operational state without secrets.
- Document resource limits and persistent volumes.

Server deployment requirements:

- NGINX terminates TLS and applies approved security headers.
- Public ingress is HTTPS only, except explicitly required ACME handling.
- Backups must be encrypted and periodically restore-tested.
- Deployment must include readiness, liveness, and external self-monitoring.
- Production configuration must not be committed.
- Provide safe `.env.example` files containing names and placeholders only.
- Deployment and rollback procedures must be documented and tested.

### 18.2 UniFi Agent Deployment

Agent deployment is governed by ADR-0006 and platform-specific evidence.

The agent should be self-contained whenever technically practical. Prefer
vendored, isolated, or ARGUS-owned runtime dependencies. Installation must not
modify the UniFi OS package database, system Python environment, global
language runtimes, or unrelated system libraries unless an accepted
platform-specific ADR and compatibility evidence explicitly authorize the
exception.

Installation resources must:

- Be limited to documented ARGUS-owned files, directories, identities,
  permissions, processes, service definitions, startup hooks, dependencies,
  runtimes, and state.
- Use a persistence mechanism validated for the exact supported console family
  and UniFi OS range.
- Avoid assuming systemd, containers, filesystem locations, or startup hooks
  behave consistently across console families or upgrades.
- Start after boot with bounded delay and jitter.
- Use bounded watchdog and restart behavior with backoff.
- Detect missing or disabled ARGUS components after supported UniFi OS upgrades.
- Provide a safe, documented repair path.
- Keep replaceable code and dependencies separate from identity and spool data
  where supported.
- Support atomic or recoverable update and safe rollback.
- Uninstall cleanly without touching non-ARGUS resources.
- Never restart or reconfigure native UniFi services.

Do not deploy, restart shared services, operate on a real console, rotate
credentials, run non-test migrations, or alter infrastructure without explicit
user approval.

## 19. Task Authorization

A named implementation task is authorized when the user explicitly approves:

- The task and its intended scope.
- The dedicated non-`main` branch on which it will be performed.

Once both are approved, Codex may perform normal in-scope development
operations without requesting permission for every action, including:

- Read and inspect repository files.
- Modify, create, rename, or remove task-scoped workspace files when required
  by the approved implementation.
- Run formatters, linters, type checkers, tests, and builds.
- Use already-declared repository tooling.
- Run disposable local services and test containers.
- Run migrations against disposable task-local test databases.
- Generate expected task-scoped build, test, migration, or code-generation
  artifacts.
- Update relevant tests and documentation.
- Make other reversible workspace changes ordinarily required to complete the
  approved task.

Ordinary implementation authorization does not authorize agent lifecycle
operations on a real UniFi console. Installation, update, repair, rollback, or
removal on a real console requires explicit authorization identifying the
target environment and operation.

When a lifecycle task is explicitly authorized, modifications must remain
limited to documented ARGUS-owned resources. The agent must use isolated,
vendored, or ARGUS-owned dependencies unless an accepted platform-specific ADR
and compatibility evidence authorize a specific exception. Telemetry collectors
remain strictly read-only and receive no lifecycle authority.

Task authorization does not authorize:

- Work outside the approved scope.
- Work directly on `main`.
- Commit or push operations.
- Pull-request operations.
- Merge, tag, or release operations.
- Destructive Git operations.
- Production or shared environment access.
- Real UniFi console access.
- Real agent lifecycle operations unless separately authorized.
- Non-test database migrations.
- Infrastructure changes.
- Credential issuance, rotation, revocation, or exposure.
- External publication or notification.
- Weakening security or appliance-safety controls.
- Modification of UniFi or native operating-system resources.
- Modification of the UniFi OS package database, system Python, global
  runtimes, or unrelated libraries without an accepted platform-specific ADR
  and compatibility evidence.

If the required work materially exceeds the authorized task, Codex must stop
and request expanded authorization.

## 20. Git Workflow

Never work directly on `main`.

Before modifying files:

1. Read this file and relevant nested instructions.
2. Read relevant architecture, security, and operational documentation.
3. Inspect `git status`.
4. Confirm the current branch.
5. Identify existing user changes and preserve them.
6. Confirm the task and dedicated non-`main` branch are approved.
7. If branch creation or switching is needed, obtain explicit user approval.
8. Define validation appropriate to the task.

Once a task and branch are approved, ordinary task-scoped workspace edits and
validation do not require repeated approval.

Do not overwrite, discard, stage, or reformat unrelated user changes.

Do not use destructive Git commands to resolve a dirty working tree.

## 21. Branch Naming

Use lowercase, hyphen-separated branch names with a category prefix:

- `feat/<scope>-<description>`
- `fix/<scope>-<description>`
- `security/<scope>-<description>`
- `test/<scope>-<description>`
- `docs/<scope>-<description>`
- `refactor/<scope>-<description>`
- `chore/<scope>-<description>`
- `ci/<scope>-<description>`

Examples:

- `feat/agent-heartbeat`
- `security/enrollment-replay-protection`
- `fix/api-tenant-filtering`
- `docs/adr-agent-spool`

Do not create, switch, rename, merge, or delete branches without explicit user
approval.

## 22. Commit Conventions

Use clear conventional commit messages:

- `feat:`
- `fix:`
- `security:`
- `test:`
- `docs:`
- `refactor:`
- `chore:`
- `ci:`

A component scope is encouraged:

- `feat(agent): add authenticated heartbeat`
- `security(api): enforce organization ownership`
- `test(spool): cover duplicate replay`

Commits must be cohesive and must not include unrelated changes, generated
secrets, runtime data, local configuration, or customer information.

Never commit without explicit user approval.

Never push without explicit user approval.

Never force-push. User approval does not override this prohibition.

Do not amend, rebase, squash, cherry-pick, revert, merge, tag, or rewrite
history without explicit user approval.

## 23. Pull-Request Requirements

Every pull request must include:

- Purpose and operational impact.
- Related issue or design reference when applicable.
- Affected components.
- API and schema compatibility impact.
- Database migration and rollback impact.
- Authentication, authorization, tenancy, credential, and secret-handling
  impact.
- UniFi read-only, outbound-only, and appliance-safety impact.
- Agent resource-budget impact where applicable.
- Supported console/version evidence where applicable.
- Reboot, persistence, UniFi OS upgrade, rollback, and uninstall impact for
  lifecycle changes.
- Agent runtime and dependency-isolation impact for lifecycle changes.
- Test commands executed and observed results.
- Documentation changes.
- Deployment and rollback notes where applicable.
- Screenshots for material UI changes.
- NOC screenshots at required resolutions for `/noc` changes.
- An ADR link for material architectural decisions.
- Explicit disclosure of known limitations or deferred work.

Security-sensitive and appliance-sensitive pull requests must include abuse
cases, failure cases, and negative tests.

Do not open, update, merge, or close a remote pull request without explicit
user approval.

## 24. Definition of Done

A change is complete only when:

- It satisfies the approved task and acceptance criteria.
- It preserves UniFi read-only and outbound-only invariants.
- Native UniFi operation remains higher priority than telemetry.
- Telemetry does not modify UniFi or operating-system state.
- Lifecycle changes affect only documented ARGUS-owned resources.
- Agent dependencies are self-contained or ARGUS-owned whenever technically
  practical.
- Any exception that modifies a system package database, system Python, global
  runtime, or unrelated system library has an accepted platform-specific ADR
  and supporting compatibility evidence.
- Relevant code is formatted, linted, and type-checked.
- Appropriate unit, integration, contract, E2E, migration, security, UI, and
  appliance-safety tests pass.
- Authorization and cross-tenant negative tests exist where applicable.
- Secret-redaction tests exist where applicable.
- Local shell collectors have documentation, safety bounds, sanitization, and
  non-modification tests.
- UniFi compatibility claims satisfy the UniFi Evidence Rule.
- Agent resource budgets are evidenced per claimed console family/version.
- CPU, RAM, process/thread, disk, write-rate, bandwidth, and concurrency bounds
  are enforced where applicable.
- Restart behavior is bounded and cannot create an infinite crash loop.
- Recurring and boot-time scheduling use appropriate jitter.
- Spool disk/time bounds and overflow behavior are tested.
- Identity and unacknowledged spool preservation are tested across applicable
  reboot and agent-upgrade paths.
- Install, update, repair, rollback, and uninstall changes prove they do not
  interfere with native services.
- Upgrade and persistence claims are supported by applicable evidence.
- Failure isolation is tested.
- API and schema changes are documented and version-compatible.
- Database migrations and recovery behavior are verified.
- Logs and errors do not expose secrets or excessive internal detail.
- Operational failure modes and observability are addressed.
- Required documentation and ADRs are updated.
- No unrelated files or user changes were modified.
- The final report states exactly what changed and what validation ran.
- Any unrun validation or remaining limitation is disclosed.

Scaffolding, placeholders, TODO-only implementations, and documentation of
future behavior do not count as completed runtime functionality.

## 25. Documentation Requirements

Documentation must describe implemented behavior accurately.

Update documentation when changing:

- Public or agent APIs.
- Schema versions.
- Database schema or retention.
- Authentication, enrollment, credentials, or RBAC.
- Configuration.
- Deployment or operations.
- Supported UniFi versions, models, endpoints, commands, or capabilities.
- Agent installation, persistence, resource budgets, reboot recovery, UniFi OS
  upgrade behavior, repair, rollback, removal, runtime, or dependencies.
- Health semantics.
- Events, alerts, or incident behavior.
- NOC behavior.
- Security controls or threat assumptions.
- Local shell telemetry collectors.

Do not alter the meaning of a published versioned architecture baseline in
place. Use:

- A documented correction.
- A new architecture baseline.
- An ADR for a narrower decision.

Operational procedures must include prerequisites, validation, rollback, and
recovery. Examples must use synthetic data and placeholders.

UniFi compatibility documentation must identify the evidence supporting each
claim and must not generalize evidence across versions without validation.

## 26. Architecture Decision Records

Create ADRs under `docs/architecture/decisions/` using sequential identifiers,
for example:

`ADR-0001-agent-enrollment-and-authentication.md`

An ADR must include:

- Status.
- Date.
- Context.
- Decision.
- Alternatives considered.
- Security implications.
- Appliance-safety implications where applicable.
- Operational implications.
- Compatibility and migration implications.
- Evidence and validation requirements.
- Consequences.
- Follow-up work.

Codex must not silently resolve material architectural ambiguities inside
implementation code.

### 26.1 Required Before Phase 0

The following decisions require accepted ADRs before Phase 0 implementation:

#### ADR-0001 — Agent Enrollment, Identity, and Authentication

Define:

- Enrollment-token storage and transactional consumption.
- Agent identity lifecycle.
- Console proof of possession.
- Credential format, hashing, rotation, and revocation.
- Agent request authentication.
- Pending-agent capabilities and restrictions.

#### ADR-0002 — Agent Protocol, Sequencing, Idempotency, and Replay

Define:

- Schema and version negotiation.
- Sequence and epoch semantics.
- Idempotency behavior.
- Replay protection.
- Batch and acknowledgement semantics.
- Clock-skew handling.
- Server and agent restart behavior.

#### ADR-0003 — Core PostgreSQL Data Model and Tenant Isolation

Define:

- Core organization, site, console, and agent relationships.
- Tenant ownership and enforcement.
- Current-state integrity.
- Required constraints and indexes.
- Cross-tenant authorization strategy.
- Migration ownership.

#### ADR-0004 — Offline Spool and ACK Protocol

Define:

- Local persistence format.
- Hard disk and time bounds.
- Disk-write policy.
- Overflow and telemetry-shedding policy.
- Retry and ordered replay.
- Partial acceptance.
- Corruption recovery.
- Record deletion after acknowledgement.
- Identity and spool preservation.

#### ADR-0005 — Worker and Queue Model for Phase 0

Define:

- Whether Redis is required.
- Which Phase 0 work is synchronous or asynchronous.
- Delivery, retry, deduplication, scheduling, and failure semantics.
- PostgreSQL versus queue durability responsibilities.

#### ADR-0006 — UniFi Agent Runtime, Persistence, and Resource Safety

Define:

- Installation model.
- Self-contained runtime and dependency strategy.
- Any platform-specific exception requiring modification of the UniFi OS
  package database, system Python, global runtime, or system library.
- Persistent storage and filesystem strategy.
- Boot and autostart mechanism.
- Ordinary reboot recovery.
- UniFi OS upgrade survival and recovery.
- Detection and repair of removed or disabled components.
- Resource budgets per supported console family.
- CPU, RAM, process/thread, disk, disk-write, bandwidth, and concurrency
  controls.
- Disk-write minimization policy.
- Spool limits and interaction with ADR-0004.
- Recurring and boot-time scheduling and jitter.
- Watchdog, restart, backoff, and crash-loop prevention.
- Failure isolation and native-service priority.
- Agent upgrade model.
- Atomicity or recoverability.
- Safe rollback.
- Identity and spool preservation.
- Uninstall and cleanup.
- Separation of lifecycle privilege from telemetry privilege.
- Compatibility and evidence validation for each supported platform/version.

### 26.2 Required Later When Relevant

The following ADRs are required before their associated functionality is
implemented, but they do not all block Phase 0:

- Administrative authentication, MFA, session security, and break-glass access.
- UniFi API and collector compatibility policy.
- Configuration normalization, redaction, and hashing.
- Audit immutability and security-event storage.
- Probe-target safety and SSRF prevention.
- Agent packaging, signing, release channels, and fleet update policy beyond
  the Phase 0 runtime requirements in ADR-0006.
- Undocumented Protect integration.
- Notification and delivery semantics.
- Retention, partitioning, and availability rollups.
- Any replacement of a required language or deployment technology.
- Any other material architectural decision not resolved by accepted authority.

## 27. Rules Codex Must Follow Before Modifying Code

Before any modification, Codex must:

1. Confirm that the user requested implementation rather than analysis only.
2. Confirm the named task and its scope.
3. Read repository-level and applicable nested `AGENTS.md` files completely.
4. Read relevant architecture and security documentation.
5. Inspect repository structure and relevant existing code.
6. Inspect `git status` and the current branch.
7. Refuse to modify files while on `main`.
8. Confirm the dedicated non-`main` branch is approved.
9. Request approval before creating or switching branches.
10. Preserve all unrelated and pre-existing user changes.
11. Identify affected contracts, migrations, security boundaries, appliance
    safety, and tests.
12. Determine whether the change requires an ADR.
13. State material assumptions instead of silently deciding ambiguous
    architecture.
14. Plan validation appropriate to the risk.
15. Avoid broad refactoring outside the approved scope.
16. Use repository tooling and pinned dependencies.
17. Apply the smallest coherent change that fully satisfies the task.
18. For agent changes, identify affected console families, versions, resource
    budgets, persistence assumptions, runtime/dependency assumptions,
    lifecycle privileges, and required evidence.

After the named task and branch are approved, Codex may perform the normal
development operations defined in Section 19 without repeated permission.

For analysis, review, or diagnosis requests, Codex must remain read-only unless
the user separately authorizes implementation.

## 28. Actions Requiring Explicit User Approval

The following always require explicit user approval, even when an
implementation task and branch are already authorized:

- Stage files.
- Create a commit.
- Amend, rebase, squash, cherry-pick, revert, merge, or rewrite commits.
- Push any branch or tag.
- Open, update, merge, or close a pull request.
- Create or delete a tag or release.
- Perform destructive Git operations.
- Access a production or shared environment.
- Access a real UniFi console or customer environment.
- Install, update, repair, roll back, or remove an agent on a real console.
- Use real credentials, tokens, certificates, or customer telemetry.
- Run database migrations against a non-test or non-disposable database.
- Delete or destructively transform retained data.
- Deploy, restart, or reconfigure shared or remote services.
- Change cloud, DNS, TLS, firewall, IAM, backup, or production infrastructure.
- Issue, rotate, revoke, recover, or otherwise operate on real credentials.
- Send email, webhook, Teams, or other external notifications.
- Publish packages, containers, artifacts, or releases.

Branch creation, switching, renaming, merging, and deletion also require
explicit user approval.

The following do not require repeated approval after a named implementation
task and branch are authorized:

- Task-scoped workspace edits.
- Tests, builds, formatters, linters, and type checks.
- Already-declared project tooling.
- Disposable local test services and containers.
- Disposable test-database migrations.
- Task-required local code generation.
- Relevant documentation and test updates.

Never commit without explicit user approval.

Never push without explicit user approval.

Never force-push.

Force-push is prohibited under all circumstances and cannot be authorized by
ordinary task approval.

## 29. Required Stop Conditions

Stop and ask the user for direction when:

- The current branch is `main` and modifications are requested.
- The named task or dedicated branch has not been approved.
- Required work would overwrite or conflict with existing user changes.
- A required architectural decision lacks an accepted ADR.
- Authentication, tenancy, credential, sequencing, ACK, replay, persistence,
  dependency, or resource-safety behavior is ambiguous.
- The requested behavior could modify UniFi configuration.
- Telemetry collection could modify operating-system state.
- An agent lifecycle operation would affect a non-ARGUS resource.
- Agent installation would modify the UniFi OS package database, system Python,
  a global runtime, or an unrelated library without an accepted
  platform-specific ADR and compatibility evidence.
- An install, update, repair, rollback, or uninstall path could restart,
  reconfigure, or interfere with a native UniFi service.
- Lifecycle privileges could become accessible to telemetry collectors.
- A local command cannot be shown to be read-only, bounded, and safe.
- A UniFi compatibility, persistence, upgrade-survival, dependency, or
  resource-budget claim lacks evidence.
- Resource pressure cannot be safely resolved through telemetry reduction,
  shedding, or shutdown.
- Spool behavior could fill or materially endanger console storage.
- Restart or watchdog behavior could create an infinite or high-frequency loop.
- Identity or unacknowledged spool state could be lost during an ordinary
  reboot or supported agent upgrade without an approved recovery decision.
- Secure or appliance-safe completion requires new credentials or unauthorized
  external access.
- Real UniFi console testing is required but has not been explicitly
  authorized.
- A destructive operation is necessary.
- Tests would require weakening a security or appliance-safety control.
- The scope would materially expand beyond the approved task.
