---
mode: "agent"
description: "試験項目を作成する"
---

# 試験項目作成

選択したコード（または指定されたクラス・関数）に対して、網羅的な試験項目を作成してください。

**テスト命名規則**: `対象_条件_期待結果`
```
例: ProcessBuffer_WithNullBuffer_ReturnsErrorCode
    CalcTax_WithMaxAmount_ReturnsCorrectValue
```

## 🧪 試験項目作成ガイドライン

### テスト命名規則

テスト名は **「対象_条件_期待結果」** の形式で記述する。

```cpp
// C++ (Google Test)
TEST(FmOscillator, ProcessBuffer_WithZeroFrequency_OutputsAllZeros)
TEST(FmOscillator, ProcessBuffer_WithNullBuffer_ReturnsErrorCode)
```

```csharp
// C# (NUnit / xUnit)
[Test]
public void ProcessBuffer_WithZeroFrequency_OutputsAllZeros() { }
```

### 試験項目の網羅観点

各機能に対して以下の観点でテストを作成すること。

1. 正常系（Happy Path）- 典型的な入力値での期待動作
2. 異常系・エラー系 - `null` / 空バッファ・ゼロサイズ・範囲外の値
3. 状態遷移テスト - オブジェクトが複数の状態を持つ場合の遷移
4. 境界値分析 - 最小値・最大値・その周辺値
5. セキュリティテスト - バッファオーバーフロー・インジェクション等

---
