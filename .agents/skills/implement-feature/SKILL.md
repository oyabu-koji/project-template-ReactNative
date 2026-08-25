---
name: implement-feature
description: 指定された.steeringタスクのrequirements、design、tasklistに従って機能を実装し、進捗と検証証跡を同期する。計画済みタスクを実行する場合に明示的に使用する。計画作成、計画なしの実装、独立した最終検証には使用しない。
---

# Implement Feature

`.steering/`を進捗の正本として、所有範囲を分けながらタスクを1件ずつ実装する。

## 入力契約と停止条件

- `.steering/[YYYYMMDD]-[task]/`を1件、明示入力として受け取る。
- 対象が存在しない、`.steering/`外、または`requirements.md`、`design.md`、`tasklist.md`のいずれかが不足する場合は変更を開始しない。
- 要求、設計、taskの矛盾や実装に必要な重要判断がある場合は、main agentがユーザーへ確認する。
- 未コミット変更と所有対象が重なる場合は、既存変更を保護できる方針を確定するまで停止する。

## 実行手順

1. main agentが3文書と関連する永続文書を読み、taskの依存順、受け入れ条件、技術制約を確認する。
2. `$steering`を明示使用し、着手するtaskを`in-progress`として`tasklist.md`へ反映する。一度に進行中にする書き込みtaskは1件にする。
3. 必要な読み取り調査を組み込み`explorer`へ委任する。独立した調査やログ解析は並列化してよいが、書き込みtaskとファイル所有を重ねない。
4. 組み込み`worker`へtaskごとの実装とテストを委任する。所有ファイルまたはディレクトリ、入力、制約、完了条件を明示し、同じコードベースに他の作業者がいること、他者の変更を戻さないことを伝える。
5. 書き込みが競合するtaskは直列に実行する。複数workerを使う場合は所有範囲が重ならないことをmain agentが事前に確認する。
6. workerから結論、変更内容、根拠、実行した検証、残課題の要約を受け取る。main agentがdiff、要求との対応、所有範囲外の変更を確認する。
7. taskの受け入れ条件を満たした場合だけ`done`へ更新する。失敗または外部判断待ちは`blocked`として理由を記録し、次の依存taskへ進まない。
8. 実装が設計を変えた場合は、ユーザーが承認した安定した判断だけを`design.md`へ反映する。task状態と検証証跡は毎task後に`tasklist.md`へ同期する。

## 品質scriptの検出

1. `package.json`の`scripts`を直接確認する。
2. `lint`、`test`、`test:coverage`のうち、存在してtaskに関連するscriptだけを実行する。
3. 未定義scriptは実行せず、不足として報告する。未定義であることだけを失敗にしない。
4. 固定カバレッジ閾値は`docs/development-guidelines.md`で合意済みの場合だけ適用する。
5. 実行したscriptが失敗した場合はtaskを完了扱いにせず、失敗内容と再現方法を記録する。
6. 起動確認が必要な場合は`npx expo start`を使い、リモート端末が必要な場合だけ`npx expo start --tunnel`を使う。

## 完了条件

- tasklistの対象taskが受け入れ条件と検証証跡付きで完了している。
- requirements、design、実装、テストが整合し、未解決事項が明示されている。
- 利用可能な品質scriptだけが実行され、結果が記録されている。
- main agentが最終diffと所有範囲を確認している。
- 最終検証は代行せず、同じsteeringディレクトリを`$validate-implementation`へ渡す。
