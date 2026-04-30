---
applyTo: "**/*.cs,**/*.csproj,**/*.razor,**/*.cshtml"
---

# C# コーディング規約

### C# 規約

- 準拠バージョン: **.NET 6 以降 / C# 10 以降**
- 命名規則（Microsoft コーディング規約準拠）:
  - クラス・構造体・プロパティ・メソッド・イベント: `PascalCase`
  - インターフェース: `I` プレフィックス + `PascalCase`（例: `IAudioProcessor`）
  - プライベートフィールド: `_camelCase`（例: `_sampleRate`）
  - ローカル変数・引数: `camelCase`
  - 定数: `PascalCase`
- `var` は型が明確な場合のみ使用する。`new()` ターゲット型推論を積極的に使う
- `async`/`await` を使用する場合、メソッド名に `Async` サフィックスを付ける
- `null` 許容参照型（Nullable Reference Types）を有効化し、`?` を適切に使用する
- LINQ は可読性が上がる場合のみ使用。複雑なネストは禁止
- `string` 結合には `StringBuilder` または補間文字列 `$""` を使用する（`+` 連結ループは禁止）
