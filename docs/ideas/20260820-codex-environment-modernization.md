# Codex 開発環境の現行ベストプラクティス対応

- Status: implemented
- Implementation completed: 2026-08-24
- Documentation review completed: 2026-08-25
- Steering: [20260823-codex-environment-modernization](../../.steering/20260823-codex-environment-modernization/tasklist.md)

## 背景

このリポジトリは、React Native + Expo + JavaScript プロジェクトを AI 駆動で開始するためのテンプレートである。

現在の構成には、Codex が正式に認識する現行形式と、旧来または Claude Code 系の独自形式が混在している。特に `.agents/commands/` と `.agents/agents/` は、2026-08-20 時点の Codex の推奨形式と一致していない。

本仕様では、既存ワークフローの意図を維持しながら、Codex CLI、IDE extension、ChatGPT desktop app で一貫して利用できる構成へ移行するための課題と方針を整理する。

## 調査基準

- 調査日: 2026-08-20
- ローカル Codex CLI: `0.147.0`
- 参照資料:
  - [Build skills](https://learn.chatgpt.com/docs/build-skills)
  - [Subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents)
  - [Custom instructions with AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md)
  - [Custom prompts](https://learn.chatgpt.com/docs/custom-prompts)
  - [Config basics](https://learn.chatgpt.com/docs/config-file/config-basic)

## 優先度の定義

本仕様の P0、P1、P2 は障害の深刻度ではなく、今回の移行における対応必須度を表す。

- P0: 今回の移行で必ず対応する項目。未対応では移行完了としない
- P1: 今回の移行で可能な限り対応する重要項目
- P2: 今回の移行後も継続して改善できる項目

## 維持する設計方針

以下の設計方針は有効であり、移行後も維持する。

- React Native + Expo managed workflow + JavaScript を標準技術スタックとする
- Node 22 と Expo SDK 54 を明示的な依頼なしに変更しない
- Expo 関連依存は `npx expo install` で追加する
- `docs/ideas/` を仕様の作成・更新に使う
- `docs/` を安定した要件と設計の正本にする
- `.steering/` をタスク単位の計画と進捗管理に使う
- 仕様作成、計画、実装、検証を分離する
- `.agents/skills/` に再利用可能なワークフローと専門知識を配置する

## 課題一覧

### P0: 初期要件フローの誤分岐

#### 現状

`docs/ideas/initial-requirements.md` は未入力のテンプレートとして最初から存在する。

一方、現在の `define-feature` はファイルの存在有無だけで初期要件作成モードと追加仕様作成モードを判定する。`setup-project` もファイルの存在のみを確認し、必須項目が入力済みかを検証しない。

#### 影響

- 新規プロジェクトでも初期要件作成モードに入らない
- 未入力の初期要件から6つの永続ドキュメントを生成する可能性がある
- ワークフロー上は成功しても、内容のない設計文書が作成され得る

#### 修正方針

次の方針を組み合わせて採用する。

1. 初期要件テンプレートを `define-feature` Skill の `assets/` に移し、初期状態では `docs/ideas/initial-requirements.md` を配置しない
2. `define-feature` と `setup-project` はファイルの存在だけでなく、必須項目が入力済みかを検証してモードを決定する

`setup-project` は、少なくともプロジェクト概要、対象ユーザー、解決課題、主要機能、MVP範囲、受け入れ条件が入力済みであることを確認する。

#### 初期要件の入力済み判定

`define-feature` と `setup-project` は、ファイルの存在だけでなく、以下の内容を検証する。

| 必須項目 | 入力済みとみなす条件 |
| --- | --- |
| Project Overview | Project name、One-sentence summary、Problem to solveが具体的に入力されている |
| Users | Primary usersとUsage contextが具体的に入力されている |
| Product Goals | 実際の目標が1件以上入力されている |
| In Scope | MVPに含める実際の機能が1件以上入力されている。`In Scope`をMVP範囲として扱う |
| Out of Scope | MVPに含めない対象が1件以上入力されている |
| Acceptance Criteria | 検証可能な受け入れ条件が1件以上入力されている |

以下は`blank`として扱う。

- 見出しだけが存在し、本文がない
- コロンの後ろが空である
- `Project name:`、`Goal 1:`、`Core feature 1:`、`Criterion 1:` など、テンプレートのプレースホルダーがそのまま残っている
- テンプレートの説明文だけで、プロジェクト固有の内容がない
- 「使いやすくする」「一般ユーザー向け」など、非空でもプロジェクト固有の対象や検証可能な内容を特定できない一般論だけである

`Open Questions` は未決事項を記録する欄であり、項目が残っていることだけでは停止しない。ただし、永続ドキュメントの内容を大きく変える未決事項がある場合は、main agentがユーザーへ確認してから `setup-project` を続行する。

判定結果は次の3種類とする。

- `missing`: ファイル、必須セクション、または表で要求した必須フィールド・項目が存在しない
- `blank`: 必須セクションと必須フィールド・項目は存在するが、空欄、プレースホルダー、説明文だけ、またはプロジェクト固有性・検証可能性のない曖昧な一般論である
- `valid`: すべての必須項目が具体的に入力されている

複数の状態が混在する場合は、必須フィールド単位で不足理由を列挙したうえで、`missing`、`blank`、`valid` の順に優先する。したがって、必須フィールド不足と空欄が同時にあれば全体は`missing`、不足はないが空欄・placeholder・説明文・曖昧入力が1件でもあれば全体は`blank`、それ以外だけを`valid`とする。

`missing` または `blank` の場合、`setup-project` は永続ドキュメントを生成せず、不足項目を列挙して停止する。`define-feature` は初期要件作成・更新モードを選択する。追加仕様作成モードへ進めるのは初期要件が `valid` の場合だけとする。

`define-feature` は内容判定とは別にファイルの存在状態を記録する。ファイル不存在の `missing` だけはassetから新規作成し、既存ファイルの必須セクション不足は入力済み内容を保持して不足だけを補完する。既存ファイル全体をassetで置換しない。

### P0: `.agents/commands/` が Codex の共有ワークフローとして認識されない

#### 現状

以下のワークフローが `.agents/commands/*.md` に定義されている。

- `init-project`
- `define-feature`
- `setup-project`
- `plan-feature`
- `implement-feature`
- `review-docs`
- `validate-implementation`
- `add-feature`（deprecated）

Codex の現行共有ワークフロー形式は `.agents/skills/<name>/SKILL.md` であり、`.agents/commands/` は自動検出対象ではない。カスタムプロンプトも非推奨である。

#### 影響

- `AGENTS.md` や `README.md` にコマンド名を書いても、Codex がワークフロー本文を自動的に読み込まない
- 実行結果がセッション中の推測や手動ファイル探索に依存する
- CLI、IDE extension、desktop app 間で挙動が安定しない

#### 修正方針

各ワークフローを独立した Skill として移行する。

```text
.agents/skills/
├── init-project/
├── define-feature/
├── setup-project/
├── plan-feature/
├── implement-feature/
├── review-docs/
└── validate-implementation/
```

利用者向けの表記は `$init-project`、`$plan-feature` のような明示的なSkill呼び出しへ統一する。自然文による暗黙呼び出しは、後述のinvocation policyで許可したSkillだけに限定する。

`add-feature` は互換要件がなければ削除する。

#### `review-docs` Skill の入力契約

`review-docs` Skill は、任意の `docs/` 配下のMarkdownファイルを1件以上、明示的な入力として受け取れるようにする。

- ファイルを1件指定した場合は、そのファイルだけをレビューする
- ファイルを複数指定した場合は、指定されたファイル群をまとめてレビューする
- ファイルを指定しない場合は、以下の6つの永続ドキュメントを既定対象とする
  - `docs/product-requirements.md`
  - `docs/functional-design.md`
  - `docs/architecture.md`
  - `docs/repository-structure.md`
  - `docs/development-guidelines.md`
  - `docs/glossary.md`
- 指定ファイルが存在しない場合、または `docs/` 配下でない場合は、レビューを開始せず対象の再指定を求める
- どの入力方法でも、必ず `doc_reviewer` サブエージェントへレビューを委任する
- main agent は `doc_reviewer` の完了を待ち、結果を重大度順に統合して日本語で報告する

想定する呼び出し例:

```text
$review-docs
$review-docs docs/ideas/20260820-codex-environment-modernization.md
$review-docs docs/product-requirements.md docs/functional-design.md
```

### P0: カスタムエージェント定義が現行形式と一致しない

#### 現状

レビュー用エージェントは `.agents/agents/*.md` に Markdown front matter 形式で定義されている。

現在の定義には以下が含まれる。

- `model: inherit`
- `tools: Read, Grep, Glob`
- Markdown 本文による長大な役割定義
- `.codex/config.toml` 内の重複した `instructions`
- `doc-reviewer` と `doc_reviewer` の名称揺れ
- `subagent:` から始まる非公式な疑似 YAML 呼び出し例

#### 影響

- 現行 Codex の公式スキーマと一致しない
- Markdown 内の `tools` を実効的な権限制御と誤認する可能性がある
- 同じ役割が複数箇所に定義され、更新時に内容がずれる
- サブエージェント選択と呼び出しが不安定になる

#### 修正方針

次の構成へ移行する。

```text
.codex/
├── config.toml
└── agents/
    ├── document_author.toml
    ├── feature_planner.toml
    ├── doc_reviewer.toml
    └── implementation_validator.toml
```

各 agent TOML は、少なくとも以下を定義する。

- `name`
- `description`
- `developer_instructions`

必要に応じて以下も設定する。

- `model`
- `model_reasoning_effort`
- `sandbox_mode`

`doc_reviewer` は原則として `sandbox_mode = "read-only"` とする。`implementation_validator` は、テスト実行時に生成物やキャッシュを書き込む必要があるかを確認した上で既定値を決める。agent TOMLの `sandbox_mode` は既定値であり、親ターンのライブなsandbox、approval、managed policyが実効権限へ再適用される場合がある。

`document_author` は仕様書と永続ドキュメントの作成、`feature_planner` は `.steering/` の計画作成を担当する。一般的なコード調査と実装にはCodex組み込みの `explorer` と `worker` を使用する。

`.codex/config.toml` にはagent共通設定だけを置き、個別の役割定義は `.codex/agents/*.toml` に一元化する。

#### custom agent の登録方式

本テンプレートでは、`.codex/agents/*.toml` の自動検出方式へ統一する。

- custom agentごとに `.codex/agents/<agent-name>.toml` を1ファイル作成する
- agent名とファイル名はunderscore形式で統一する
- `.codex/config.toml` の `[agents.<name>]` と `config_file` による明示登録は使用しない
- `.codex/config.toml` は `[agents]` の有効化や同時実行数など、共通設定だけを持つ
- agentの役割、利用条件、禁止事項は各agent TOMLの `developer_instructions` に一元化する
- 新方式の動作確認後、旧 `.agents/agents/*.md` と重複するインラインagent指示を削除する

想定する共通設定:

```toml
[agents]
enabled = true
max_concurrent_threads_per_session = 4
```

この方針により、custom agentの追加・変更・削除は `.codex/agents/` 配下だけで完結させる。

### P0: multi-agent 設定が旧式

#### 現状

`.codex/config.toml` に以下が存在する。

```toml
[features]
collab = true
```

ローカル Codex CLI 0.147.0 の機能一覧では `collab` は現行機能名として表示されず、`multi_agent` が Stable かつ既定有効である。

#### 修正方針

- 明示的に固定する必要がなければ、multi-agent 用 feature flag を削除する
- 明示する場合は現行の `multi_agent` を使用する
- agent数の上限が必要な場合は `[agents]` の `max_concurrent_threads_per_session` を使用する

### P1: `.agents/settings.json` が Codex 設定として機能しない

#### 現状

`.agents/settings.json` に `defaultSkills` が定義されているが、Codex の公式な Skill 自動有効化設定ではない。

#### 修正方針

- `.agents/settings.json` を削除する
- Skills は `.agents/skills/` からの自動検出に任せる
- Skillを無効化する必要がある場合は `.codex/config.toml` の `[[skills.config]]` を使う
- UIメタデータ、暗黙呼び出し可否、MCP依存は各Skillの `agents/openai.yaml` で定義する

### P1: Skill のメタデータと権限境界が曖昧

#### 現状

一部の `SKILL.md` に `allowed-tools: Read, Write` などが記載されている。

この項目は、現行 OpenAI Docs では Codex の権限制御として保証されていない。Skill の安全性をこの値に依存させるべきではない。

#### 修正方針

- `SKILL.md` の必須メタデータは `name` と `description` を中心にする
- `description` に利用条件と非対象範囲を明記する
- ファイル書き込みやコマンド実行の安全境界は sandbox、approval、custom agent、hooks、CI で管理する
- 書き込みまたは高コストな処理を行うSkillは、後述のinvocation policyに従い、`agents/openai.yaml` の `allow_implicit_invocation: false` を設定する

### P1: AGENTS.md と関連文書の重複

#### 現状

以下が `AGENTS.md`、`PROJECT_CONTEXT.md`、`README.md`、`initial-requirements.md` に重複している。

- React Native + Expo + JavaScript
- Node 22
- Expo SDK 54
- `npx expo start`
- `npx expo start --tunnel`
- `npx expo install`
- SDKとNodeを自動更新しない方針

`AGENTS.md` では英語と日本語の内容も重複している。

#### 修正方針

- `AGENTS.md`: Codex が毎回必要とする短い実行規則、ルーティング、制約、完了条件
- `PROJECT_CONTEXT.md`: プロジェクト固有の背景、目的、技術前提
- `README.md`: 人間向けの利用方法とオンボーディング
- Skill: タスク固有の手順、入力、出力、停止条件

`AGENTS.md` は日本語または英語のどちらかを主言語とし、同じ内容の二重記載を避ける。

### P1: Skill が長く、固有例が混入している

#### 現状

`prd-writing/SKILL.md` は長大で、モバイルアプリとは直接関係しない CLI 製品例を含む。

また、アーキテクチャ、機能設計、開発ガイドライン、レビューエージェントの一部に、カード、デッキ、ゲーム進行、音、振動、オフライン完結など、特定のカードゲームを前提にした内容が混入している。

#### 影響

- Skill 選択後のコンテキスト消費が大きい
- 汎用 Expo テンプレートから新規プロジェクトに不要な要件が混入する
- レビュー時に存在しないゲーム要件を必須条件として扱う可能性がある

#### 修正方針

- `SKILL.md` は入力、手順、出力、停止条件、参照先に限定する
- 詳細ガイドと背景情報を `references/` に移す
- コピーして使うテンプレートを `assets/` に移す
- 特定プロダクトの例は `references/examples/` に分離するか削除する
- `development-guidelines` は、ガイドライン文書作成とコード実装支援への分割を検討する

### P1: 品質ゲートと初期化手順が整合していない

#### 現状

`implement-feature` と `implementation-validator` は、以下のコマンドや基準を前提にしている。

- `npm run lint`
- `npm test`
- `npm run test:coverage`
- 固定のカバレッジ基準

しかし `init-project` は、lint、test、coverage の scripts と依存関係を整備することを完了条件に含めていない。

#### 修正方針

以下のどちらかを明確に選ぶ。

1. `init-project` Skill で標準品質基盤を構築する
2. 実装・検証 Skill が `package.json` の存在する scripts を検出し、利用可能な検証だけを実行する

固定のカバレッジ閾値は、`docs/development-guidelines.md` などでプロジェクト要件として合意済みの場合だけ適用する。

### P1: `docs/ideas/` の用途と配置が矛盾している

#### 現状

`docs/ideas/` は仕様専用と定義されているが、`20260407-feature-spec-workflow-notes.md` は AI ワークフロー改修用の作業メモである。また、Claude Code 固有の記述や解消済みの検討事項を含んでいる。

#### 修正方針

- 本仕様のように今後の変更入力として扱う間は `docs/ideas/` に置いてよい
- 変更完了後、安定した運用判断を `docs/decisions/` または適切な永続ドキュメントへ移す
- 解消済みの作業メモを仕様ファイルとして残し続けない

### P2: `.steering/` と Codex Plan mode の関係が曖昧

#### 現状

`.steering/` はリポジトリに残るタスク計画として有用だが、Codex の Plan mode やセッション内プランとの関係が定義されていない。

`tasklist.md` は完了と未完了を中心にしており、ユーザー判断待ちや外部要因によるブロックを十分に表現できない。

#### 修正方針

- `.steering/` をリポジトリに残す計画と進捗の正本とする
- Codex Plan mode は `.steering/` 作成前の調査、対話、計画ドラフトに使用する
- taskには `pending`、`in-progress`、`done`、`blocked`、`cancelled` を表現できる状態を持たせる
- 実行したlint、test、起動確認などの検証証跡を `tasklist.md` に残す

### P2: 実装ワークフローが過度に直列化されている

#### 現状

`implement-feature` は1タスクずつの実装を強く要求している。安全性は高い一方、互いに独立した調査、テスト、実装まで一律に直列化する。

#### 修正方針

- 書き込み競合する実装は直列化する
- 読み取り中心の調査、テスト、ログ解析、独立した検証はサブエージェントへの委任を許可する
- main agent は要求、判断、統合、最終検証に集中する
- 複数agentが同じファイルを同時編集しないよう所有範囲を明示する

## multi-agent の基本方針

main agentに調査、文書作成、実装、検証を集中させると、長いファイル内容、コマンド出力、テストログ、中間検討がmain threadへ蓄積する。これによるコンテキスト汚染を避けるため、main agentはオーケストレーターとして振る舞い、境界の明確な実作業をサブエージェントへ委任する。

### main agentの責務

- ユーザーとの対話と不足情報の確認
- 要求、制約、優先順位、スコープの決定
- サブエージェントへ渡すタスクと所有範囲の定義
- 複数agentの結果の統合と矛盾解消
- 変更差分と検証結果の最終確認
- 完了判定とユーザーへの報告

main agentは、長い生ログや全調査結果を保持しない。サブエージェントからは、結論、根拠となるファイル、変更概要、実行した検証、未解決事項を要約して受け取る。

サブエージェントは追加のトークンと実行時間を消費するため、単純で短い一段階の作業まで一律に委任しない。調査量、ログ量、専門性、独立性、並列化効果のいずれかが十分にある、境界の明確な作業を委任対象とする。

### custom agentの責務

| agent | 主な責務 | 論理的な所有範囲 | 推奨sandbox既定値 |
| --- | --- | --- | --- |
| `document_author` | `docs/ideas/` の仕様と `docs/` の永続ドキュメントを作成・更新する | 委任時に指定された文書だけ | `workspace-write` |
| `feature_planner` | 仕様、永続ドキュメント、調査結果から `.steering/` の3文書を作成・更新する | 指定された `.steering/[YYYYMMDD]-[task]/` だけ | `workspace-write` |
| `doc_reviewer` | 指定された文書をレビューし、重大度順にfindingを返す | 読み取りだけ。ファイルを変更しない | `read-only` |
| `implementation_validator` | 実装と仕様の整合性、テスト、品質を検証する | 読み取りと検証だけ。コードを変更しない | `read-only`を基本とし、テスト実行要件に応じて決定する |

論理的な所有範囲はagentへの指示であり、`workspace-write`がパス単位で強制する権限境界ではない。パス制限を機械的に強制する必要がある場合は、permission profile、hook、CIなどを別途検討する。main agentは作業後の最終diffを確認し、所有範囲外の変更がないことを検証する。

### 組み込みagentの責務

| agent | 主な責務 |
| --- | --- |
| `explorer` | コードベース、関連文書、既存実装を読み取り中心で調査し、要点を返す |
| `worker` | 所有範囲を明示された初期化、実装、テスト追加、限定的な修正を行う |

組み込みagentはCodexが提供するため、`.codex/agents/*.toml` を追加しない。custom agentは役割ごとに1つのTOMLを作成し、同じ役割の複数インスタンスで再利用する。

### Skillごとの委任方針

| Skill | main agentが担当すること | 委任先と委任内容 |
| --- | --- | --- |
| `$init-project` | 技術制約、作成範囲、完了条件の確認 | `worker`がExpo基盤と開発環境を作成する |
| `$define-feature` | ユーザーへの質問、要求とスコープの決定 | `document_author`が確定内容から仕様ファイルを作成・更新する |
| `$setup-project` | 文書作成順序、前提、文書間の整合性を管理する | `document_author`が永続文書を作成し、`doc_reviewer`がレビューする |
| `$plan-feature` | 入力仕様、計画範囲、最終計画を確認する | `explorer`が関連コードと文書を調査し、`feature_planner`が `.steering/` を作成する |
| `$implement-feature` | taskの分割、workerの所有範囲、統合、最終確認を担当する | `explorer`が必要な調査を行い、`worker`が競合しない実装・テストを担当する |
| `$review-docs` | 対象を確認し、レビュー結果を統合する | `doc_reviewer`が指定文書をレビューする |
| `$validate-implementation` | 検証対象を確認し、結果と次の判断を統合する | `implementation_validator`が実装を検証する |

### 既存の専門Skillとの関係

`document_author` は独自の文書構成を作らず、既存の専門Skillを明示的に使用する。

既存の専門Skillである `$prd-writing`、`$functional-design`、`$architecture-design`、`$repository-structure`、`$development-guidelines`、`$glossary-creation`、`$steering` は、すべて明示呼び出し専用とする。各Skillの `agents/openai.yaml` に `policy.allow_implicit_invocation: false` を設定し、main agentまたは委任先agentが必要なSkill名を明示して使用する。

`$setup-project` は、文書間の依存関係に従って次の順序で実行する。

1. `document_author` が `$prd-writing` を使用して `docs/product-requirements.md` を作成する
2. `doc_reviewer` がPRDをレビューし、main agentがユーザーの承認を得る
3. `document_author` が `$functional-design` を使用して `docs/functional-design.md` を作成する
4. `document_author` が `$architecture-design` を使用して `docs/architecture.md` を作成する
5. `document_author` が `$repository-structure` を使用して `docs/repository-structure.md` を作成する
6. `document_author` が `$development-guidelines` を使用して `docs/development-guidelines.md` を作成する
7. `document_author` が `$glossary-creation` を使用して `docs/glossary.md` を作成する
8. `doc_reviewer` が6文書の完全性と相互整合性をレビューする
9. main agentがレビュー結果、未解決事項、最終差分を統合して報告する

文書ごとに新しい `document_author` インスタンスを使用してよい。同じcustom agent TOMLを再利用し、各インスタンスには対象の出力ファイルと参照すべき先行文書だけを渡す。これにより、個々のsubagent threadのコンテキストを文書単位に保つ。

`$steering` は次のように使用する。

- `feature_planner` は `$plan-feature` で `.steering/` を作成するときに使用する
- `worker` は `$implement-feature` で `tasklist.md` と実装状態を同期するときに使用する
- `implementation_validator` は `$validate-implementation` で対象 `.steering/` の要求、設計、進捗を検証コンテキストとして使用する。ただし検証時には `.steering/` を変更しない
- main agentはtaskの状態、agentの所有範囲、検証証跡が整合していることを最終確認する

### Skillの呼び出し方針

予期しない書き込みや高コストな処理を避けるため、Skillごとに暗黙呼び出しの可否を固定する。

| Skill | invocation policy | 理由 |
| --- | --- | --- |
| `$init-project` | 明示呼び出し専用 | リポジトリへアプリ基盤と設定ファイルを作成するため |
| `$define-feature` | 明示呼び出し専用 | 仕様ファイルを作成・更新し、対話フローを開始するため |
| `$setup-project` | 明示呼び出し専用 | 複数の永続ドキュメントを作成し、複数agentを使用するため |
| `$plan-feature` | 明示呼び出し専用 | `.steering/` を作成するため |
| `$implement-feature` | 明示呼び出し専用 | コードとtasklistを変更するため |
| `$review-docs` | 暗黙呼び出しを許可 | 読み取り専用レビューであり、「この文書をレビューして」という自然文に対応するため |
| `$validate-implementation` | 明示呼び出し専用 | テスト実行と専門agentによる高コストな検証を開始するため |

明示呼び出し専用のSkillでは、`agents/openai.yaml` に `policy.allow_implicit_invocation: false` を設定する。`review-docs` は暗黙呼び出しを許可し、自然文のレビュー依頼と明示的な `$review-docs` の両方に対応する。

`allow_implicit_invocation: false` が制御するのはSkillの暗黙選択だけであり、main agentが自然文を通常タスクとして処理することまで機械的に禁止する設定ではない。`AGENTS.md` に「明示呼び出し専用ワークフローと同等の自然文依頼を受けた場合は、書き込みや高コストな処理を開始せず、対応する `$skill-name` の明示を求める」と定義する。さらに強い機械的制御が必要な場合は、approval、hook、permission policyの追加を別途検討する。

### 委任時のルール

- サブエージェントへは目的、入力ファイル、制約、出力形式、完了条件を明示する
- 委任プロンプトには作業に必要な情報、パス、制約、完了条件だけを明記し、無関係な文書や生ログを再掲しない
- 使用中のクライアントがcontext fork範囲を明示的に選べる場合だけ、作業に必要な最小範囲を指定する。選べない場合は、委任プロンプトの情報量を必要最小限に保つ
- 書き込みを行うagentには所有ファイルまたは所有ディレクトリを明示する
- 同じファイルを複数agentへ同時に編集させない
- 調査、テスト、ログ解析など独立した読み取り作業は可能な範囲で並列化する
- 依存関係がある文書や実装は順序を守り、前段の成果物を確認してから次へ進む
- サブエージェントは生ログ全文ではなく、結論、根拠、変更内容、検証結果、残課題を返す
- main agentはサブエージェントの報告だけで完了とせず、差分、重要な検証結果、所有範囲外の変更がないことを確認する

## 目標ディレクトリ構成

```text
AGENTS.md
PROJECT_CONTEXT.md

.codex/
├── config.toml
└── agents/
    ├── document_author.toml
    ├── feature_planner.toml
    ├── doc_reviewer.toml
    └── implementation_validator.toml

.agents/
└── skills/
    ├── init-project/
    │   ├── SKILL.md
    │   ├── agents/openai.yaml
    │   └── assets/
    ├── define-feature/
    │   ├── SKILL.md
    │   ├── agents/openai.yaml
    │   └── assets/
    ├── setup-project/
    │   ├── SKILL.md
    │   └── agents/openai.yaml
    ├── plan-feature/
    │   ├── SKILL.md
    │   └── agents/openai.yaml
    ├── implement-feature/
    │   ├── SKILL.md
    │   └── agents/openai.yaml
    ├── review-docs/
    │   ├── SKILL.md
    │   └── agents/openai.yaml
    ├── validate-implementation/
    │   ├── SKILL.md
    │   └── agents/openai.yaml
    ├── prd-writing/
    │   ├── SKILL.md
    │   ├── agents/openai.yaml
    │   ├── references/
    │   └── assets/
    └── ...

docs/
├── ideas/
├── decisions/
└── ...

.steering/
```

省略している `functional-design`、`architecture-design`、`repository-structure`、`development-guidelines`、`glossary-creation`、`steering` にも、それぞれ `agents/openai.yaml` を配置して明示呼び出し専用であることを定義する。

## 移行順序

1. 初期要件フローの誤分岐を修正する
2. `.agents/commands/` を `.agents/skills/` へ移行する
3. `.agents/agents/*.md` を `.codex/agents/*.toml` の自動検出方式へ移行する
4. `.codex/config.toml` を現行形式へ整理する
5. `.agents/settings.json` と非標準メタデータを整理する
6. Skill の本文、references、assets を分割する
7. カードゲーム固有例と CLI 固有例を汎用テンプレートから分離する
8. 初期化と実装検証の品質ゲートを統一する
9. `AGENTS.md`、`PROJECT_CONTEXT.md`、`README.md` の責務を整理する
10. `.steering/` と Plan mode の役割を明文化する
11. 新方式の動作確認後、旧 `.agents/agents/` と `.agents/commands/` を削除する
12. 旧ワークフローメモを決定記録へ整理する

## 移行完了条件（P0）

- [x] P0-01: 主要ワークフローが `.agents/skills/<name>/SKILL.md` として自動検出される
- [x] P0-02: 明示呼び出し専用のワークフローを `$skill-name` で起動できる
- [x] P0-03: `review-docs` を `$review-docs` と自然文の両方で起動できる
- [x] P0-04: カスタムエージェントが `.codex/agents/*.toml` に定義されている
- [x] P0-05: カスタムエージェントが `.codex/agents/*.toml` から自動検出され、`config_file` に依存していない
- [x] P0-06: agentの役割定義が各agent TOMLの `developer_instructions` に一元化されている
- [x] P0-07: `document_author`、`feature_planner`、`doc_reviewer`、`implementation_validator` がそれぞれの名前で自動検出される
- [x] P0-08: main agentがユーザー対話、意思決定、タスク分割、結果統合、完了判定に集中している
- [x] P0-09: 各Skillに本仕様どおりの委任先・所有範囲・停止条件が定義され、代表ゲートでagent routingが確認されている
- [x] P0-10: 書き込みを行うサブエージェントの所有範囲が明示され、同じファイルを複数agentが同時編集しない
- [x] P0-11: サブエージェントが結論、根拠、変更概要、検証結果、残課題を要約してmain agentへ返す
- [x] P0-12: `collab`、`.agents/settings.json`、非公式な `subagent:` 呼び出し例に依存していない
- [x] P0-13: 新規プロジェクトで初期要件作成モードへ正しく入れる
- [x] P0-14: 未入力の初期要件では `setup-project` が停止する
- [x] P0-15: 初期要件の `missing`、`blank`、`valid` が定義どおりに判定される
- [x] P0-16: 明示呼び出し専用Skillと暗黙呼び出し許可Skillに、確定したinvocation policyが設定されている
- [x] P0-17: 7つのワークフローSkillと7つの既存専門Skillの `agents/openai.yaml` が、定義したinvocation policyと一致している
- [x] P0-18: 関連する `README.md` と `.agents/README.md` が新しい呼び出し形式を案内している
- [x] P0-19: 旧 `.agents/agents/` と `.agents/commands/` が削除され、新旧形式が併存していない

## 今回の品質目標（P1）

以下は今回の移行で可能な限り対応する。未完了項目がある場合は、理由と後続タスクを記録する。

- [x] P1-01: Skill の `description` に利用条件と非対象範囲が明記されている
- [x] P1-02: 長大な説明とテンプレートが `references/` と `assets/` に分離されている
- [x] P1-03: 汎用テンプレートからカードゲーム固有要件とCLI製品固有例が除去されている
- [x] P1-04: `init-project` が用意する品質基盤と、実装・検証工程が要求するコマンドが一致している
- [x] P1-05: `AGENTS.md` が短い恒久ルールとルーティングに整理されている

## 移行後の改善バックログ（P2）

以下は移行後も継続して改善できる項目とする。

- [x] `.steering/` が進捗の正本であり、Plan modeとの役割分担が明記されている
- [ ] `.steering/` のtaskで `pending`、`in-progress`、`done`、`blocked`、`cancelled` の代表的な状態遷移を検証する
- [ ] 書き込み競合のない実装タスクで、複数workerによる安全な並列化を検証する
- [ ] subagentの利用によるコンテキスト削減効果と追加トークン消費を代表ケースで比較する
- [ ] `$setup-project` をvalid入力から6文書の最終レビュー完了まで通すE2E fixtureを実行する

P0-09は移行したSkillのrouting契約と代表的な委任ゲートを対象とし、生成する製品文書そのものの全工程E2Eは対象外とする。5状態の定義と更新規則は`steering` Skillへ実装済みだが、`blocked`や`cancelled`を含む代表taskの状態遷移fixtureは未実行である。custom agent TOML群とWorkflow Skill群を所有範囲別に並列作成した実績はあるが、複数の組み込み`worker`がアプリ実装を分担する代表ケースも未検証である。`setup-project`はPRD作成・レビュー・承認待ち停止までを確認済みだが、残り5文書と最終6文書レビューは未検証であるため、これらをP2の未完了項目として残す。

## 実装・検証証跡

詳細な実行結果は[対応steeringの検証証跡](../../.steering/20260823-codex-environment-modernization/tasklist.md#検証証跡)を正本とする。

| ID | 対応する検証証跡 | 日付 | 結果 |
| --- | --- | --- | --- |
| P0-01 | `新規セッションのSkill検出` | 2026-08-23 | 成功 |
| P0-02 | `新規セッションのSkill検出`、`init-project候補構成`、`plan-feature候補構成`、`implement-feature候補構成`、`validate-implementation候補構成` | 2026-08-23〜24 | 成功 |
| P0-03 | `review-docs入力契約`、`新規セッションのSkill検出` | 2026-08-23〜24 | 成功 |
| P0-04 | `最終静的確認` | 2026-08-24 | 成功 |
| P0-05 | `custom agent起動`、`削除後smoke test` | 2026-08-23〜24 | 成功 |
| P0-06 | `最終静的確認` | 2026-08-24 | 成功 |
| P0-07 | `custom agent起動` | 2026-08-23 | 成功 |
| P0-08 | `plan-feature候補構成`、`implement-feature候補構成` | 2026-08-23〜24 | 成功 |
| P0-09 | `Workflow Skills静的検証`、`Workflow委任先マトリクス`、`init-project候補構成`、`既存初期要件の部分不足fixture`、`setup-project委任ゲート`、`plan-feature候補構成`、`implement-feature候補構成`、`validate-implementation候補構成`、`review-docs入力契約` | 2026-08-23〜25 | 成功 |
| P0-10 | `並列書き込みの所有範囲`、`implement-feature候補構成` | 2026-08-23〜24 | 成功 |
| P0-11 | `plan-feature候補構成`、`implement-feature候補構成` | 2026-08-23〜24 | 成功 |
| P0-12 | `実リポジトリ旧形式削除`、`最終静的確認` | 2026-08-24 | 成功 |
| P0-13 | `初期要件fixture` | 2026-08-23 | 成功 |
| P0-14 | `初期要件fixture`、`複合初期要件fixture`、`曖昧初期要件fixture` | 2026-08-23〜25 | 成功 |
| P0-15 | `初期要件fixture`、`既存初期要件の部分不足fixture`、`複合初期要件fixture`、`曖昧初期要件fixture` | 2026-08-23〜25 | 成功 |
| P0-16 | `全Skill構造検証`、`新規セッションのSkill検出` | 2026-08-23 | 成功 |
| P0-17 | `全Skill構造検証` | 2026-08-23 | 成功 |
| P0-18 | `最終静的確認` | 2026-08-24 | 成功 |
| P0-19 | `実リポジトリ旧形式削除`、`削除後smoke test` | 2026-08-24 | 成功 |
| P1-01 | `Workflow Skills静的検証`、`全Skill構造検証` | 2026-08-23 | 成功 |
| P1-02 | `Workflow Skills静的検証`、`全Skill構造検証` | 2026-08-23 | 成功 |
| P1-03 | `全Skill構造検証` | 2026-08-23 | 成功 |
| P1-04 | `init-project候補構成`、`implement-feature候補構成`、`validate-implementation候補構成` | 2026-08-24 | 成功 |
| P1-05 | `最終静的確認` | 2026-08-24 | 成功 |

## 受け入れ条件の検証方法

| 確認対象 | 操作 | 期待結果 |
| --- | --- | --- |
| Skillの自動検出 | 新規Codexセッションを開始し、`/skills` または `$` のSkill選択を確認する | `init-project`、`define-feature`、`setup-project`、`plan-feature`、`implement-feature`、`review-docs`、`validate-implementation` が表示される |
| 明示呼び出し専用Skillの正の起動 | 安全なfixtureまたは検証用worktreeで `$init-project`、`$define-feature`、`$setup-project`、`$plan-feature`、`$implement-feature`、`$validate-implementation` をそれぞれ実行する | 各Skillが明示的に選択され、定義済みの入力検証、委任、停止条件に従う |
| Skillの明示呼び出し | `$review-docs docs/ideas/20260820-codex-environment-modernization.md` を実行する | 指定した1ファイルだけがレビュー対象になる |
| Skillの引数なし呼び出し | `$review-docs` を実行する | 存在する6つの永続ドキュメントが既定のレビュー対象になる。必要な文書が不足している場合は不足内容を報告して停止する |
| Skillの複数入力 | `$review-docs docs/product-requirements.md docs/functional-design.md` を実行する | 指定した2ファイルだけがまとめてレビューされる |
| Skillの自然文呼び出し | 「この仕様書をレビューして」のような代表的な依頼を行う | 暗黙呼び出しを許可した `review-docs` が選択される |
| 明示呼び出し専用Skillの自然文入力 | `init-project`、`define-feature`、`setup-project`、`plan-feature`、`implement-feature`、`validate-implementation` に相当する自然文を入力する | `allow_implicit_invocation: false` により対象Skillは暗黙選択されない。main agentは `AGENTS.md` の運用ルールに従い、書き込みや高コストな処理を開始せず、対応する `$skill-name` の明示を求める |
| 既存専門Skillの起動方針 | 7つの既存専門Skillの `agents/openai.yaml` を確認し、自然文と明示的な `$skill-name` の両方を代表ケースで試す | 全Skillに `policy.allow_implicit_invocation: false` があり、自然文では暗黙選択されず、責任を持つagentが名前を明示した場合だけ選択される |
| custom agentの検出 | `document_author`、`feature_planner`、`doc_reviewer`、`implementation_validator` をそれぞれ使用する代表タスクを実行する | 指定した名前のcustom agentが起動し、別のagentへ誤って委任されない |
| setup-project全工程（P2） | `$setup-project` のagent activity、各委任プロンプト、生成ファイルを確認する | `document_author` がPRDから用語集まで既定の依存順で専門Skillを使用し、PRDレビューと最終6文書レビューが行われる。未実施のためP2に残す |
| steering Skillの適用 | `$plan-feature`、`$implement-feature`、`$validate-implementation` の代表ケースを確認する | `feature_planner`、`worker`、`implementation_validator` が定義された役割で `steering` を使用し、validatorは `.steering/` を変更しない |
| main agentのコンテキスト分離 | `$setup-project`、`$plan-feature`、`$implement-feature` の代表ケースで委任プロンプト、agent activity、返却要約、main agentの最終報告を確認する | 委任プロンプトには必要なパス、制約、完了条件だけが含まれ、サブエージェントは要約を返し、main agentは判断と統合を扱う |
| 書き込み所有範囲 | 複数workerを使う実装ケースで各agentへの指示と最終diffを確認する | 各agentに所有ファイルまたは所有ディレクトリが明示され、編集対象が重複せず、最終diffに所有範囲外の変更がない |
| レビュー委任 | `$review-docs <対象ファイル>` を実行してagent activityを確認する | `doc_reviewer` が起動し、main agentが完了を待って結果を統合する |
| 初期要件のmissingケース | 初期要件ファイルまたは必須セクションが存在しない状態で `$setup-project` を実行する | `missing` と判定し、永続ドキュメントを生成せず、不足項目を報告して停止する |
| 初期要件のblankケース | 空欄またはプレースホルダーが残る初期要件で `$setup-project` を実行する | `blank` と判定し、永続ドキュメントを生成せず、未入力項目を報告して停止する |
| 初期要件の複合ケース | 必須フィールド不足と、空欄・placeholder・曖昧入力が混在する初期要件で `$setup-project` を実行する | 必須フィールドごとの理由を報告し、優先順位により全体を`missing`と判定して停止する |
| 初期要件の曖昧入力ケース | 必須フィールドはすべて存在するが、プロジェクト固有性または検証可能性のない一般論だけを含む初期要件で `$setup-project` を実行する | 全体を`blank`と判定して停止する |
| 初期要件のvalidケース | すべての必須項目を具体的に満たす初期要件で `$setup-project` を実行する | `valid` と判定し、初期要件を入力として永続ドキュメント作成へ進む |
| 旧形式と旧multi-agent設定の除去 | `.agents/settings.json`、`.agents/agents/`、`.agents/commands/` の不存在を直接確認する。`.codex/`、`AGENTS.md`、`README.md`、`.agents/README.md`、`.agents/skills/` を対象に、TOML設定は `^\s*collab\s*=`、疑似YAMLは `^\s*subagent:\s*$` のパターンで検索する。あわせて `codex features list` を確認する | 旧ファイルと旧ディレクトリが存在せず、実行時に参照される設定・案内・Skillが `collab` や非公式な `subagent:` 呼び出しに依存せず、現行のmulti-agent機能を利用している |
| 利用者向け案内 | `AGENTS.md`、`README.md`、`.agents/README.md` のワークフロー、呼び出し例、agent説明を確認する | `$skill-name` 形式、明示・暗黙呼び出し方針、main agentとsubagentの役割が一致し、削除済みの `.agents/commands/` や `.agents/agents/` を案内していない |
| Codex設定のsmoke test | 新規セッションを開始し、`codex doctor --summary` のConfiguration欄を確認する | project configが読み込める。custom agentとSkillの個別検出は別の起動テストで判定し、外部到達性やdoctor全体の終了コードは合否に使用しない |
| 初期化と品質ゲートの整合性 | 初期化後の `package.json` scriptsと、実装・検証Skillが要求するコマンドを比較する | 要求されるlint、test、coverageコマンドが利用可能か、未導入時の扱いが明記されている |

## スコープ外

- React Native、Expo、JavaScript から別技術スタックへの変更
- Node 22 または Expo SDK 54 のアップグレード
- アプリケーション機能の実装
- MCPサーバーや外部サービス連携の追加
- テンプレートのプラグイン配布対応

## 確定事項と未決事項

- 確定: `init-project` は全プロジェクトへ品質基盤を強制せず、後続工程は `package.json` に存在する品質scriptだけを実行する
- 確定: `implementation_validator` はread-onlyを既定とし、書き込みが必要な検証は未実行理由と必要権限を報告する
- 未決: 汎用Skill群を将来プラグインとして配布するか
