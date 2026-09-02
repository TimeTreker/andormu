# ADR-0005: TaskRun and TaskAttempt are separate aggregates

**Status:** Proposed

A logical TaskRun may contain multiple immutable TaskAttempts. Retrying never overwrites attempt history.
