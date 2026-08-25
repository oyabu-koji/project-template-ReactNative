# 設計書

## アーキテクチャ概要

Codexが標準探索する3層へ責務を分ける。

```text
人間 / main agent
    │  明示的な $skill-name または許可された自然文
    ▼
.agents/skills/<workflow>/SKILL.md
    │  目的・入力・停止条件・委任先を定義
    ├── built-in explorer / worker
    └── .codex/agents/<role>.toml
            │  役割・禁止事項・既定sandboxを定義
            ▼
       docs/ / .steering/ / application files
```

- `AGENTS.md` は全ターンで必要な短い規則とルーティングを保持する
- Skillはタスク固有のオーケストレーションを保持する
- custom agent TOMLは役割固有の `developer_instructions` を保持する
- `references/` は必要時だけ読む説明、`assets/` はコピー・生成元として使うテンプレートを保持する
- `.steering/20260823-codex-environment-modernization/` を今回の実装進捗の正本とする
- `.steering/` のタスク文書はGit管理対象とし、キャッシュや一時ログだけを個別に除外する

## 設計判断

### 自動検出を標準にする

custom agentは `.codex/agents/*.toml` に1役割1ファイルで配置し、`[agents.<role>] config_file` による二重登録は行わない。`.codex/config.toml` は次の共通設定だけを持つ。

```toml
[agents]
enabled = true
max_concurrent_threads_per_session = 4
```

### invocation policyをメタデータと運用規則の両方で表現する

各Skillの `agents/openai.yaml` にUI情報と `policy.allow_implicit_invocation` を置く。`review-docs` だけを `true`、残り13 Skillを `false` とする。

`false` は通常の自然文処理そのものを止めないため、`AGENTS.md` に明示呼び出し専用ワークフロー相当の自然文では書き込みや高コスト処理を開始せず、対応する `$skill-name` の明示を求める規則を追加する。

### 初期要件はファイルの存在ではなく内容で判定する

空テンプレートを `.agents/skills/define-feature/assets/initial-requirements-template.md` へ移し、初期状態の `docs/ideas/` には置かない。`define-feature` と `setup-project` は次の共通判定を同じ定義で使用する。

```text
ファイルなし ─────────────────────> absent + missing ──> assetから新規作成
既存ファイルの必須セクションなし ──> present + missing ─> 既存内容を保持して不足だけ補完
必須値が空・説明文・placeholder ───> present + blank ───> 既存内容を保持して空欄だけ補完
全必須値が具体的 ─────────────────> present + valid
```

ファイルの存在状態と内容判定を別に記録する。判定対象はProject Overview、Users、Product Goals、In Scope、Out of Scope、Acceptance Criteriaとする。`Open Questions` は存在だけでは停止条件にしない。

### 検証agentには論理的read-onlyを要求する

- `doc_reviewer` は `sandbox_mode = "read-only"` とする
- `implementation_validator` も既定は `sandbox_mode = "read-only"` とする
- テストがキャッシュや生成物への書き込みを必要とする場合は、main agentが対象コマンドと書き込み先を確認し、親ターンのapproval/sandboxで必要最小限を許可する
- どちらのagentも文書、ソースコード、`.steering/` を修正しないことを `developer_instructions` に明記する

### 品質scriptは検出方式にする

ソースコードを含まない汎用テンプレートへ特定のテストライブラリやカバレッジ閾値を強制しない。`package.json` の `scripts` を確認し、存在する `lint`、`test`、`test:coverage` を実行する。未定義のscriptは不足として報告するが、暗黙に実行したり固定閾値を適用したりしない。

## コンポーネント設計

### 1. ワークフローSkills

各 `SKILL.md` は次の共通構成にする。

- front matter: `name`、選択条件と非対象範囲を含む `description`
- 役割と入力契約
- 事前確認と停止条件
- main agentの責務
- 委任先、所有範囲、返却形式
- 実行手順
- 出力と完了条件
- 参照する `references/` / `assets/`

Skill別の主要設計は次のとおり。

| Skill | 入力 | 書き込み先 | 主な委任先 |
| --- | --- | --- | --- |
| `init-project` | リポジトリルート | Expo基盤・共通環境ファイル | `worker` |
| `define-feature` | アイデアまたは対象spec | 指定された `docs/ideas/*.md` | `document_author` |
| `setup-project` | validな初期要件 | 6つの永続文書 | `document_author`、`doc_reviewer` |
| `plan-feature` | `docs/ideas/YYYYMMDD-*.md` | 指定 `.steering/` の3文書 | `explorer`、`feature_planner` |
| `implement-feature` | 指定 `.steering/` | task所有範囲、`tasklist.md` | `explorer`、`worker` |
| `review-docs` | 0件または1件以上の `docs/**/*.md` | なし | 必ず `doc_reviewer` |
| `validate-implementation` | 指定 `.steering/` | なし | 必ず `implementation_validator` |

`review-docs` は引数なしの場合だけ6つの永続文書を対象とし、明示された1件または複数件を勝手に追加・置換しない。存在しないパスと `docs/` 外のパスでは停止する。

### 2. custom agents

| TOML | 既定sandbox | 責務 | 禁止事項 |
| --- | --- | --- | --- |
| `document_author.toml` | `workspace-write` | 指定文書の作成・更新 | 指定外の文書・コード変更 |
| `feature_planner.toml` | `workspace-write` | 指定 `.steering/` の3文書作成 | 実装、指定外の変更 |
| `doc_reviewer.toml` | `read-only` | 文書の重大度順レビュー | ファイル変更、実装検証の代行 |
| `implementation_validator.toml` | `read-only` | 仕様整合性・テスト・品質検証 | コード修正、`.steering/` 更新 |

各agentは結論、根拠ファイル、変更概要またはfinding、実行した検証、残課題を日本語で簡潔に返す。書き込みagentには、同じコードベースに他の作業者がいること、他者の変更を戻さないこと、指定された所有範囲だけを変更することを含める。

### 3. 既存専門Skills

- `SKILL.md` は短いルーティングと手順に絞る
- `guide.md` や長い説明は `references/` 配下へ移す
- `template.md` は `assets/` 配下へ移す
- `prd-writing/SKILL.md` のCLI製品例を汎用説明から除去または例として分離する
- 設計・ガイドライン・レビュー資料からカードゲーム固有の要求を除去する
- `allowed-tools` は権限制御として使用しないため削除する
- 全7 Skillに `agents/openai.yaml` を追加し、暗黙呼び出しを無効化する

### 4. 利用者向け文書

- `AGENTS.md`: 毎回必要な制約、正本へのリンク、Skillルーティング、明示呼び出し規則、委任・レビュー・検証の必須条件
- `PROJECT_CONTEXT.md`: React Native / Expo / JavaScript / Node 22 / Expo SDK 54とプロジェクト目的
- `README.md`: テンプレート利用者向けの新規開始手順と `$skill-name` 例
- `.agents/README.md`: `skills/`、`agents/openai.yaml`、`.codex/agents/` の関係

## データフロー

### 新規プロジェクトのbootstrap

```text
$init-project
  -> Expo基盤を作成
$define-feature
  -> assetから initial-requirements.md を作成・対話更新
  -> missing/blank/validを判定
$setup-project
  -> validのみ通過
  -> document_authorが専門Skillを順次使用
  -> PRDレビューと承認
  -> 残る5文書
  -> doc_reviewerが6文書を最終レビュー
```

### 追加機能

```text
$define-feature -> docs/ideas/YYYYMMDD-feature.md
$plan-feature -> .steering/[YYYYMMDD]-[task]/{requirements,design,tasklist}.md
$implement-feature -> workerが所有taskを実装しtasklistを同期
$validate-implementation -> implementation_validatorが読み取り・検証
```

### 文書レビュー

```text
$review-docs [docs path ...] または許可された自然文
  -> 入力検証
  -> doc_reviewer
  -> main agentが重大度順に統合
```

## 目標ディレクトリ構造

```text
.codex/
├── config.toml
└── agents/
    ├── document_author.toml
    ├── feature_planner.toml
    ├── doc_reviewer.toml
    └── implementation_validator.toml

.agents/
├── README.md
└── skills/
    ├── init-project/
    ├── define-feature/
    ├── setup-project/
    ├── plan-feature/
    ├── implement-feature/
    ├── review-docs/
    ├── validate-implementation/
    ├── prd-writing/
    ├── functional-design/
    ├── architecture-design/
    ├── repository-structure/
    ├── development-guidelines/
    ├── glossary-creation/
    └── steering/
```

各Skillディレクトリには `SKILL.md` と `agents/openai.yaml` を置き、必要な場合だけ `references/` と `assets/` を追加する。最終状態では `.agents/commands/`、`.agents/agents/`、`.agents/settings.json`、共通 `.agents/templates/` を残さない。

## 移行戦略

1. Git差分と現行構成を記録し、`.gitignore` の `.steering/` 一律除外を解除して本計画をGit管理可能にする
2. 新しいcustom agent TOML、ワークフローSkill、assetを追加する。この時点では旧インラインagent設定、旧command、旧agent Markdown、旧template、初期配置された空の初期要件を残す
3. 既存専門Skillとasset/reference配置を整理する
4. `AGENTS.md`、README類、`PROJECT_CONTEXT.md` の責務と案内を揃える
5. `/private/tmp` の使い捨てコピーへ最終形の `.codex/config.toml` を適用し、旧形式を除いた新構成一式を作る
6. 使い捨てコピーの新規セッションでSkill・agent・初期要件の正負ケースを確認する
7. 動的検証の成功後に、実リポジトリの `.codex/config.toml` を最終形へ切り替え、旧commands、旧agent Markdown、settings、旧template、空の初期要件を削除する
8. 実リポジトリの新規セッションで検出・起動テストを再実行する
9. 静的検索、設定smoke test、最終差分確認を行う
10. `.steering/` の検証証跡を更新し、`validate-implementation` へ引き渡す

この順序により、新方式が動かない状態で旧設定や旧資料を先に失うことを防ぐ。使い捨てコピーで最終構成を確認するまでは、実リポジトリの `.codex/config.toml` から旧インラインagent定義と `collab` を除去しない。assetへの移行も検証前は複製として扱い、旧参照元を残す。削除対象はGit管理下にあるため、切替後に問題が判明した場合も履歴から復元できる。

## エラーハンドリングと停止条件

- Skill入力のパスが存在しない、対象ディレクトリ外、必須ファイル不足の場合は変更を開始せず報告する
- 初期要件が `missing` / `blank` の場合は永続文書生成を停止する
- custom agentまたはSkillが新規セッションで検出されない場合は旧形式の削除へ進まない
- TOML/YAMLが解析できない場合は該当設定を修正してから起動テストを再実行する
- subagentが所有範囲外を変更した場合はmain agentが差分を隔離し、原因を解消してから続行する
- 利用可能な品質scriptが失敗した場合は失敗を記録し、移行完了にしない
- 外部到達性など今回の設定と無関係な `codex doctor` の終了コードは合否判定に使わない

## テスト戦略

### 静的検証

- 全 `SKILL.md` のfront matterに `name` と具体的な `description` がある
- 全 `agents/openai.yaml` が構文上有効でinvocation policyが設計どおりである
- 全custom agent TOMLが構文上有効で必須キーを持つ
- `.agents/settings.json`、`.agents/commands/`、`.agents/agents/` が最終状態で存在しない
- `.steering/[YYYYMMDD]-[task]/` が `.gitignore` で除外されない
- 実行時に参照されるファイルに `^\s*collab\s*=` と `^\s*subagent:\s*$` がない
- README類とAGENTSのパス・呼び出し例が実在する構成と一致する

### 起動・統合検証

- 新規CodexセッションのSkill一覧で主要7 Skillを確認する
- 6つの明示専用ワークフローを一時コピーで明示起動する
- `review-docs` を明示・自然文・0件・1件・複数件で試す
- 4 custom agentsを代表タスクで起動し、期待した名前と役割を確認する
- `setup-project` の `missing`、`blank`、`valid` fixtureを実行する
- `setup-project` で専門Skillの依存順、PRDレビュー、最終レビューを確認する
- `plan-feature`、`implement-feature`、`validate-implementation` でsteeringの受け渡しとvalidatorの非変更を確認する
- `codex doctor --summary` のConfiguration欄でproject configを確認する

破壊的な初期化やテストはリポジトリ本体ではなく、`/private/tmp` 配下の使い捨てコピーまたは検証用worktreeで行う。

## 依存ライブラリ

この移行自体ではアプリケーション依存ライブラリを追加しない。TOML/YAMLの検証は、リポジトリまたは実行環境に既に存在する安全なparserを優先し、検証だけを目的とした恒久依存を追加しない。

## セキュリティ・権限・並行性

- `workspace-write` はパス単位の強制境界ではないため、所有範囲は委任指示と最終diffで検証する
- 書き込みagentを並列化する場合は所有ファイルを重複させない
- 同じSkillや文書を複数agentに同時編集させない
- reviewとvalidationは原則read-onlyにする
- approvalや親ターンのsandboxをcustom agentの既定値より優先する
- 秘密情報や外部サービスを今回の設定へ追加しない

## 将来の拡張性

`.steering/` の状態名と更新規則として`pending`、`in-progress`、`done`、`blocked`、`cancelled`を定義した。`blocked`や`cancelled`を含む代表taskの状態遷移fixture、複数workerによるアプリ実装の並列化効果、subagentによるコンテキスト削減量、Skillのプラグイン配布は将来検討とし、今回のタスクリストには含めない。
