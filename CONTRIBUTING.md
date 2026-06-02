# Contributing to ai-readiness-diagnostic

Thank you for considering contributing to ai-readiness-diagnostic! This document provides guidelines for contributing to the project.

## How to Contribute

### Reporting Bugs
1. Check [existing issues](https://github.com/vladarchitectservicenow-oss/ai-readiness-diagnostic/issues) to avoid duplicates
2. Use the Bug Report template
3. Include: ServiceNow version, scanner version, steps to reproduce, expected vs actual behavior, relevant logs

### Suggesting Features
1. Check existing issues for similar requests
2. Use the Feature Request template
3. Describe the use case, pain point, and proposed solution
4. If possible, suggest implementation approach

### Pull Requests
1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Write tests for your changes
4. Ensure all tests pass: `pytest tests/ -v`
5. Update documentation if applicable
6. Commit with descriptive messages in English
7. Push and open a PR against `main`

### Code Style
- All code, comments, variable names, and commit messages must be in English
- ServiceNow Script Includes: follow ServiceNow JS patterns (Class.create(), GlideRecord API)
- Python: follow PEP 8
- Copyright header required on all source files:
  ```
  /**
   * Copyright (c) 2026 Vladimir Kapustin
   * SPDX-License-Identifier: AGPL-3.0-only
   */
  ```

### Testing Requirements
- All new features require tests
- Bug fixes require a regression test
- Python tests: use `pytest` with mock ServiceNow runtime
- Target: 90%+ code coverage for new code

### Documentation
- Architecture changes: update `memory/checkpoints/architecture_summary.md`
- New dependencies: update `memory/checkpoints/dependency_report.md`
- Risk changes: update `memory/checkpoints/risk_report.md`
- New test scenarios: update `Validation/TEST CASES/ai-readiness-diagnostic/`

## License
By contributing, you agree that your contributions will be licensed under the AGPL-3.0-only License.
