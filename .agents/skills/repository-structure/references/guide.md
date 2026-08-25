# リポジトリ構造ガイド

## 設計原則

- プロジェクト規模と機能境界に合う最小構成にする。
- 変更理由が同じファイルを近くに置く。
- feature固有領域と複数featureで共有する領域を分ける。
- アーキテクチャの依存方向をディレクトリ規則として表現する。
- 現在の構造と将来の目標構造を混同しない。

## 記述項目

各ディレクトリについて次を定義する。

- 責務
- 配置するファイルと配置しないファイル
- 命名規則
- 外部へ公開する入口
- 許可する依存と禁止する依存
- testの配置

## React Native・Expo・JavaScript

- 実際のExpoエントリーポイントとrouting方式を確認する。
- UIは`.jsx`、非UIロジックは`.js`を標準とし、例外を明示する。
- Expo APIや端末I/Oは設計されたPlatform / Service境界へ置く。
- assetの種類、命名、参照方法を定義する。
- TypeScript用構造を明示依頼なしに導入しない。

## 依存ルールの例

```text
screens -> hooks/controllers -> domain
screens -> hooks/controllers -> services
shared -X-> feature-specific modules
domain -X-> React / Expo / platform APIs
```

実際に採用したアーキテクチャへ置き換え、機械的にこの例を固定しない。

## testと文書

- unit / component / integration / e2eのうち、採用した種類の配置を定義する。
- `docs/ideas/`を仕様、`.steering/`をタスク進捗、`docs/`を永続文書として区別する。
- `.agents/skills/`と`.codex/agents/`の役割を混同しない。

## 完成チェック

- ツリーに記載した主要パスの責務が説明されている。
- 命名例が特定機能へ偏っていない。
- 依存禁止がアーキテクチャ文書と一致している。
- test、asset、docs、設定の置き場所が明確である。
