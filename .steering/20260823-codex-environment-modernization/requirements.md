# 要求内容

## 概要

AI駆動開発テンプレートに混在する旧式・非公式なCodex設定を、現行Codexが自動検出できるSkillとcustom agentの構成へ移行する。既存の `define-feature` → `setup-project` → `plan-feature` → `implement-feature` → `validate-implementation` という工程分離は維持しつつ、main agentのコンテキスト消費を抑える委任境界を明確にする。

## 背景

現状は `.agents/commands/*.md`、`.agents/agents/*.md`、`.agents/settings.json`、`collab` feature flag、疑似YAMLの `subagent:` 呼び出しが混在している。このため、ワークフローやagentの検出がセッション中の推測と手動探索に依存し、Codex CLI、IDE extension、desktop appで一貫した動作を保証できない。

また、空の `docs/ideas/initial-requirements.md` が最初から配置されているため、初期要件フローが追加仕様モードへ誤分岐し、未入力の要件から永続ドキュメントを生成する可能性がある。

## 計画時の前提と例外

- 正本は `docs/ideas/20260820-codex-environment-modernization.md` とする
- `AGENTS.md`、`PROJECT_CONTEXT.md`、現行の `.agents/`、`.codex/`、`README.md` を現状把握の入力とする
- 通常の機能計画で前提となる6つの永続ドキュメントは、このテンプレートにはまだ存在しない
- 今回は永続ドキュメントを生成する開発環境自体の移行であるため、`setup-project` 前提を例外扱いとし、本計画の作成を許可する
- React Native、Expo managed workflow、JavaScript、Node 22、Expo SDK 54という技術前提は変更しない

## 実装対象

### 1. 初期要件フローの正常化

- 未入力テンプレートを `docs/ideas/` からSkillのassetへ移す
- `define-feature` と `setup-project` が `missing`、`blank`、`valid` を内容で判定する
- `missing` または `blank` では永続ドキュメントを生成しない
- `valid` の場合だけ追加仕様作成または永続ドキュメント作成へ進む

### 2. 主要ワークフローのSkill化

以下の7ワークフローを `.agents/skills/<name>/SKILL.md` として自動検出可能にする。

- `init-project`
- `define-feature`
- `setup-project`
- `plan-feature`
- `implement-feature`
- `review-docs`
- `validate-implementation`

`add-feature` は削除し、利用者向けの呼び出しは `$skill-name` 形式へ統一する。

### 3. custom agentの現行形式への移行

以下の4役割を `.codex/agents/*.toml` に定義し、ファイル名とagent名をunderscore形式で統一する。

- `document_author`
- `feature_planner`
- `doc_reviewer`
- `implementation_validator`

各TOMLに `name`、`description`、`developer_instructions` を定義し、役割定義を一元化する。`.codex/config.toml` の `config_file` 登録には依存せず、自動検出を使用する。

### 4. main agentとsubagentの責務分離

- main agentはユーザー対話、意思決定、分割、統合、差分確認、完了判定を担当する
- `document_author` は指定された仕様・永続文書だけを作成する
- `feature_planner` は指定された `.steering/` だけを作成・更新する
- `doc_reviewer` は読み取り専用で文書をレビューする
- `implementation_validator` は実装と仕様を検証し、ソースコードや `.steering/` を変更しない
- 一般的な調査には組み込み `explorer`、実装には組み込み `worker` を使う
- 書き込みを委任する場合は所有ファイルまたは所有ディレクトリを明示し、同一ファイルの同時編集を避ける

### 5. 既存専門Skillの整理

以下の7つを明示呼び出し専用として維持し、役割を持つagentが名前を指定して使用する。

- `prd-writing`
- `functional-design`
- `architecture-design`
- `repository-structure`
- `development-guidelines`
- `glossary-creation`
- `steering`

長大な説明は `references/`、コピー用テンプレートは `assets/` へ分離し、汎用Expoテンプレートに不要なCLI製品例・カードゲーム固有例を除去する。権限制御を `allowed-tools` に依存させない。

### 6. invocation policyと安全境界

- `review-docs` だけは自然文からの暗黙呼び出しを許可する
- 残る6つのワークフローSkillと7つの専門Skillは `policy.allow_implicit_invocation: false` とする
- `allow_implicit_invocation` はSkill選択だけを制御するため、明示呼び出し専用ワークフロー相当の自然文依頼を受けた場合の停止規則を `AGENTS.md` に記載する
- 実効権限はsandbox、approval、managed policyに従い、論理的な所有範囲は委任指示と最終diff確認で担保する

### 7. 利用者向け文書と設定の更新

- `AGENTS.md` は短い恒久ルール、Skillルーティング、停止条件、委任規則に整理する
- `PROJECT_CONTEXT.md` はプロジェクト背景と技術前提の正本とする
- `README.md` と `.agents/README.md` は新方式の導入手順と `$skill-name` 呼び出しを案内する
- `.steering/` をリポジトリに残す進捗の正本として扱えるよう、`.gitignore` の一律除外を解除する
- 使い捨てコピーで最終形の `.codex/config.toml` と新しいagent定義を動的検証した後、実リポジトリの旧 `collab` と重複したagent定義を除去してagent共通設定だけを置く
- `.agents/settings.json`、旧 `.agents/commands/`、旧 `.agents/agents/` は新方式の起動確認後に削除する

### 8. 品質ゲートの整合

- `implement-feature` と `validate-implementation` は `package.json` に存在するscriptsを検出して、利用可能なlint、test、coverageだけを実行する
- 存在しないscriptを暗黙に必須化しない
- 固定カバレッジ閾値は `docs/development-guidelines.md` で合意済みの場合だけ適用する
- `init-project` はExpo managed workflow + JavaScriptの土台を作り、利用可能な品質scriptを報告するが、全プロジェクトへ一律のテストライブラリを強制しない

## 受け入れ条件

### 初期要件

- [x] 初期状態で `docs/ideas/initial-requirements.md` が存在せず、テンプレートが `define-feature` Skillのassetとして存在する
- [x] 必須セクションまたはファイル不足を `missing` と判定する
- [x] 空欄またはプレースホルダーを含む必須項目を `blank` と判定する
- [x] 全必須項目を具体的に満たす場合を `valid` と判定する
- [x] ファイル不存在の `missing` だけがテンプレートから新規作成され、既存ファイルのセクション不足では入力済み内容を保持して不足だけを補完する
- [x] `setup-project` は `missing` と `blank` で生成を開始せず、不足項目を報告する

### Skills

- [x] 7つの主要ワークフローが `.agents/skills/<name>/SKILL.md` から自動検出される
- [x] 明示呼び出し専用の6ワークフローを `$skill-name` で起動できる
- [x] `review-docs` を `$review-docs` と自然文の両方で起動できる
- [x] `review-docs` が1ファイル、複数ファイル、引数なしの入力契約を満たし、常に `doc_reviewer` を使用する
- [x] 7ワークフローSkillと7専門Skillの `agents/openai.yaml` が定義したinvocation policyと一致する

### Agents

- [x] 4つのcustom agentが `.codex/agents/*.toml` から名前どおりに自動検出される
- [x] custom agentの役割定義が各TOMLの `developer_instructions` に一元化される
- [x] `.codex/config.toml` の `config_file` 登録に依存しない
- [x] main agent、custom agent、組み込み `explorer` / `worker` の役割が各Skillで一貫している
- [x] 書き込みagentの所有範囲、返却要約、main agentによる最終diff確認がワークフローに含まれる

### 設定・文書・旧形式の除去

- [x] `.codex/config.toml` が現行スキーマで読み込まれ、旧 `collab` に依存しない
- [x] `.agents/settings.json`、`.agents/commands/`、`.agents/agents/` が存在しない
- [x] 実行時に参照される設定・案内・Skillに疑似YAMLの `subagent:` 呼び出しがない
- [x] `AGENTS.md`、`README.md`、`.agents/README.md` が新しいSkill・agent形式だけを案内する
- [x] `.steering/[YYYYMMDD]-[task]/` の計画文書が `.gitignore` で除外されず、Gitで進捗の正本として管理できる
- [x] 新旧形式が最終状態で併存しない

### 品質

- [x] TOMLとYAMLが構文上有効である
- [x] 新規CodexセッションでSkillとcustom agentの検出テストを実行できる
- [x] `codex doctor --summary` のConfiguration欄でproject configの読み込みを確認できる
- [x] 利用可能な品質scriptだけを実行する方針が `init-project`、`implement-feature`、`validate-implementation` で一致する
- [x] 変更後の静的検索とGit差分で旧形式への依存と所有範囲外の変更がない

## 成功指標

- 主要7ワークフローとcustom agent 4役が新規セッションで検出される
- 初期要件の `missing`、`blank`、`valid` の正負ケースがすべて期待どおりになる
- P0受け入れ条件がすべて検証証跡付きで完了する
- main agentが長い文書生成・調査・検証ログを抱えず、境界の明確な作業を委任できる

## スコープ外

以下は今回の実装では行わない。

- React Native、Expo、JavaScriptから別技術スタックへの変更
- Node 22またはExpo SDK 54のアップグレード
- アプリケーション機能の実装
- MCPサーバーや外部サービス連携の追加
- Skill群のプラグイン配布
- permission profileやhookによるファイル単位の強制的な所有権制御
- 複数workerの効果測定やトークン消費比較など、元仕様のP2改善項目

## 参照ドキュメント

- `docs/ideas/20260820-codex-environment-modernization.md` - 元の仕様
- `AGENTS.md` - 現行のプロジェクト運用ルール
- `PROJECT_CONTEXT.md` - 技術前提
- `.agents/README.md` - 現行のAIワークフロー案内
- `.codex/config.toml` - 現行のCodex設定
- `.agents/skills/steering/SKILL.md` - steeringの運用規則
