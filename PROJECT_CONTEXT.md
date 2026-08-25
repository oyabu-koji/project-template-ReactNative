# Project Context

## 目的

このリポジトリは、AI駆動でReact Nativeモバイルアプリを新規開発するための再利用テンプレートである。アプリケーション固有の要件と実装は含めず、仕様作成、設計、実装、検証を一貫して進めるCodex開発環境を提供する。

## 技術前提

- App type: React Native mobile application
- Framework: Expo managed workflow
- Language: JavaScript（TypeScriptは既定で使用しない）
- Package manager: npm
- Node: 22
- Expo SDK: 54

## 開発コマンド

- 通常起動: `npx expo start`
- リモート端末確認: `npx expo start --tunnel`
- Expo関連依存の追加: `npx expo install <package>`

## 制約

- Expo SDKとNodeのバージョンは明示依頼なしに変更しない
- Expo関連依存にはExpo互換バージョンを使用する
- `.devcontainer/` は任意の将来用構成であり、Docker利用を必須にしない
- プロジェクト固有の要件は `docs/ideas/initial-requirements.md` から開始し、安定後は `docs/` の永続文書を正本とする
