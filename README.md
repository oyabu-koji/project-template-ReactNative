# AI-driven Expo Project Template

React Native + Expo managed workflow + JavaScriptの新規プロジェクトを、CodexのSkillsとsubagentsで進めるためのテンプレートです。

アプリケーション本体、`node_modules`、`package-lock.json` は含みません。プロジェクト作成後に `$init-project` でExpoの土台を生成します。

## 前提

- Node 22
- Expo SDK 54
- npm
- JavaScript（TypeScriptは既定で導入しない）

詳細な技術前提は `PROJECT_CONTEXT.md`、Codex向けの実行規則は `AGENTS.md` を参照してください。

## 開発フロー

Codex CLIまたはIDE extensionでは、`$` または `/skills` からWorkflow Skillを選択します。

1. `$init-project` でExpo managed workflowの土台を作る
2. `$define-feature` で `docs/ideas/initial-requirements.md` を作成・入力する
3. `$setup-project` で6つの永続文書を作る
4. `$define-feature` で `docs/ideas/YYYYMMDD-[feature-name].md` を作成・更新する
5. `$plan-feature docs/ideas/YYYYMMDD-[feature-name].md` で `.steering/` の計画を作る
6. `$implement-feature .steering/[YYYYMMDD]-[task]/` で実装する
7. `$validate-implementation .steering/[YYYYMMDD]-[task]/` で実装を検証する

文書だけをレビューする場合は、次のように指定できます。

```text
$review-docs
$review-docs docs/ideas/YYYYMMDD-feature.md
$review-docs docs/product-requirements.md docs/functional-design.md
```

`$review-docs` の引数を省略すると、存在する6つの永続文書を対象にします。「この文書をレビューして」のような自然文にも対応します。それ以外の主要Workflowは、意図しない書き込みを避けるため `$skill-name` を明示してください。

指定できるのは、実在する `docs/` 配下のMarkdownファイルだけです。1件でも不存在、`docs/` 配下外、またはMarkdown以外の入力があれば、レビューを開始せず再指定を求めます。

## ディレクトリ

- `.agents/skills/`: 共有Workflowと専門Skill
- `.codex/agents/`: project-scoped custom agents
- `docs/ideas/`: 初期要件と変更仕様
- `docs/`: 安定したプロダクト・設計文書
- `docs/decisions/`: 安定した運用・設計判断
- `.steering/`: task単位の要求・設計・進捗
- `.devcontainer/`: 任意の将来用開発コンテナ設定

`docs/ideas/initial-requirements.md` は初期状態では存在しません。`$define-feature` がSkill内のテンプレートから必要時に作成します。

## 永続文書

`$setup-project` はvalidな初期要件から次の6文書を依存順に作成します。

- `docs/product-requirements.md`
- `docs/functional-design.md`
- `docs/architecture.md`
- `docs/repository-structure.md`
- `docs/development-guidelines.md`
- `docs/glossary.md`

## 開発コマンド

- 通常起動: `npx expo start`
- リモート端末確認: `npx expo start --tunnel`
- Expo関連依存: `npx expo install <package>`

lint、test、coverageは、初期化後の `package.json` に存在するscriptsだけを実行します。固定カバレッジ基準はプロジェクトの `docs/development-guidelines.md` で合意した場合にだけ適用します。
