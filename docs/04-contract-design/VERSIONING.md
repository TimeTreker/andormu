# Versioning and Compatibility

## Contract versioning

Use explicit schema/API versions and define backward/forward compatibility rules before code generation.

## Definition versioning

WorkflowRevision and TaskDefinition revisions are immutable.

Mutable aliases such as `latest` may exist only as user conveniences. They are resolved before ExecutionSnapshot creation.

## Redrive

Uses the original ExecutionSnapshot contract/version unless an explicit migration mechanism is designed later.

## Deprecation

Do not delete a revision still referenced by historical runs. Deprecation means “not recommended for new runs,” not “erase history.”
