# Security Triage Summary

## Overview

Lightweight security CI was added using GitHub Actions, Semgrep, a custom Semgrep rule focused on broken access control, and dependency auditing with pip-audit.

The CI pipeline produced six actionable findings:

* Broken Access Control (custom rule)
* SQL Injection (2 findings)
* SSRF
* Hardcoded JWT Secret
* Container Running as Root

No vulnerable third-party dependencies were identified by pip-audit.

---

## Priority 1: Broken Access Control

Severity: High

Source:
Custom Semgrep Rule

Location:
app/routes/records.py

Finding:

A record is retrieved using a client-supplied record_id and returned directly:

```python
record = db.get_record(record_id)
...
return record
```

The route validates that the record exists but does not verify ownership or authorization before returning the resource.

Risk:

An authenticated user may be able to access records belonging to other users by modifying the identifier supplied in the request.

Reason for Priority:

This finding aligns directly with the challenge hint regarding services trusting client-supplied resource identifiers. Broken access control is consistently one of the highest-impact application security issues because exploitation often results in unauthorized access to sensitive data.

Recommended Fix:

Validate ownership or authorization before returning the record. Access decisions should be based on authenticated user identity and server-side authorization logic rather than user-supplied identifiers.

---

## Priority 2: SQL Injection

Severity: High

Source:
Built-in Semgrep Rules

Location:
app/db.py

Finding:

A dynamically constructed SQL query is executed:

```python
rows = connection.execute(query).fetchall()
```

Semgrep identified both a formatted SQL query pattern and raw SQL execution pattern.

Risk:

An attacker may be able to manipulate database queries, resulting in unauthorized data access or modification.

Recommended Fix:

Use parameterized queries or SQLAlchemy query APIs rather than string-concatenated SQL.

---

## Priority 3: SSRF

Severity: Medium

Source:
Built-in Semgrep Rules

Location:
app/routes/webhooks.py

Finding:

User-controlled callback_url is passed directly into a server-side HTTP request.

```python
requests.get(str(request.callback_url), timeout=2)
```

Risk:

An attacker may be able to induce requests to internal services, cloud metadata endpoints, or other unintended destinations.

Recommended Fix:

Validate destination hosts against an allowlist and restrict supported URL schemes.

---

## Priority 4: Hardcoded JWT Secret

Severity: Medium

Source:
Built-in Semgrep Rules

Location:
app/auth.py

Finding:

JWT signing relies on a hardcoded secret.

Risk:

Secrets committed to source control are difficult to rotate and may be exposed through repository access or accidental disclosure.

Recommended Fix:

Load secrets from environment variables or a centralized secrets-management solution.

---

## Priority 5: Container Running as Root

Severity: Low

Source:
Built-in Semgrep Rules

Location:
Dockerfile

Finding:

No non-root USER is configured.

Risk:

Successful compromise of the application may provide elevated privileges within the container.

Recommended Fix:

Create and run the application under a dedicated non-root user.

---

## CI Coverage Assessment

The combination of built-in Semgrep rules and the custom broken-access-control rule successfully identified multiple classes of application security issues:

* Authorization flaws
* Injection flaws
* SSRF
* Secret management issues
* Container hardening issues

The custom rule provided additional value by identifying a business-logic authorization flaw that is not reliably detected by generic SAST rules.

---

## Dependency Audit Results

pip-audit completed successfully.

Result:

No known vulnerable dependencies were identified in the current dependency set.
