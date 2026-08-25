# JavaScript・React Native・Expo実装ガイド

## 基本方針

- 新規コードはJavaScriptで実装し、公開されるデータ契約は必要に応じてJSDocで補う
- 画面は表示とイベント受付、hook/controllerは状態遷移、serviceは端末・外部I/O、純粋logicはフレームワーク非依存の処理を担当する
- Expo APIを利用する場合は、利用不可・権限拒否・端末差異を扱う
- 既存の `docs/development-guidelines.md` に別の境界があれば、そちらを優先する

## 命名とファイル

- 変数・関数: `camelCase`
- component: `PascalCase`
- hook: `use` で始める
- 真偽値: `is`、`has`、`can`、`should` で意図を示す
- screen: `PascalCaseScreen.jsx`
- component: `PascalCase.jsx`
- hook、service、utility: `camelCase.js`
- test: `*.test.js` または `*.test.jsx`

## JSDoc

入力、出力、制約、副作用、状態値をコードから判断しにくい場合に記載する。型注釈だけを目的に過剰なコメントを追加しない。

```javascript
/**
 * @typedef {Object} UserProfile
 * @property {string} id
 * @property {string} displayName
 */

/**
 * @param {UserProfile} profile
 * @returns {string}
 */
function formatDisplayName(profile) {
  return profile.displayName.trim();
}
```

## エラーと品質

- 入力不正は呼び出し側が判定できる明示的なエラーにする
- recoverableな端末機能失敗はUI全体を壊さず、必要な通知とログを残す
- エラー表示とログへ秘密情報を含めない
- 不要な権限を要求しない
- 高コスト処理と不要な再描画を測定せずに最適化しない

## テスト

- 純粋logicはunit testを優先する
- componentは利用者に見える振る舞いを検証する
- Expo API境界はadapterを通し、成功・拒否・失敗を検証する
- test runnerやmock APIはプロジェクトの `package.json` と既存規約に従う
