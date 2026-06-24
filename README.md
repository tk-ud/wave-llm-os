# Wave LLMOS

このリポジトリは、LLMとの対話で発生しやすい「意味の使い捨て」「候補の早期collapse」「仕様や判断根拠の喪失」を、検査可能なデータベース構造として扱うために作りました。

この目的を達成するためのアプローチとして、観測・語彙・文法・文法連携・残差・圧力を PostgreSQL 上の永続構造として保持し、LLM的な意味探索を固定重みではなく、成長する意味構造として外部化しています。

ここで使う `wave`、`coherence`、`decoherence`、`pressure`、`Phase`、`Sleep`、`collapse` などの語は、詩的装飾ではありません。

それらは抽象的な runtime behavior の名前です。

PostgreSQL の table、row、index array、log は、その振る舞いを検査可能・再現可能・operation-gated にするための具象 projection です。

読み方は常に次の向きです。

```text
abstract runtime behavior
→ concrete inspectable PostgreSQL projection
```

逆向きに、DB table や row そのものを意味の正本として読んではいけません。

実装上の正本は `SPEC_CANONICAL_CORE_RESUME.md` から配線される各 `SPEC_*` ファイルです。

`NOTE_*` ファイルは SQL / DDL / 実装 projection / 補足説明として扱います。

過去の wave-geometry / psi / spectral-band 設計は `legacy_design/` に historical context として残しています。

---

# 波動と四元数

Wave LLMOS では、意味を単なる出力テキストではなく、接続・差分・候補・履歴として扱います。

会話は token の列だけではありません。

語彙、文法、関係、残差、蓄積された圧力によってできる意味の波です。

新しい入力が来たとき、Wave はすぐに答えへ飛びません。

入力を起点として固定し、その周囲の意味場を探索します。

語彙候補が現れます。

文法候補がその周囲で曲がります。

文法連携が離れた断片を接続します。

解決されなかった差分は残差として残ります。

近い波は同じ盆地へ吸収されます。

遠い波は別の盆地として残ります。

似ているが構造が違う波は、alias、branch、parent-child 候補になります。

このため、Wave LLMOS は四元数風に説明されます。

これは、実行時に物理的な四元数をそのまま計算するという意味ではありません。

一回の会話 scope において、複数の意味軸を合成して出力候補状態を作るための抽象表現です。

```text
q = w + xi + yj + zk
```

```text
w  = input anchor
xi = vocabulary-side coherence search
yj = grammar-side residual and draft exploration
zk = relation-guided correction and decode
q  = output candidate state
```

実装上は、これらの処理は順序立てて計算されます。

ただし、一回の会話 scope で見れば、それらはひとつの出力候補状態として合成されます。

```text
implementation view:
anchor-fixed semantic sweep

conversation-scope view:
quaternion-like output composition
```

Sleep maintenance は、この意味場を整えます。

単なるデータ整理ではありません。

近い波を吸収し、遠い波を残し、曖昧な波を draft evidence として保持します。

目的は、意味を神秘化することではありません。

意味が動ける状態を保ちながら、検査可能にすることです。

探索量の直感は、次のように読めます。

```text
expected_search_cost
≈ near_candidate_topK
+ Σ(missing_slot_i_candidate_topK × verification_cost_i)
+ scope_crossing_verification_cost
```

たとえば、一回の会話入力が約100文字で、出力候補が3文法程度の場合を考えます。

```text
near_candidate_topK = 3
missing_slot_count = 2
missing_slot_i_candidate_topK = 2
verification_cost_i = 1
scope_crossing_verification_cost = 1
```

この場合、

```text
expected_search_cost
≈ 3 + (2 × 2 × 1) + 1
≈ 8
```

つまり、3文法程度の短い応答であれば、探索は全文法・全語彙の自由組み合わせではなく、固定された語彙アンカーと文法配列の周辺にある未解決 slot の補完として収まります。

Wave LLMOS の探索は、すべての候補を総当たりするのではなく、近傍候補、虫食い slot、構造検証、scope-crossing continuity によって局所化されます。

---

# Repository Resume

このリポジトリは、`SPEC` と `NOTE` の束として読む構成です。

```text
SPEC = 意味・処理・判定の正本
NOTE = SQL / DDL / 実装 projection / 補足説明
```

同期応答は入力から検証済み collapse へ進み、cron は集約、Phase、Sleep、archive を扱います。

```text
reply: input -> observation -> candidates -> verification -> decoder -> collapse -> logs
cron : logs + semantic tables -> aggregate.current -> Phase -> Sleep -> archive
```

API は SQL function へ指示を出す reception / orchestration / merge / delivery layer であり、API 自身の作業記憶や判断根拠として DB table を直接使ってはいけません。

## Core Routing Bundle

```text
正本入口と、SPEC / NOTE の対応関係を配線する最上位 bundle。
```

- spec **`SPEC_CANONICAL_CORE_RESUME.md`**
- note **`NOTE_SQL_IMPLEMENTATION_MAP.md`**

## Runtime Boundary Bundle

```text
API orchestration、SQL Response Engine、tmp_context、runtime envelope、DB scheduled job の境界を扱う bundle。
```

- spec **`SPEC_RUNTIME_ENGINE_BOUNDARY.md`**
- spec **`SPEC_API_THINKING_ENGINE_BOUNDARY.md`**
- spec **`SPEC_API_SECTION_LOOP.md`**
- spec **`SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`**
- spec **`SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`**
- spec **`SPEC_RUNTIME_RESULT_ENVELOPE.md`**
- spec **`SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`**
- spec **`SPEC_BACKGROUND_DRIVER_API_WIRING.md`**

## SQL Runtime Design Routing Bundle

```text
SQL function package、section fragment、tmp_context.json / temporary context、runtime envelope schema を決める前に参照する routing bundle。
```

- spec **`SPEC_SQL_FUNCTION_PACKAGE.md`**
- spec **`SPEC_SECTION_FRAGMENT_SCHEMA.md`**
- spec **`SPEC_SQL_RESPONSE_ENGINE_BOUNDARY.md`**
- spec **`SPEC_API_SECTION_LOOP.md`**
- spec **`SPEC_TMP_CONTEXT_JSON_BOUNDARY.md`**
- spec **`SPEC_RUNTIME_RESULT_ENVELOPE.md`**
- spec **`SPEC_DB_SCHEDULED_JOB_BOUNDARY.md`**
- spec **`SPEC_OPERATION_GATE.md`**
- spec **`SPEC_REFERENCE_MODEL.md`**
- spec **`SPEC_LOG_AGGREGATE_ARCHIVE.md`**
- spec **`SPEC_PROHIBITED_CANONICAL_PATTERNS.md`**

## Runtime Behavior Bundle

```text
wave、coherence、decoherence、pressure、Phase、Sleep、collapse を定義する bundle。
```

- spec **`SPEC_RUNTIME_BEHAVIOR_MODEL.md`**
- note **`NOTE_AGENT_METAPHOR_MAPPING.md`**
- note **`NOTE_QUATERNION_PHILOSOPHY.md`**

## Reference / Identity Bundle

```text
UUID、index、bigint[] index array の参照責務を分離する bundle。
```

- spec **`SPEC_REFERENCE_MODEL.md`**
- note **`NOTE_INDEX_ARRAY_CANONICAL.md`**

## Semantic Table / SQL Projection Bundle

```text
観測、語彙、文法、連携、残差を PostgreSQL projection として扱う bundle。
```

- spec **`SPEC_SEMANTIC_TABLES.md`**
- note **`NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`**

## Reply / Search / Decode Bundle

```text
同期応答、意味探索、verification、decode / collapse の流れを扱う bundle。
```

- spec **`SPEC_REPLY_PIPELINE.md`**
- spec **`SPEC_SEARCH_AND_VERIFICATION.md`**
- note **`NOTE_NEAR_NEIGHBOR_SEARCH.md`**
- note **`NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`**

## Cron / Phase / Sleep Bundle

```text
scheduled processing、Phase relation、Sleep consolidation を扱う bundle。
```

- spec **`SPEC_CRON_PIPELINE.md`**
- spec **`SPEC_SCORING_AND_THRESHOLDS.md`**
- note **`NOTE_PHASE_RELATION_CANDIDATE.md`**

## Log / Aggregate / Archive Bundle

```text
観測証跡、mutation 証跡、current pressure、snapshot、archive を扱う bundle。
```

- spec **`SPEC_LOG_AGGREGATE_ARCHIVE.md`**
- note **`NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`**

## Operation / Runtime Support Bundle

```text
promotion、delete、quarantine、freeze、scheduler、remote trust を扱う bundle。
```

- spec **`SPEC_OPERATION_GATE.md`**
- spec **`SPEC_CORE_STATE_AND_SCHEDULER.md`**
- spec **`SPEC_REMOTE_TRUST.md`**
- note **`NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`**

## Prohibited / Legacy Bundle

```text
現在の正本から除外する pattern と、過去設計の historical context を扱う bundle。
```

- spec **`SPEC_PROHIBITED_CANONICAL_PATTERNS.md`**
- note **`MIGRATION_LEGACY_REGISTER.md`**
- note **`legacy_design/`**

## Scale / Cost Bundle

```text
学習量データ換算、探索計算量、storage-scale intelligence を扱う bundle。
```

- spec **`SPEC_SCALE_AND_COST_MODEL.md`**
- note **`NOTE_QUATERNION_PHILOSOPHY.md`**

---

# Public Deployment Note

Wave LLMOS は、入力された context を一時的な会話履歴として扱うだけではありません。

観測、語彙、文法、連携、残差、圧力として永続化し、後続の意味探索や応答に影響させる runtime です。

そのため、このモデルを公開サービスとして扱う場合は、`freeze mode` または同等の non-mutating boundary で運用することを推奨します。

理由は、ユーザー入力には個人情報、秘密情報、第三者の著作物、他社サービスの出力、その他の継続学習に適さない情報が含まれる可能性があり、通常運用ではそれらがその場で永続構造や aggregate pressure に反映され得るためです。

公開サービスとして継続学習を行わない場合は、UI または利用規約上で明示してください。

```text
入力内容は応答生成のためにのみ使用され、継続学習には使用されません。
```

継続学習を行う場合は、利用者への明示、同意、保存範囲、削除方針、source_kind / consent / retention policy の記録、および operation-gated ingest policy を定義してください。

---

# Contribution

Issue / PR 発行は大歓迎です！

設計上の違和感、実装上の改善案、仕様の読みづらさ、説明不足などがあれば、遠慮なく投げてください。
