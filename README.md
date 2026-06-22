# Wave LLMOS

このリポジトリは、LLMとの対話で発生しやすい「意味の使い捨て」「候補の早期collapse」「仕様や判断根拠の喪失」を、検査可能なデータベース構造として扱うために作りました。

この目的を達成するためのアプローチとして、観測・語彙・文法・文法連携・残差・圧力を PostgreSQL 上の永続構造として保持し、LLM的な意味探索を固定重みではなく、成長する意味構造として外部化しています。

実装上の正本は以下です。

```text
SPEC_CANONICAL_CORE.md
```

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

一回の会話 scope において、複数の意味軸を合成して出力候補状態を作るための説明モデルです。

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
# Document Agenda

以下は、上記の設計を説明するためのファイルです。

詳細は各ファイルをご確認ください。

## `SPEC_CANONICAL_CORE.md`

実装上の正本です。

canonical tables、reference rules、logs、operation gate、Phase、decoder/collapse boundary を定義します。

## `NOTE_SQL_TOKENIZATION_IMPLEMENTATION.md`

PostgreSQL 実装スケッチです。

input observation、token、vocabulary、grammar、logs、Phase candidates などのSQL上の表現を説明します。

## `NOTE_INDEX_ARRAY_CANONICAL.md`

semantic reference rule の説明です。

なぜ UUID ではなく `bigint[]` index array を意味参照として使うのかを説明します。

## `NOTE_PHASE_RELATION_CANDIDATE.md`

Phase Attention と `phase_relation_candidate` の説明です。

`logs.current` pressure、grammar_array expansion、sleep maintenance、attractor basin refinement を扱います。

## `NOTE_NEAR_NEIGHBOR_SEARCH.md`

構造的近傍探索の説明です。

index array search、SQL join diff、pgvector acceleration、decode-time verification の役割分担を説明します。

## `NOTE_QUATERNION_PHILOSOPHY.md`

Wave LLMOS の四元数風説明です。

`q = w + xi + yj + zk` の考え方と、起点固定の phase sweep を説明します。

## `MIGRATION_LEGACY_REGISTER.md`

legacy design から current core への移行・再解釈・棄却の記録です。

## `legacy_design/`

過去設計のアーカイブです。

現在の実装 authority ではありませんが、設計意図を追うための historical context として残しています。

---

# Contribution

Issue / PR 発行は大歓迎です！

設計上の違和感、実装上の改善案、仕様の読みづらさ、説明不足などがあれば、遠慮なく投げてください。
