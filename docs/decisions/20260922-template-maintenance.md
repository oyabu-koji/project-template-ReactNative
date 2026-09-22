# テンプレート保守の決定

- Date: 2026-09-22
- Status: accepted

Expo SDK基準を54から57へ更新する。根拠は[公式発表](https://expo.dev/changelog/sdk-57)と[SDK資料](https://docs.expo.dev/versions/v57.0.0/)。Node・Expoの指定は`PROJECT_CONTEXT.md`を正本とし、Skill・agent等の重複した固定値を正本参照へ置き換える。

ユーザー指定によりNode基準を22から24へ更新し、`PROJECT_CONTEXT.md`、`.nvmrc`、開発コンテナ、`README.md`の指定を揃える。

`define-feature`では、明示パス`docs/ideas/initial-requirements.md`の不存在を新規作成として許可する。既存要件はユーザーが合意した変更と整合に必要な箇所を更新し、それ以外の入力済み内容を保持する。

[既存の責務分離](20260823-ai-development-workflow.md)を継承し、自然文によるWorkflow起動方針は変更しない。過去の仕様・steeringの検証結果は当時の記録として維持する。本記録は保守方針の決定であり、検証完了を示すものではない。
