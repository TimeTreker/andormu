# FailureDescriptor Contract Proposal

A failure should be machine-actionable and human-readable.

Proposed fields:

- `origin`: workload | adapter | compute | engine | user
- `class`: workload | transient_external | infrastructure | resource | timeout | cancelled | engine | unknown
- stable `code`
- message
- retryable hint
- details reference
- causal attempt/event id
- backend code/category (namespaced)

Retry policy uses stable class/code matching, not free-form message parsing.
