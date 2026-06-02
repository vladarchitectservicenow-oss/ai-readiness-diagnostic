# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in ai-readiness-diagnostic, please report it responsibly.

**Do NOT open a public issue.** Instead, contact the maintainer directly:

- Email: vladarchitect@github
- Subject line: "Security Vulnerability: ai-readiness-diagnostic"

### What to Include
- Description of the vulnerability
- Steps to reproduce
- Affected versions
- Potential impact
- Suggested fix (if available)

### Response Timeline
- Acknowledgment within 48 hours
- Initial assessment within 5 business days
- Fix timeline depends on severity:
  - Critical (credential exposure, data breach): 24-48 hours
  - High (privilege escalation, auth bypass): 1 week
  - Medium (information disclosure): 2 weeks
  - Low (hardening, best practices): Next release

## Supported Versions

| Version | Supported |
|---------|-----------|
| Latest (main) | ✅ Yes |
| Previous releases | ❌ No |

## Security Design Principles

- All REST API calls use HTTPS only
- Credentials encrypted at rest via GlideEncrypter
- Read-only operations on target instances (never modifies data)
- No PII collected in scan results
- Least-privilege access model
- Audit logging for all operations

## Credential Handling

- Scanner credentials for target instances are encrypted via GlideEncrypter
- Never hardcoded in source code
- CLI runner accepts credentials via environment variables or CLI arguments
- Credentials never logged in plaintext (sys_log references scan_id only)

## Known Issues

No known security vulnerabilities at this time.

## Hall of Fame

We appreciate responsible disclosure. Researchers who report valid vulnerabilities will be acknowledged here (with permission).
