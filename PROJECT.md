# PROJECT: fast-stable-diffusion

- 作成日: 2026-04-27
- 最終更新: 2026-04-27
- ステータス: **進行中**

---

## プロジェクト概要

オリジナルの [TheLastBen/fast-stable-diffusion](https://github.com/TheLastBen/fast-stable-diffusion) は更新が停止している。
`markn333/fast-stable-diffusion` をフォークした独立リポジトリとして、Google Colab 環境の変化に追従しながら継続的にメンテナンスする。

---

## リポジトリ情報

| 項目 | 内容 |
|------|------|
| 管理リポジトリ | https://github.com/markn333/fast-stable-diffusion |
| オリジナル | https://github.com/TheLastBen/fast-stable-diffusion |
| メインブランチ | `main` |
| 動作環境 | Google Colab（GPU/TPU） |

---

## 管理対象ノートブック

| ファイル | 概要 |
|----------|------|
| `fast_stable_diffusion_AUTOMATIC1111.ipynb` | AUTOMATIC1111 WebUI を Colab で起動 |
| `fast_stable_diffusion_ComfyUI.ipynb` | ComfyUI を Colab で起動 |
| `fast-DreamBooth.ipynb` | DreamBooth ファインチューニング |

---

## 運用ルール

### ブランチ戦略・コミット規約

- バグ修正時は必ず `bugs/BUG-NNNN.md` を作成する
- NNNN は既存ファイル数 +1 のゼロ埋め4桁

### ブランチ戦略

#### ブランチ構成

| ブランチ | 用途 | 備考 |
|---------|------|------|
| `main` | **常時安定**。Colab から直接参照されるブランチ | マージ前に動作確認必須 |
| `fix/{説明}` | バグ修正・互換性対応 | 例: `fix/xformers-pytorch-mismatch` |
| `feat/{説明}` | 新機能・改善 | 例: `feat/add-flux-support` |
| `upstream-sync` | upstream 取込の検証用 | 本体に影響を与えずテスト可能 |

#### 運用フロー

```
upstream/main
      ↓ (必要に応じて upstream-sync ブランチで検証)
  upstream-sync → main
  
  main → fix/xxx → main
  main → feat/xxx → main
```

#### ルール

- **`main` への直接コミットは緊急修正のみ**。通常は `fix/` or `feat/` ブランチを切って PR（または merge）する
- `main` にマージする前に Colab で動作確認を行う
- マージ後は作業ブランチを削除する
- upstream の変更取込は `upstream-sync` ブランチで差分確認を行い、問題がなければ `main` へマージする

#### コミットメッセージ規約

```
feat:  新機能追加
fix:   バグ修正・互換性対応（BUG-NNNN があれば末尾に記載）
docs:  ドキュメント更新
chore: 依存関係・設定変更
```

例: `fix: upgrade xformers for PyTorch 2.10.0 compatibility (BUG-0001)`

### upstream（オリジナル）との関係

- `TheLastBen/fast-stable-diffusion` の変更は随時チェックするが、自動マージは行わない
- 取り込む場合は内容を確認し、Colab 環境への影響を検証してからコミットする

---

## upstream 差分状況

最終確認日: 2026-04-27

### 現状サマリー

| 項目 | 状態 |
|------|------|
| upstream リモート | `https://github.com/TheLastBen/fast-stable-diffusion.git` |
| 共通ベースコミット | `5490c8d` (fix) |
| fork 独自コミット | **1件**（upstream より 1 コミット先行） |
| upstream 未取込コミット | **0件**（upstream から遅れなし） |

### fork 独自の変更（tracked）

| コミット | 変更内容 | 対象ファイル |
|---------|---------|-------------|
| `8308f00` | fix: reinstall xformers to match current PyTorch version on Colab | `fast_stable_diffusion_AUTOMATIC1111.ipynb` |

**変更の詳細**: Requirements セル直後に「Fix xformers」セル（`pip install xformers --upgrade -q`）を追加。PyTorch 2.10.0 への自動アップデートにより xformers バイナリが非互換になる問題を修正。→ 詳細は [bugs/BUG-0001.md](bugs/BUG-0001.md)

### fork 独自の追加ファイル（untracked）

管理用ドキュメント群。upstream には存在しない。

```
.github/
  copilot-instructions.md
  instructions/ (4ファイル)
  prompts/ (4ファイル)
PLAN.md / PROJECT.md / SPEC.md
PROGRESS.md / TODO.md / REQUEST.md
bugs/ / reports/
```

### upstream 未取込コミット（追従候補）

現時点では取込対象なし（fork は upstream と同一ベースの最新状態）。

### upstream 確認コマンド

```powershell
# upstream の最新を fetch
git fetch upstream

# upstream にあって fork にないコミット一覧（追従要確認）
git log --oneline HEAD..upstream/main

# fork にあって upstream にないコミット一覧（自分の独自変更）
git log --oneline upstream/main..HEAD

# ファイル差分の概要
git diff upstream/main HEAD --stat
```

---

## 担当

| 役割 | 担当 |
|------|------|
| メンテナー | markn333 |
