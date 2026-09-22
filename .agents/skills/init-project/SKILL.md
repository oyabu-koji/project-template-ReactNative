---
name: init-project
description: Expo managed workflow + JavaScriptの新規プロジェクト基盤をリポジトリへ初期化する。アプリ基盤や共通開発ファイルが未作成で、要件定義より前に環境を整える場合に明示的に使用する。既存アプリの機能実装、TypeScript導入、NodeやExpo SDKのアップグレードには使用しない。
---

# Init Project

新規リポジトリへ、このテンプレートが定めるExpoアプリの最小基盤だけを作成する。

## 入力

- 対象リポジトリのルートを受け取る。省略時は現在のリポジトリルートを使う。
- `AGENTS.md` と `PROJECT_CONTEXT.md` を読み、技術スタック、Node、Expo SDK、禁止事項を正本として扱う。

## 事前確認と停止条件

1. `package.json`、Expo設定、アプリentry point、既存ソース、Git差分を確認する。
2. 既存Expoアプリがある場合は初期化で上書きせず、検出内容と不足だけを報告して停止する。既存環境の補修へ進むには対象をユーザーと合意する。
3. `node --version`と`.nvmrc`、`PROJECT_CONTEXT.md`のNode指定を照合する。不一致なら、導入済みのバージョン管理ツールで指定済みNodeへ切り替え、同じ実行環境で以後のnpm・Expoコマンドを実行する。切り替えられない場合は必要なNodeバージョンと切り替え手順を報告して停止する。プロジェクトの指定バージョンや端末全体の既定値を自動変更しない。
4. 対象ルートが不明、または安全に所有範囲を切り出せない場合は変更を開始しない。

## 実行手順

1. main agentが作成対象、技術制約、完了条件を確定する。
2. 組み込み`worker`へ初期化を委任する。対象リポジトリ内の作成予定ファイルを所有範囲として列挙し、同じコードベースに他の作業者がいること、他者の変更を戻さないこと、機能実装をしないことを伝える。
3. Expo managed workflow、JavaScript、および`PROJECT_CONTEXT.md`に指定されたNode・Expo SDKに適合する最小起動構成を作成する。TypeScriptを導入しない。
4. `.gitignore`、`.nvmrc`、任意利用の`.devcontainer/devcontainer.json`を整える。既存の`.devcontainer/`を削除しない。
5. Expo関連依存の追加には`npx expo install`を使う。Expo SDKやNodeのバージョンを暗黙に変更しない。
6. `package.json`の`scripts`を読み、`lint`、`test`、`test:coverage`の有無を報告する。存在しない品質scriptや特定テストライブラリを一律に追加・必須化しない。
7. 利用可能で安全な起動確認または品質scriptだけを実行し、未実行の確認は理由を示す。
8. workerから結論、変更ファイル、実行した検証、残課題の要約を受け取る。main agentが最終diffと所有範囲外の変更がないことを確認する。

## 完了条件

- Expo managed workflow + JavaScriptの最小起動構成が存在する。
- `.gitignore`、`.nvmrc`、任意利用の`.devcontainer/devcontainer.json`が整っている。
- 利用可能な品質scriptと検証結果が報告されている。
- 要件定義、`.steering/`作成、アプリ機能実装を行っていない。
- 次の操作として`$define-feature`を案内する。
