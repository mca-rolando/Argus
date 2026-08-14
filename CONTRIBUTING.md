# Contributing to ARGUS

Thank you for your interest in ARGUS.

ARGUS is publicly available under the MIT License. The upstream repository is
maintained exclusively by Rolando Blanco and does not grant direct write access
to external contributors.

Contributions must use the fork-and-pull-request workflow.

## Contribution Workflow

1. Fork the ARGUS repository.
2. Clone your fork locally.
3. Create a dedicated branch for your change.
4. Implement and test the change in your fork.
5. Push the branch to your fork.
6. Open a Pull Request against the ARGUS upstream repository.

Do not request direct write or push access to the upstream repository.

## Before Starting

Before implementing a significant change:

- Review the existing issues and Pull Requests.
- Confirm that the proposed functionality is consistent with the ARGUS
  architecture and security model.
- Open an issue describing substantial architectural or behavioral changes.
- Avoid combining unrelated changes in one Pull Request.

## Development Requirements

Contributions should:

- Preserve the read-only monitoring model.
- Preserve outbound-only agent communication.
- Avoid GUI scraping and browser automation.
- Never include passwords, API keys, tokens, private keys, production
  configuration, or customer data.
- Include appropriate tests for new functionality.
- Update documentation when behavior or configuration changes.
- Follow the existing formatting, linting, typing and testing conventions.

## Commit Messages

Use clear, concise commit messages. Conventional commit prefixes are
recommended:

- `feat:` new functionality.
- `fix:` defect correction.
- `docs:` documentation-only changes.
- `test:` test additions or corrections.
- `refactor:` internal restructuring without behavioral changes.
- `chore:` repository maintenance.
- `ci:` continuous integration changes.

Example:

```text
feat(agent): add UniFi console heartbeat reporting
```

## Pull Request Review

Rolando Blanco is the sole maintainer and code owner of the upstream
repository.

All Pull Requests are reviewed at the maintainer's discretion. Submission of a
Pull Request does not guarantee acceptance or inclusion.

The maintainer may:

- Request additional tests or documentation.
- Request architectural or security changes.
- Close changes that do not align with the project.
- Modify, squash or reorganize accepted changes before merging.

External contributors cannot merge their own Pull Requests.

## Security Vulnerabilities

Do not disclose suspected security vulnerabilities through public issues,
discussions or Pull Requests.

Use GitHub's private vulnerability reporting mechanism whenever it is
available.

## License

By submitting a contribution, you agree that your contribution may be
distributed under the ARGUS MIT License.
