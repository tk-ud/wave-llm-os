# Specification: Remote Trust

This file is the canonical authority for remote node trust and remote event intake.

Remote trust is linked to core state.

Global switch:

```text
core_state.distributed_sync.enabled
```

Canonical remote node registry:

```text
remote_node_trust
```

Canonical fields:

```text
remote_node_uuid
remote_node_id
trust_level
enabled
public_key
metadata_json
created_at
updated_at
```

Trust levels:

```text
unknown
trusted
limited
blocked
```

Acceptance rule:

```text
core_state.distributed_sync.enabled = true
and remote_node_trust.enabled = true
and trust_level in ('trusted', 'limited')
-> remote_event may become input_observation(source_kind='remote_event')
```

Remote events are normal input observations after inbox or quarantine acceptance.

Even trusted nodes cannot mutate semantic tables directly.

Remote trust changes are mutations and must pass the operation gate.

Remote quarantine is represented by operation key:

```text
quarantine.remote_event
```
