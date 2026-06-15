# Security Triage Summary

## Finding 1: Broken Access Control

Severity: High

Description:
A user can retrieve resources by supplying arbitrary identifiers without ownership verification.

Impact:
An attacker could access other users' data.

Recommendation:
Validate ownership or authorization before returning resources.

---

## Finding 2: SQL Injection

Severity: High

Description:
User input is incorporated into database queries without parameterization.

Impact:
An attacker could manipulate queries and access or modify data.

Recommendation:
Use parameterized queries or ORM protections.

---

## Finding 3: JWT Validation Weakness

Severity: Medium

Description:
Token validation does not adequately verify critical claims or signature requirements.

Impact:
Improperly validated tokens may allow unauthorized access.

Recommendation:
Use strict JWT verification and claim validation.
