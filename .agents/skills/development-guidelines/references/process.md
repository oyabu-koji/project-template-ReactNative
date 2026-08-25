# 開発プロセスガイド

## 基本フロー

1. `docs/ideas/` の仕様と関連する永続文書を確認する
2. `.steering/[YYYYMMDD]-[task]/` のrequirements、design、tasklistを確認する
3. taskを所有範囲が判定できる粒度にする
4. taskを1件実装し、受け入れ条件を検証する
5. tasklistの状態と検証証跡を更新する
6. 全task完了後、同じsteeringをvalidationへ渡す

## 品質ゲート

`package.json` に定義され、変更範囲に関係するscriptだけを実行する。

- `npm run lint`
- `npm test`
- `npm run test:coverage`

未定義scriptは実行せず、未導入であることを報告する。起動確認が必要なら `npx expo start`、リモート端末が必要な場合だけ `npx expo start --tunnel` を使用する。

## Review

- 要求と受け入れ条件を満たすか
- architectureとrepository structureの境界を守るか
- エラー、権限拒否、空状態、待機状態を扱うか
- 変更に対応するtestがあるか
- 安定した判断が関連docsへ反映されているか
- tasklistの記録と実際のdiffが一致するか

## Git

- ユーザーの既存変更を保護する
- 無関係な差分を整形・修正しない
- commit規則とbranch戦略はリポジトリの合意を正本とする
- Conventional Commitsを採用する場合も、scopeは実際の変更境界に合わせる
