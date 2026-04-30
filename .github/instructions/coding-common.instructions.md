---
applyTo: "**"
---

# 共通コーディング規約・スタイル

## 🔤 コーディング規約・スタイル

### 共通規約

- インデントは **スペース4文字**（タブ文字は禁止）
- 行末の空白は不要。ファイル末尾は改行1つ
- 1行の最大文字数は **120文字**
- 文字コードは **UTF-8**（BOMなし。C/C++のヘッダを除く）
- マジックナンバーは必ず名前付き定数・`enum` で定義する

## 💬 コメント・ドキュメント方針

- **"何を"ではなく"なぜ"をコメントに書く**。コードから読み取れることは書かない
- 複雑なアルゴリズム・ビジネスロジックには必ず説明コメントを付ける
- `TODO:` / `FIXME:` / `HACK:` には **担当者・日付・チケット番号** を記載する

### C / C++ ドキュメントコメント（Doxygen形式）

```cpp
/**
 * @brief 音声バッファを処理しFM合成を行う
 *
 * @param[out] buffer    出力先バッファ（呼び出し元が確保）
 * @param[in]  frameCount 処理するフレーム数（1〜MAX_FRAME_COUNT）
 * @param[in]  sampleRate サンプリングレート（Hz）
 * @return 処理成功時は 0、エラー時はエラーコード（ERROR_* 定数参照）
 * @note スレッドセーフではない。呼び出し元で排他制御を行うこと
 */
int32_t fm_process_buffer(
    float*   buffer,
    int32_t  frameCount,
    uint32_t sampleRate
);
```

### C# ドキュメントコメント（XML ドキュメント形式）

```csharp
/// <summary>
/// 音声バッファに対してFM合成処理を行います。
/// </summary>
/// <param name="buffer">出力先バッファ（呼び出し元が確保済みであること）</param>
/// <param name="frameCount">処理するフレーム数（1〜<see cref="MaxFrameCount"/>）</param>
/// <returns>処理結果を格納した <see cref="ProcessResult"/></returns>
public ProcessResult ProcessBuffer(Span<float> buffer, int frameCount)
```
