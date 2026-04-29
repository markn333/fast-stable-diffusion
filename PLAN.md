# PLAN: fast-stable-diffusion メンテナンス

- 作成日: 2026-04-27
- ステータス: **承認済み**

---

## 概要

オリジナルの [TheLastBen/fast-stable-diffusion](https://github.com/TheLastBen/fast-stable-diffusion) は更新が止まっている。
`markn333/fast-stable-diffusion` を独立したメンテナンスリポジトリとして管理し、
Google Colab 環境の変化に追従しながら Jupyter Notebook を継続的に保守する。

---

## 目標

| # | 目標 | 優先度 |
|---|------|--------|
| 1 | ソースコードから仕様書を作成する | 高 |
| 2 | 今回の修正内容（xformers対応）を記録・管理する | 高 |
| 3 | markn333 リポジトリをオリジナルから分離し独立メンテする | 高 |
| 4 | ソースコードを解析し課題・問題を記録管理する | 中 |

---

## フェーズ計画

### Phase 1: 現状把握・仕様書作成（1週間）

| タスク | 内容 |
|--------|------|
| 1-1 | fast_stable_diffusion_AUTOMATIC1111.ipynb の全セル解析 |
| 1-2 | fast_stable_diffusion_ComfyUI.ipynb の全セル解析 |
| 1-3 | fast-DreamBooth.ipynb の全セル解析 |
| 1-4 | SPEC.md（仕様書）の作成 |
| 1-5 | 現時点の既知問題を TODO.md / bugs/ に記録 |

### Phase 2: 独立リポジトリ化（3日）

| タスク | 内容 |
|--------|------|
| 2-1 | オリジナルリポジトリとの差分を整理 |
| 2-2 | upstream（TheLastBen）からの取り込みルールを決める |
| 2-3 | ブランチ戦略・コミット規約の策定 |

### Phase 3: 継続メンテナンス体制の確立（随時）

| タスク | 内容 |
|--------|------|
| 3-1 | Colab 環境変化（PyTorch / CUDA / Python）の定期チェック |
| 3-2 | 修正内容を bugs/ に記録（BUG-NNNN.md 形式） |
| 3-3 | 問題・課題を TODO.md で管理 |

---

## ノートブック サポート方針

| ノートブック | サポート状態 | 方針 |
|------------|------------|------|
| AUTOMATIC1111 | ✅ アクティブメンテ | メインターゲット。継続的に修正・追従する |
| ComfyUI | 🔧 計画中 | PyTorch 2.10 互換性確認・修正を予定（T-010〜T-012）|
| DreamBooth | ❄️ Not Supported（凍結）| メンテナンスなし。旧環境向けに現状維持のまま放置 |

---

## 今回実施済みの修正（記録）

| 修正日 | 内容 | コミット |
|--------|------|---------|
| 2026-04-27 | xformers がPyTorch 2.10.0 で動作しない問題を修正（pip upgrade セル追加） | `8308f00` |

---

## 成果物

| ファイル | 内容 |
|----------|------|
| SPEC.md | ノートブック仕様書 |
| PROJECT.md | プロジェクト管理（目的・体制・ルール） |
| PROGRESS.md | 進捗記録 |
| TODO.md | タスク一覧 |
| bugs/BUG-0001.md | xformers 修正記録 |
| reports/ | 定期レポート格納フォルダ |

---

## リスク

| リスク | 対応 |
|--------|------|
| Colab が Python / PyTorch を突然アップデートする | Fix セルで動的に対応、bugs/ に記録 |
| オリジナルrepoが突然復活してコンフリクトする | upstream のコミットは随時チェック、意図的に分離管理 |
| 依存ライブラリの破壊的変更 | Requirements セルのバージョンピン留めで対応 |
