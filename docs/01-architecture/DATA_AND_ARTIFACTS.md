# Data and Artifact Passing

## Small values

Small structured parameters/results may be stored inline if they satisfy contract size/security rules.

## Large values

Large data uses references:

```text
ArtifactRef
  uri / locator
  media_type
  digest
  size (optional)
  metadata (bounded)
```

Examples include datasets, model files, checkpoints, logs, videos, and simulation outputs.

## Important rules

- No credentials/secrets inside durable WorkflowSpec/Event payloads; use SecretRef.
- Store content digests when available to improve reproducibility.
- A successful TaskRun should not depend on an ephemeral local file that cannot be resolved after worker loss.
- Inline payload size limits must be decided before contract freeze.
