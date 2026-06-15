# Security Triage Summary
## Overview
Lightweight security CI was added using GitHub Actions, Semgrep, a custom Semgrep rule focused on broken access control, and dependency auditing with pip-audit.

The CI pipeline produced six actionable findings:
* Broken Access Control (custom rule), High, app/routes/records.py
* SQL Injection, High, app/db.py
* SSRF, Medium, app/routes/webhooks.py
* Hardcoded JWT Secret, Medium, app/auth.py
* Container Running as Root, Low, Dockerfile

No vulnerable third-party dependencies were identified by pip-audit.

---

## Priority 1: Broken Access Control
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
Finding:

JWT signing relies on a hardcoded secret.

Risk:

Secrets committed to source control are difficult to rotate and may be exposed through repository access or accidental disclosure.

Recommended Fix:

Load secrets from environment variables or a centralized secrets-management solution.

---

## Priority 5: Container Running as Root
Finding:
No non-root USER is configured.

Risk:
Successful compromise of the application may provide elevated privileges within the container.

Recommended Fix:
Create and run the application under a dedicated non-root user.

---

## Dependency Audit Results
pip-audit completed successfully.

Result:
No known vulnerable dependencies were identified in the current dependency set.
