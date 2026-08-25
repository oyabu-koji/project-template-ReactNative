---
name: development-guidelines
description: docs/architecture.mdとdocs/repository-structure.mdからdocs/development-guidelines.mdを作成・更新し、JavaScript・React Native・Expoの実装規約と開発プロセスを定義する場合、または合意済み規約をコード実装時に参照する場合に明示的に使用する。機能要件やアーキテクチャの決定には使用しない。
---

# Development Guidelines

## 入力と出力

- 文書作成の必須入力: `docs/architecture.md`、`docs/repository-structure.md`
- 出力: `docs/development-guidelines.md`
- 実装支援時: 既存の `docs/development-guidelines.md` を最優先し、このSkillは補助にだけ使う

必須文書がない、品質方針が未決定、またはアーキテクチャと配置規則が矛盾する場合は、推測で規約を確定せず不足を返す。

## 手順

1. 技術前提、レイヤー境界、ファイル配置、既存の品質scriptを確認する。
2. 新規作成時だけ [assets/template.md](assets/template.md) を使う。更新時は既存文書の確定済み判断を維持する。
3. コード規約の詳細が必要な場合は [references/implementation.md](references/implementation.md)、作業工程と品質ゲートは [references/process.md](references/process.md) を読む。
4. JavaScript/JSDoc、component、hook、service、エラー、セキュリティ、性能、test、Git、review、Definition of Doneをプロジェクト要件に合わせて定義する。
5. `package.json` に存在する `lint`、`test`、`test:coverage` だけを品質ゲート候補にする。未定義scriptを暗黙に必須化しない。
6. 固定カバレッジ閾値はユーザーがプロジェクト要件として合意した場合だけ記載する。

## 完了条件

- 文書作成時は `docs/development-guidelines.md` だけを変更する。
- 規約がarchitecture、repository structure、利用可能なscriptsと整合する。
- 実装支援時は合意済み規約を適用するだけで、未合意の規約を追加しない。
- 変更内容、根拠、検証、残課題を日本語で簡潔に返す。
