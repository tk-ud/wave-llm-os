# Specification: Runtime Engine Boundary Resume

## Authority

This file routes the runtime boundary specifications for API-side thinking orchestration, Wave LLM section flow, SQL-side response execution, temporary decoded context projections, result envelopes, and database scheduled jobs.

This file is a resume and routing map only.

Detailed rules belong to the routed specification files below.

---

# Runtime Role Summary

```text
API Thinking Engine = API-side orchestration, merge, reinjection, and delivery control
API Section Loop    = split / think / generate / audit loop following Wave LLM core logic
Wave LLM            = section flow actor
SQL Response Engine = PostgreSQL function package for committed response computation
Temporary Context   = tmp_context.json decoded context projection, not reply context authority
Runtime Envelope    = committed response-engine result summary
DB Scheduled Jobs   = bounded database jobs callable through SQL functions
```

The API Thinking Engine and SQL Response Engine are separate runtime roles.

They are not one scheduler.

Reply-time source context enters the core through the canonical reply pipeline.

SQL-produced decoded context projections may be stored temporarily for API-side merge and reinjection.

---

# Routed Specification Map

## API Thinking Engine Boundary

```text
Controls continuation, stop / ask decisions, decoded-context merge, reinjection, and output branching.
Holds only keys, ordered decode keys, and small envelope metadata in process memory.
```

- spec `SPEC_API_THINKING_ENGINE_BOUNDARY.md`

## API Section Loop

```text
Defines the split -> think -> generate -> audit loop following Wave LLM core logic.
SQL produces res_context fragments; API stores tmp entries, sorts keys, reconstructs context, and reinjects.
```

- spec `SPEC_API_SECTION_LOOP.md`

## SQL Response Engine Boundary

```text
Defines the PostgreSQL function package that performs committed search, verification, mutation evidence, corpus / decode work, decoded projection writes, and envelope generation.
```

- spec `SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`

## Temporary Context JSON Boundary

```text
Defines tmp_context.json as SQL-produced decoded context projection storage for API merge and reinjection, not reply context authority and not semantic authority.
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
API Thinking Engine does not directly change semantic authority.
API Thinking Engine does not use database tables as API-side working memory or authority.
SQL Response Engine does not emit user-visible output directly.
tmp_context.json does not become reply context authority or semantic authority.
DB Scheduled Jobs do not replace operation-gated promotion.
Runtime envelopes and decoded projections are usable only after SQL commit success.
API section loop does not move tokenization or splitting out of SQL.
```
