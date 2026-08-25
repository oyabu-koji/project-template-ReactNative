# AI開発ワークフローの責務分離

- Date: 2026-08-23
- Status: accepted
- Related specification: `docs/ideas/20260820-codex-environment-modernization.md`

## Context

AI駆動開発では、アイデア整理、安定した設計、実装計画、コード変更、検証を同じ文書や会話へ混在させると、正本と進捗が曖昧になり、main agentのコンテキストも増大する。

旧運用ではワークフローを `.agents/commands/*.md` として案内していたが、repo共有の現行Codex形式は `.agents/skills/<name>/SKILL.md` である。各工程の責務分離は有効なため、形式だけを現行方式へ移行する。

## Decision

### 正本を分ける

- `docs/ideas/`: 初期要件と変更仕様
- `docs/`: 安定したプロダクト・設計文書
- `docs/decisions/`: 長期的に参照する運用・設計判断
- `.steering/[YYYYMMDD]-[task]/`: 実装単位の要求、設計、task進捗、検証証跡

### Workflowを分ける

```text
$init-project
  -> $define-feature
  -> $setup-project
  -> $define-feature
  -> $plan-feature
  -> $implement-feature
  -> $validate-implementation
```

- `$define-feature` は `docs/ideas/` の仕様だけを作成・更新する
- `$setup-project` はvalidな `initial-requirements.md` から6つの永続文書を作成する
- `$plan-feature` は追加仕様から `.steering/` の3文書だけを作り、実装しない
- `$implement-feature` はtasklistと実装を同期する
- `$validate-implementation` は実装を変更せず、専門agentで検証する
- `$review-docs` は指定された `docs/` 配下の文書を専門agentでレビューし、明示呼び出しと自然文レビュー依頼の両方を受け付ける

`add-feature` のように仕様、計画、実装、検証をまとめる入口は設けない。

### Agentの役割を分ける

- main agentはユーザー対話、判断、分割、統合、完了判定を担当する
- 文書作成、計画、レビュー、実装検証は境界の明確なagentへ委任する
- 調査には組み込み `explorer`、実装には所有範囲を指定した `worker` を使用できる
- 書き込みagentは指定された所有範囲だけを変更し、main agentが最終diffを確認する

## Consequences

- 仕様、安定文書、実装計画の所在が明確になる
- 実装前に要求と設計を確認でき、validationへ同じsteeringを引き渡せる
- main agentへ長い文書生成・調査・テストログを集中させずに済む
- 書き込みまたは高コストな6 Workflowは工程ごとに明示的なSkill起動が必要になる。read-onlyの`review-docs`は自然文からも起動できる
- 安定した判断へ昇格した作業メモは `docs/ideas/` に残さず、決定記録へ整理する必要がある
