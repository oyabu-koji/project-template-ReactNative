# タスクリスト

## 実行ルール

- このファイルを今回の実装進捗の正本とする
- 1つの親タスクを完了するたびに、検証結果とともにチェック状態を更新する
- 書き込みをsubagentへ委任する場合は所有ファイルまたは所有ディレクトリを明示する
- 同じファイルを複数agentへ同時に割り当てない
- 旧形式の削除は、新方式の静的検証と新規セッションでの検出確認後に行う
- スキップする場合は技術的理由を取り消し線付きで記録する

## フェーズ1: 移行前の基準確認

- [x] Git statusと現行ファイル一覧を記録し、ユーザーの既存変更を識別する
- [x] 元仕様のP0・P1受け入れ条件と本計画の対応関係を再確認する
- [x] 現行CodexドキュメントでSkill、`agents/openai.yaml`、custom agent TOML、`[agents]` のスキーマを実装直前に再確認する
- [x] `.gitignore` の `.steering/*` 一律除外を解除し、このtaskディレクトリがGit管理対象として表示されることを確認する
- [x] `/private/tmp` 配下に起動・初期化検証用の使い捨てコピーを用意する手順を決める

## フェーズ2: custom agentと共通設定の移行

- [x] `.codex/agents/document_author.toml` を作成し、指定文書だけを所有する書き込み役として定義する
- [x] `.codex/agents/feature_planner.toml` を作成し、指定 `.steering/` だけを所有する計画役として定義する
- [x] `.codex/agents/doc_reviewer.toml` を作成し、read-onlyの文書レビュー役として定義する
- [x] `.codex/agents/implementation_validator.toml` を作成し、read-onlyの実装検証役として定義する
- [x] 4 agent TOMLの `name`、`description`、`developer_instructions`、sandbox既定値を構文検証する
- [x] 最終形の `.codex/config.toml` 候補を使い捨てコピーへ適用できる形で用意し、実リポジトリの旧インラインagent定義と `collab` はまだ変更しない
- [x] review・validationの必須委任規則がagent TOML、Skill、`AGENTS.md` の間で矛盾しないことを確認する

## フェーズ3: 初期要件と仕様テンプレートの移行

- [x] `.agents/skills/define-feature/assets/initial-requirements-template.md` を作成し、現行テンプレート内容を複製する
- [x] `.agents/skills/define-feature/assets/feature-spec-template.md` を作成し、現行 `.agents/templates/feature-spec-template.md` を複製する
- [x] 初期要件の必須6領域とplaceholder判定を `define-feature` と `setup-project` で同一に参照できる形で記述する
- [x] `missing`、`blank`、`valid` の判定例と停止条件をSkill本文またはreferenceへ定義する
- [x] `define-feature` が必要時にassetから `docs/ideas/initial-requirements.md` を作成するよう参照を切り替え、現行の初期配置ファイルは動的検証まで残す

## フェーズ4: 主要7ワークフローのSkill化

- [x] `.agents/skills/init-project/SKILL.md` を作成し、Expo managed workflow + JavaScriptの初期化範囲、停止条件、`worker`への所有範囲指定を定義する
- [x] `.agents/skills/init-project/agents/openai.yaml` を作成し、暗黙呼び出しを無効化する
- [x] `.agents/skills/define-feature/SKILL.md` を作成し、初期要件・追加仕様・既存仕様更新の分岐と `document_author` 委任を定義する
- [x] `.agents/skills/define-feature/agents/openai.yaml` を作成し、暗黙呼び出しを無効化する
- [x] `.agents/skills/setup-project/SKILL.md` を作成し、初期要件検証、文書依存順、PRD承認、最終レビューを定義する
- [x] `.agents/skills/setup-project/agents/openai.yaml` を作成し、暗黙呼び出しを無効化する
- [x] `.agents/skills/plan-feature/SKILL.md` を作成し、入力契約、`explorer`調査、`feature_planner`所有範囲、計画だけで停止する条件を定義する
- [x] `.agents/skills/plan-feature/agents/openai.yaml` を作成し、暗黙呼び出しを無効化する
- [x] `.agents/skills/implement-feature/SKILL.md` を作成し、task単位の所有、`worker`委任、tasklist同期、利用可能な品質scriptの検出を定義する
- [x] `.agents/skills/implement-feature/agents/openai.yaml` を作成し、暗黙呼び出しを無効化する
- [x] `.agents/skills/review-docs/SKILL.md` を作成し、0件・1件・複数件の入力契約と `doc_reviewer` 必須使用を定義する
- [x] `.agents/skills/review-docs/agents/openai.yaml` を作成し、暗黙呼び出しを許可する
- [x] `.agents/skills/validate-implementation/SKILL.md` を作成し、steering入力、`implementation_validator` 必須使用、非変更、利用可能な品質scriptの検出を定義する
- [x] `.agents/skills/validate-implementation/agents/openai.yaml` を作成し、暗黙呼び出しを無効化する
- [x] 7つの `SKILL.md` のdescriptionに利用条件と非対象範囲が含まれることを確認する

## フェーズ5: 既存7専門Skillの整理

- [x] `prd-writing` の長いガイドを `references/`、テンプレートを `assets/` へ分離し、CLI製品固有例を汎用フローから除去する
- [x] `functional-design` のガイドを `references/`、テンプレートを `assets/` へ移し、参照パスを更新する
- [x] `architecture-design` のガイドを `references/`、テンプレートを `assets/` へ移し、カードゲーム固有例を除去する
- [x] `repository-structure` のガイドを `references/`、テンプレートを `assets/` へ移し、参照パスを更新する
- [x] `development-guidelines` の詳細ガイドを `references/`、テンプレートを `assets/` へ移し、文書作成と実装支援の利用条件を明確にする
- [x] `glossary-creation` のガイドを `references/`、テンプレートを `assets/` へ移し、カードゲーム固有例を除去する
- [x] `steering` のtemplatesを `assets/` として整理し、計画・実装・検証の入力契約と進捗正本の規則を維持する
- [x] 7つの専門Skillから権限制御目的の `allowed-tools` を除去し、descriptionに利用条件と非対象範囲を記載する
- [x] 7つの専門Skillへ `agents/openai.yaml` を追加し、すべて暗黙呼び出しを無効化する
- [x] 6永続文書の作成依存順と、各専門Skillの必須入力・出力パスが一致することを確認する

## フェーズ6: AGENTS・README・関連文書の更新

- [x] `AGENTS.md` を短い恒久規則、正本へのルーティング、`$skill-name` フロー、委任規則、明示呼び出し停止規則に整理する
- [x] `PROJECT_CONTEXT.md` を確認し、AGENTSやREADMEへ重複させない技術前提の正本として整える
- [x] `README.md` の開始手順と例を `$init-project` から `$validate-implementation` までの新形式へ更新する
- [x] `.agents/README.md` をSkill、`agents/openai.yaml`、`.codex/agents/`、references、assetsの案内へ更新する
- [x] `docs/ideas/20260407-feature-spec-workflow-notes.md` の安定した判断を必要に応じて `docs/decisions/` の決定記録へ整理する
- [x] `docs/ideas/` が仕様入力専用であることと、完了済み作業メモの扱いを文書間で統一する

## フェーズ7: 新方式の静的検証

- [x] 全14 Skillの `SKILL.md` front matterを検査する
- [x] 全14 Skillの `agents/openai.yaml` をYAML parserで検査し、invocation policyを一覧照合する
- [x] 4 custom agent TOMLと `.codex/config.toml` をTOML parserで検査する
- [x] 新しいSkillとagentが参照するファイルパスがすべて存在することを確認する
- [x] workflow Skill間の入力・出力・停止条件と委任先を相互照合する
- [x] AGENTS、README類、Skill、agent TOMLで名称がkebab-case / underscoreの規則どおりであることを確認する
- [x] Markdownのリンク、末尾空白、コードブロック、表の崩れを検査する

## フェーズ8: 新規セッションとfixtureによる動作確認

- [x] 使い捨てコピーへ `[agents] enabled = true` と `max_concurrent_threads_per_session = 4` だけを持つ最終形の `.codex/config.toml` を適用し、旧commands・旧agent Markdown・settings・旧template・初期配置された空の初期要件を除いた候補構成を作る
- [x] 新規Codexセッションで7つの主要ワークフローSkillが一覧表示されることを確認する
- [x] 使い捨てコピーで `$init-project` を明示起動し、環境だけを作成して機能実装へ進まないことを確認する
- [x] `$define-feature` のファイル不存在、既存ファイルのセクション不足、blank、valid分岐をfixtureで確認する
- [x] `$setup-project` がmissing・blankで停止し、validで専門Skillの依存順へ進むことを確認する
- [x] `$plan-feature` が対象specからsteeringの3文書だけを作成することを確認する
- [x] `$implement-feature` がtask所有範囲とtasklist同期を守り、存在する品質scriptだけを実行することを確認する
- [x] `$validate-implementation` が `implementation_validator` を使い、コードとsteeringを変更しないことを確認する
- [x] `$review-docs` の引数なし、1件、複数件、無効パスを確認する
- [x] 自然文の文書レビューで `review-docs` が暗黙選択されることを確認する
- [x] 明示専用の13 Skillが自然文で暗黙選択されず、明示した場合は選択されることを代表ケースで確認する
- [x] 4 custom agentを名前指定の代表タスクで起動し、役割とsandbox既定値を確認する
- [x] subagentへの所有範囲、返却要約、main agentの最終diff確認が代表フローで行われることを確認する
- [x] `codex doctor --summary` のConfiguration欄でproject configの読み込みを確認する

## フェーズ9: 旧形式の削除

- [x] フェーズ7と8の成功証跡を確認してから、実リポジトリの `.codex/config.toml` から個別agentの重複定義と `collab` を除去する
- [x] 実リポジトリの `.codex/config.toml` に `[agents] enabled = true` と `max_concurrent_threads_per_session = 4` を設定する
- [x] `.agents/commands/` を削除する
- [x] `.agents/agents/` を削除する
- [x] `.agents/settings.json` を削除する
- [x] assetへ複製済みの `.agents/templates/` を削除する
- [x] assetへ複製済みの `docs/ideas/initial-requirements.md` を削除し、初期状態では存在しないことを確認する
- [x] 決定記録へ整理済みの `docs/ideas/20260407-feature-spec-workflow-notes.md` を削除し、`docs/ideas/` を仕様入力だけにする
- [x] `.codex/`、`AGENTS.md`、`README.md`、`.agents/README.md`、`.agents/skills/` から `^\s*collab\s*=` と `^\s*subagent:\s*$` を検索し、旧依存がないことを確認する
- [x] `.agents/settings.json`、`.agents/commands/`、`.agents/agents/` が存在しないことを直接確認する
- [x] `git check-ignore` で `.steering/20260823-codex-environment-modernization/` の3文書が除外されないことを確認する
- [x] 削除後にSkill一覧、custom agent起動、config smoke testを再実行する

## フェーズ10: 品質確認と引き継ぎ

- [x] Git差分を確認し、今回の仕様・計画外の変更がないことを確認する
- [x] 新旧形式が併存せず、元仕様のP0受け入れ条件がすべて満たされることを確認する
- [x] P1品質目標の実施結果を確認し、未達がある場合は技術的理由を記録する
- [x] 実行した静的検査、fixture、新規セッション、doctorの結果を本ファイルの「検証証跡」へ記録する
- [x] `requirements.md`、`design.md`、`tasklist.md` が実装結果と一致していることを確認する
- [x] 同じ `.steering/20260823-codex-environment-modernization/` を `$validate-implementation` に渡せる状態にする

## 最終検証の是正

- [x] 初期要件のファイル不存在と既存ファイルの必須セクション不足を分岐し、既存内容を保持して不足だけを補完する契約とfixture証跡を追加する

## 最終文書レビューの是正

- [x] 初期要件の必須フィールド不足、blank、曖昧入力が混在する場合の分類優先順位とfixtureを追加する
- [x] P2の状態表現と複数worker並列化について、完了表示を実績・証跡と一致させる
- [x] 決定記録の明示起動規範を、6つの明示専用Workflowと自然文対応のreview-docsに分ける
- [x] 仕様へ完了日・対応steering・P0/P1検証証跡の対応表を追加する

## 検証証跡

実装中に、実行日・操作・結果・未解決事項を追記する。

| 日付 | 検証 | 結果 | 備考 |
| --- | --- | --- | --- |
| 2026-08-23 | 移行前Git status・現行パス | 成功 | 既存変更は未追跡の元仕様 `docs/ideas/20260820-codex-environment-modernization.md`。旧commands、agent Markdown、settings、`collab` 設定が存在 |
| 2026-08-23 | 公式OpenAI Docsスキーマ確認 | 成功 | `.agents/skills/<name>/SKILL.md`、`agents/openai.yaml`、`.codex/agents/*.toml` 必須キー、`[agents]` 共通設定を確認 |
| 2026-08-23 | 使い捨て検証領域 | 成功 | `/private/tmp/codex-modernization-validation.zJ7Wpr` を作成。検証時に最終候補をコピーし、本体の旧形式を先に削除しない |
| 2026-08-23 | Workflow Skills静的検証 | 成功 | 7件のquick_validate、openai.yaml構文・policy、asset一致、TODO・allowed-tools・末尾空白なしを確認。依存は `/private/tmp` のみに導入 |
| 2026-08-23 | 全Skill構造検証 | 成功 | quick_validate 14/14成功。openai.yaml 14件を解析し、`review-docs` のみimplicit=true。製品固有語・allowed-tools・TODO・末尾空白なし |
| 2026-08-23 | Candidate `codex doctor --summary` | 条件付き成功 | Configurationのconfig/auth/sandboxはok。state DB整合性と外部reachabilityは環境側failのため移行判定から除外 |
| 2026-08-23 | 新規セッションのSkill検出 | 成功 | 主要7 workflow Skillを検出。自然文では`review-docs`だけが暗黙選択され、明示指定では残り13 Skillを読み込めた |
| 2026-08-23 | custom agent起動 | 成功 | 通常セッションで4 agentを名前指定して起動し、review/validationはread-only、author/plannerはworkspace-writeを確認 |
| 2026-08-23 | 並列書き込みの所有範囲 | 成功 | custom agent TOML群と主要workflow Skill群を別agentへ同時委任し、所有ディレクトリを分離。main agentが最終差分で重複・範囲外変更なしを確認 |
| 2026-08-23 | 初期要件fixture | 成功 | ファイル不存在のmissingは新規作成、blankは更新、validは既存更新として分類。setup-projectはmissing/blankで停止し、validのみ依存順へ進むことを確認 |
| 2026-08-23 | plan-feature候補構成 | 成功 | feature_plannerへ入力・所有範囲を限定し、対象specからsteering 3文書だけを生成。main agentが成果物と変更範囲を再確認 |
| 2026-08-24 | init-project候補構成 | 成功 | 組み込み`worker`へ最小5ファイルを所有範囲として委任。Node 22.23.2でExpo 54.0.37、React 19.1.0、React Native 0.81.5を生成し、`expo install --check`と公開config成功。docs・steering・機能・TypeScriptなし |
| 2026-08-24 | implement-feature候補構成 | 成功 | T1〜T3を1件ずつ同期し、worker所有外checksum不変。定義済み`npm test` 6件成功、未定義lint/coverageは理由付き未実行 |
| 2026-08-24 | validate-implementation候補構成 | 成功 | implementation_validatorがread-onlyで合格・findingなし。`npm test` 6件成功、検証前後81ファイルのchecksum一致 |
| 2026-08-24 | review-docs入力契約 | 成功 | 引数なしは6文書、1件・複数件は指定対象だけをdoc_reviewerが確認。不存在パスはagent起動前に停止し、全ケース無変更 |
| 2026-08-24 | 実リポジトリ旧形式削除 | 成功 | 動作ゲート通過後に旧commands・agent Markdown・settings・templates・初期要件配置・完了作業メモを削除。Gitから復元可能 |
| 2026-08-24 | 削除後smoke test | 成功 | `codex doctor --summary` は17 ok・0 fail。新規セッションで主要7 Skillを検出し、doc_reviewerをread-onlyで起動 |
| 2026-08-24 | 最終静的確認 | 成功 | TOML 5件、YAML 14件、Skill 14件、参照パス、`git diff --check`、旧依存検索、steering追跡対象を再確認 |
| 2026-08-24 | 既存初期要件の部分不足fixture | 成功 | `present + missing` を新規セッションの `$define-feature` で`document_author`へ対象1ファイルを委任して更新。保持マーカーを含む既存5領域を維持し、不足していたAcceptance Criteriaだけを追加。対象外ファイルのchecksum不変 |
| 2026-08-24 | High finding是正後の最終検証 | 合格 | `implementation_validator` がread-onlyで再検証しfindingなし。対象外55ファイルの前後checksum一致、Git status・index不変、staged差分なし |
| 2026-08-25 | 複合初期要件fixture | 成功 | 新規セッションの`$setup-project`で、必須フィールド不足1件とblank・曖昧入力の混在を`present + missing`と判定。フィールド別理由を報告して生成前に停止し、変更なし |
| 2026-08-25 | 曖昧初期要件fixture | 成功 | 必須フィールドは存在するが一般論だけの入力を`present + blank`と判定。全9項目の理由を報告して生成前に停止し、変更なし |
| 2026-08-25 | setup-project委任ゲート | 成功 | valid fixtureで`document_author`が`$prd-writing`を使いPRDだけを作成し、`doc_reviewer`がPRDだけをread-onlyレビュー。main agentが所有範囲を確認し、残り5文書を作らずユーザー承認待ちで停止 |
| 2026-08-25 | Workflow委任先マトリクス | 成功 | `init-project→worker`、`define-feature→document_author`、`setup-project→document_author/doc_reviewer`、`plan-feature→feature_planner`、`implement-feature→worker`、`review-docs→doc_reviewer`、`validate-implementation→implementation_validator`をSkill定義と代表実行で照合。各書き込み先またはread-only境界も一致 |
| 2026-08-25 | 最終文書レビュー | 合格 | `doc_reviewer`が対象仕様と決定記録をread-onlyで再レビューし、新規findingなし。P0-09の代表ゲート、P2の全6文書E2Eとの境界、実装完了日と追加検証日の区別が整合 |

## 実装後の振り返り

### 実装完了日

2026-08-24

### 計画と実績の差分

- 初回の使い捨て候補にはGitメタデータとNode 22がなかったため、変更範囲はchecksumで監査し、`init-project` はNode 22ランタイムとネットワークを明示した別fixtureで完了確認した
- `codex doctor` の候補コピーでは環境側エラーがあったが、旧形式削除後の実リポジトリでは17 ok・0 failになった

### 学んだこと

- custom agentを名前指定する場合、full-history forkとagent type指定は併用できないため、必要な入力・所有範囲・完了条件を独立コンテキストへ明記する
- review/validationは前後checksumを取ることでread-only契約を明確に検証できる

### 次回への改善提案

- 元仕様のP2項目は今回の実装完了後に別タスクとして評価する
