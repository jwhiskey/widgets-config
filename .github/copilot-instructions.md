You are implementing the Widget Configuration Platform in Java 21 + Spring Boot 3.

Source of truth:
- docs/spec/README_Widget_Configuration_Platform.md

Hard rules:
- Registry is the only source of parameter definitions (types/defaults/options).
- A profile/config cannot introduce parameters not present in the Registry.
- Options can only be restricted downstream, never expanded.
- Deterministic resolution order:
  Registry defaults -> Base Tenant Profile -> Tenant Profile -> inherited Configs (optional) -> target Config overrides.
- Validate every override: type mismatch, out-of-range option, not editable, unknown key => reject.

Engineering rules:
- Use clean architecture packages: api / application / domain / infrastructure.
- Use PostgreSQL + Flyway.
- Provide OpenAPI annotations.
- Add tests for inheritance + validation (at least 8 cases).
