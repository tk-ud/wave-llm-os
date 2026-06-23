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
anchor-fixed phase sweep

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

このリポジトリは、`SPEC` と `NOTE` を束として読む構成です。

```text
SPEC = 意味・処理・判定の正本
NOTE = SQL / DDL / 実装 projection / 補足説明
```

## Core Routing Bundle

全体の入口です。

```text
SPEC_CANONICAL_CORE_RESUME.md
NOTE_SQL_IMPLEMENTATION_MAP.md
```

`SPEC_CANONICAL_CORE_RESUME.md` は、不変条件と各SPECへのルーティングを持ちます。

`NOTE_SQL_IMPLEMENTATION_MAP.md` は、NOTE がどの SPEC の実装 projection / 補足説明として残るかを整理します。

## Runtime Behavior Bundle

`wave`、`coherence`、`decoherence`、`pressure`、`Phase`、`Sleep`、`collapse` の読み方を扱います。

```text
SPEC_RUNTIME_BEHAVIOR_MODEL.md
NOTE_AGENT_METAPHOR_MAPPING.md
NOTE_QUATERNION_PHILOSOPHY.md
```

SPEC が runtime behavior 名の正本です。

NOTE はラベル説明と四元数風説明の補足です。

## Reference / Identity Bundle

UUID、index、`bigint[]` index array の役割分担を扱います。

```text
SPEC_REFERENCE_MODEL.md
NOTE_INDEX_ARRAY_CANONICAL.md
```

SPEC が semantic reference rule の正本です。

NOTE は SQL 実装者向けの短い reminder です。

## Semantic Table / SQL Projection Bundle

観測、語彙、文法、文法連携、残差保存の table family を扱います。

```text
SPEC_SEMANTIC_TABLES.md
NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md
```

SPEC が semantic table の意味を定義します。

NOTE は DDL / SQL projection を保持します。

## Reply / Search / Decode Bundle

同期応答、入力処理、構造探索、verification、decoder / collapse を扱います。

```text
SPEC_REPLY_PIPELINE.md
SPEC_SEARCH_AND_VERIFICATION.md
NOTE_NEAR_NEIGHBOR_SEARCH.md
NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md
```

SPEC が Reply と verification の正本です。

NOTE は retrieval / SQL diff / DDL の実装補助です。

## Cron / Phase / Sleep Bundle

scheduled processing、Phase relation candidate、Sleep consolidation を扱います。

```text
SPEC_CRON_PIPELINE.md
SPEC_SCORING_AND_THRESHOLDS.md
NOTE_PHASE_RELATION_CANDIDATE.md
```

SPEC が cron job、Phase、Sleep、score / threshold の正本です。

NOTE は `phase_relation_candidate` の storage / query sketch です。

## Log / Aggregate / Archive Bundle

観測証跡、mutation証跡、current pressure、履歴snapshot、archive registry を扱います。

```text
SPEC_LOG_AGGREGATE_ARCHIVE.md
NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md
```

SPEC が `logs.coherence`、`logs.diff`、`logs.current`、`aggregate.current`、archive registry の役割を定義します。

NOTE は必要な SQL / DDL projection を保持します。

## Operation / Runtime Support Bundle

promotion、delete、quarantine、freeze、scheduler state、remote event trust を扱います。

```text
SPEC_OPERATION_GATE.md
SPEC_CORE_STATE_AND_SCHEDULER.md
SPEC_REMOTE_TRUST.md
NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md
```

SPEC が operation gate と runtime support table の正本です。

NOTE は必要な実装projectionだけを保持します。

## Prohibited / Legacy Bundle

現在の正本から除外された設計や、禁止された canonical pattern を扱います。

```text
SPEC_PROHIBITED_CANONICAL_PATTERNS.md
MIGRATION_LEGACY_REGISTER.md
legacy_design/
```

`SPEC_PROHIBITED_CANONICAL_PATTERNS.md` は、現在の正本に入れてはいけない pattern を定義します。

`MIGRATION_LEGACY_REGISTER.md` と `legacy_design/` は historical context です。

## Scale / Cost Bundle

学習量データ換算、一回の探索計算量、storage-scale intelligence を扱います。

```text
SPEC_SCALE_AND_COST_MODEL.md
NOTE_QUATERNION_PHILOSOPHY.md
```

SPEC が scale / cost model の正本です。

NOTE は背景説明に留まります。

---

# Contribution

Issue / PR 発行は大歓迎です！

設計上の違和感、実装上の改善案、仕様の読みづらさ、説明不足などがあれば、遠慮なく投げてください。
