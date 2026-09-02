# ADR-0006: Dispatch is at least once with stable attempt identity

**Status:** Proposed

Andormu does not promise exactly-once external side effects. Duplicate dispatch of the same TaskAttempt id must be deduplicated by adapters/backends; retries use new attempt ids.
