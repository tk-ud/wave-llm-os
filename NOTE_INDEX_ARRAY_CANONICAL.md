# Supporting Note: Index Arrays

Canonical authority: `SPEC_CANONICAL_CORE.md`.

This file only explains the index-array rule.

---

# Rule

```text
UUID  = internal row / event identity only
index = semantic reference key
array = bigint[] index array only
```

Semantic references never use UUID.

Canonical schema does not define `uuid[]` columns.

Evidence UUIDs live in JSONB evidence only.

---

# Canonical Arrays

```text
vocabulary.token_array                 bigint[] of token.token_index
grammar.vocabulary_array               bigint[] of vocabulary.vocabulary_index
grammar_relation.grammar_array         bigint[] of grammar.grammar_index
phase_relation_candidate.grammar_array bigint[] of grammar.grammar_index
```

---

# Link Tables

Semantic link tables use indexes only.

Example:

```sql
create table grammar_relation_link (
  grammar_relation_index bigint not null references grammar_relation(grammar_relation_index),
  grammar_index bigint not null references grammar(grammar_index),
  position integer not null,
  primary key (grammar_relation_index, position)
);
```

---

# Short Form

```text
UUID is identity.
Index is meaning reference.
Array is index array.
```
