# Specification Draft: Runtime Engine Boundary Resume

## Authority

This draft routes the runtime boundary specifications for API-side thinking orchestration, SQL-side response execution, temporary context memory, result envelopes, and database scheduled jobs.

This file is a resume and routing map only.

Detailed rules belong to the routed specification files below.

---

# Runtime Role Summary

```text
API Thinking Engine = API-side orchestration and continuation control
SQL Response Engine = PostgreSQL function package for committed response computation
Temporary Context   = tmp_context.json working-memory projection
Runtime Envelope    = committed response-engine result summary
DB Scheduled Jobs   = bounded database jobs callable through SQL functions
```

The API Thinking Engine and SQL Response Engine are separate runtime roles.

They must not be collapsed into one scheduler.

---

# Routed Specification Map

## API Thinking Engine Boundary

```text
Controls continuation, stop / ask / retry decisions, and output branching.
Holds only keys and small envelope metadata in process memory.
```

- spec `SPEC_API_THINKING_ENGINE_BOUNDARY.md`

## SQL Response Engine Boundary

```text
Defines the PostgreSQL function package that performs committed search, verification, mutation evidence, and envelope generation.
```

- spec `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`

## Temporary Context JSON Boundary

```text
Defines tmp_context.json as external working memory addressed by key, not semantic authority.
```

- spec `SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`

## Runtime Result Envelope Boundary

```text
Defines the small committed result returned from SQL to the API Thinking Engine for output branching.
```

- spec `SPEC_RUNTIME_RESULT_ENVELOPE.md`

## DB Scheduled Jobs Boundary

```text
Defines bounded database jobs callable through SQL functions, separate from API thinking orchestration.
```

- spec `SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`

---

# Boundary Invariants

```text
API Thinking Engine must not directly mutate semantic authority.
SQL Response Engine must not emit user-visible output directly.
tmp_context.json must not become semantic authority.
DB Scheduled Jobs must not replace operation-gated promotion.
Runtime envelopes are usable only after SQL commit success.
```
