# Workflow Definitions Product Design

A definition page should show:

- name/namespace,
- latest and historical immutable revisions,
- content digest,
- producer/source platform,
- graph preview,
- input/output interface,
- referenced TaskDefinitions,
- revision diff,
- validation status,
- runs by revision.

## Actions

- validate proposed revision,
- compare revisions,
- create run from exact revision,
- deprecate revision (without deleting historical references).

Avoid a “latest silently follows changes” execution experience.
