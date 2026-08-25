---
name: validate-implementation
description: 指定された.steeringの要求・設計・tasklistと実装・テストの整合性をread-onlyで厳密に検証する。実装完了後の独立した品質判定に明示的に使用する。コード修正、tasklist更新、文書だけのレビューには使用しない。
---

# Validate Implementation

実装を変更せず、必ずcustom agent `implementation_validator`を使って完了可否を判定する。

## 入力契約と停止条件

- `.steering/[YYYYMMDD]-[task]/`を1件、明示入力として受け取る。
- 引数なし、対象不存在、`.steering/`外、または`requirements.md`、`design.md`、`tasklist.md`不足では検証を開始しない。
- 未実装taskが残る場合も変更はせず、検証可能な範囲と未完了状態を報告する。
- `implementation_validator`を利用できない場合はmain agentだけで代行せず、検証未完了として停止する。

## 実行手順

1. main agentが対象steering、関連する永続文書、仕様、実装範囲を特定する。
2. `implementation_validator`へ対象パス、関連コードとテスト、制約、期待する報告形式を渡し、`$steering`を明示使用させる。
3. validatorにrequirementsと受け入れ条件の充足、designとの一致、tasklistの実態、エラー処理、回帰リスク、テスト不足をread-onlyで確認させる。ソース、docs、`.steering/`を変更させない。
4. `package.json`の`scripts`を確認させ、`lint`、`test`、`test:coverage`のうち存在して対象に関連するscriptだけを実行させる。未定義scriptは不足として報告し、暗黙に必須化しない。
5. 固定カバレッジ閾値は`docs/development-guidelines.md`で合意済みの場合だけ評価する。テストが書き込みを必要とする場合は、main agentがコマンドと書き込み先を確認し、親ターンのsandboxとapprovalに従う。
6. agentの完了を待ち、結論、重大度順のfinding、根拠ファイルと位置、実行した検証、残課題を受け取る。
7. main agentが結果と重要な検証出力を照合し、合格、要修正、検証不能のいずれかを日本語で報告する。修正が必要でも、このSkill内では実装へ戻らない。

## 完了条件

- `implementation_validator`が対象steeringと実装をread-onlyで検証している。
- 利用可能な品質scriptだけが実行され、未定義・失敗・未実行理由が区別されている。
- main agentが重大度順のfindingと完了判定を統合している。
- コード、docs、`.steering/`に変更がない。
