# Development Guidelines

## 基本原則

- [利用者価値と品質の優先順位]
- [技術制約と依存方針]
- [docsとsteeringを同期する方針]

## 開発環境

- Node.js: [version]
- Package manager: [name/version]
- Language: JavaScript
- Runtime: React Native + Expo managed workflow

## コーディング規約

### JavaScript / JSDoc

- [公開契約をJSDocで記載する条件]
- [状態・エラー・副作用の表現]

### 命名とファイル

- 変数・関数: `camelCase`
- component: `PascalCase`
- hook: `use` prefix
- test: `*.test.js` / `*.test.jsx`

### 責務境界

- Screen: [責務]
- Component: [責務]
- Hook / Controller: [責務]
- Logic: [責務]
- Service / Adapter: [責務]

## エラーハンドリング

- [入力不正]
- [端末機能・外部I/O失敗]
- [利用者への表示]
- [ログと秘密情報]

## テスト戦略

- Unit test: [対象]
- Component test: [対象]
- Integration / device test: [対象]

## 品質ゲート

利用可能な `package.json` scriptsを列挙する。

- [ ] lint: [command / 未導入]
- [ ] test: [command / 未導入]
- [ ] coverage: [command / 未導入]
- [ ] Expo起動確認: [必要条件]

## Git・Review

- Branch strategy: [方針]
- Commit convention: [方針]
- Review requirements: [項目]

## Definition of Done

- [ ] 受け入れ条件を満たす
- [ ] 利用可能な品質ゲートを通過する
- [ ] 必要なdocsとsteeringを更新する
- [ ] 未検証事項と残課題を記録する
