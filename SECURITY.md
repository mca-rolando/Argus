# Security Policy

ARGUS is a security-sensitive observability platform. It communicates with
UniFi consoles, processes infrastructure telemetry, and maintains operational
and historical information about monitored environments.

Security reports are taken seriously and should be handled privately.

## Project Status

ARGUS is currently under initial development. There is no production-ready or
stable release at this time.

| Version | Status |
|---|---|
| `26.08-01-dev` | Active development; not supported for production use |
| Stable release | Not yet available |

## Reporting a Vulnerability

Do not report suspected security vulnerabilities through public GitHub issues,
discussions, comments, commits, or Pull Requests.

Use GitHub's private vulnerability reporting feature:

https://github.com/mca-rolando/Argus/security/advisories/new

A useful report should include:

- A clear description of the vulnerability.
- The affected component and version.
- Preconditions required to reproduce the issue.
- Reproduction steps or a minimal proof of concept.
- The expected and observed behavior.
- The potential security or operational impact.
- Any proposed mitigation, when available.

Do not include real credentials, API keys, private keys, customer information,
production telemetry, or other sensitive data in the report.

## Security-Relevant Areas

Examples of issues that should be reported privately include:

- Authentication or authorization bypasses.
- Agent enrollment or identity weaknesses.
- Cross-site or cross-organization data exposure.
- Exposure of locally stored UniFi credentials.
- Command, query, template, or path injection.
- Unsafe agent or server update mechanisms.
- Unauthorized modification of monitoring data.
- Sensitive information disclosed through APIs, logs, exports, or diagnostics.
- Cryptographic, token-handling, or secret-storage weaknesses.
- Dependency vulnerabilities with a demonstrated impact on ARGUS.

## Out of Scope

The following are generally outside the ARGUS vulnerability-reporting scope:

- Vulnerabilities exclusively affecting third-party UniFi products or services.
- Issues caused only by an unsupported or insecure deployment configuration.
- Denial-of-service tests performed against systems without explicit
  authorization.
- Automated scanner output without a reproducible security impact.
- Social engineering, phishing, or physical-security testing.
- Reports about versions that have been explicitly marked unsupported.

## Disclosure

Do not publicly disclose a vulnerability before a fix or mitigation has been
prepared and coordinated with the maintainer.

Rolando Blanco is the sole maintainer and determines the validation,
remediation, release, and disclosure process for ARGUS vulnerabilities.

## Safe Testing

Security testing must be performed only against systems you own or systems for
which you have explicit authorization. Do not access, modify, retain, or
disclose data belonging to other users or organizations.
