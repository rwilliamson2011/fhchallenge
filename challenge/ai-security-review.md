# AI-Assisted PR Security Reviewer

Use this template for the `ai-security-review.md` deliverable described in the candidate brief. Keep it short. The goal is to describe how you would safely add an AI-assisted PR security reviewer to this repo, not to build a full LLM agent.

## Inputs

The reviewer receives:

- Pull request diff
- Route map generated from FastAPI routes
- Authentication and authorization model notes
- Semgrep findings
- Dependency scan findings
  
The route map and auth model are especially important because broken access control issues often require understanding how a request is authenticated, which identity is being used, and whether access decisions are enforced consistently across routes.

## Outputs

Describe the reviewer's output format.

- Risk label (Low / Medium / High)
- Finding summary.
- Confidence. (Low / Medium / High)
- Suggested owner.
- Block/comment/escalate decision.
- Remediation suggestion.

## Guardrails

Describe how the system avoids unsafe or noisy behavior.

- Blocking requires deterministic scanner evidence or a custom rule match.
- Require deterministic evidence for PR failure.
- Detect likely false positives and provide confidence score (Low / Medium / High)
- Secrets, tokens, credentials, and production data must be excluded from model prompts. Reviews should use repository code and metadata only.

## Evaluation

Describe how you would measure whether the reviewer is working.

- Seeded test cases.
- False positive rate.
- Missed broken access control variants.
- Engineering feedback.
- Time saved on high confidence reviewed items
