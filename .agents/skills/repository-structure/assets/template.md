# リポジトリ構造定義書

## 適用方針

- 技術前提: React Native + Expo managed workflow + JavaScript
- 構成方針: [feature-first / layer-first / hybridと理由]
- 現在との差分: [新規構成または既存構成の更新]

## プロジェクト構造

```text
project-root/
├── [Expo entry]
├── assets/
├── src/
│   ├── app/
│   ├── features/
│   └── shared/
├── tests/
├── docs/
├── .agents/
├── .codex/
└── .steering/
```

## ディレクトリ詳細

### `[path]/`

- 責務: [内容]
- 配置するもの: [内容]
- 配置しないもの: [内容]
- 命名規則: [内容]
- 公開入口: [内容]
- test配置: [内容]

## ファイル命名

| 種別 | 配置先 | 命名規則 | 汎用例 |
| --- | --- | --- | --- |
| Screen | `screens/` | `PascalCaseScreen.jsx` | `FeatureScreen.jsx` |
| Component | `components/` | `PascalCase.jsx` | `ItemRow.jsx` |
| Hook | `hooks/` | `use*.js` | `useFeatureState.js` |
| Service | `services/` | `camelCase.js` | `storageService.js` |
| Logic | `logic/` | `camelCase.js` | `selectVisibleItems.js` |
| Test | [近傍またはtest領域] | `*.test.js(x)` | `FeatureScreen.test.jsx` |

## 依存関係

```text
[許可する依存方向]
```

禁止事項:
- [禁止する依存]

## Assetと設定

- asset配置・命名: [内容]
- 環境設定: [内容]
- 秘密情報: [管理方針]

## Docs・AIワークフロー

- `docs/ideas/`: [役割]
- `docs/`: [役割]
- `.steering/`: [役割]
- `.agents/skills/`: [役割]
- `.codex/agents/`: [役割]

## 移行上の注意

- [現状から変更する場合の順序と互換性]
