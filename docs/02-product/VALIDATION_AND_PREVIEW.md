# Validation and DAG Preview

Before submission, the product/API should detect:

- graph cycles,
- missing referenced tasks,
- invalid dependency ids,
- impossible or unsupported trigger rules,
- invalid input bindings,
- unresolved mutable references,
- invalid timeout/retry combinations,
- invalid dynamic expansion capability,
- forbidden inline secrets,
- version incompatibility.

Preview should show the normalized graph and resolved revision/digests before execution starts.
