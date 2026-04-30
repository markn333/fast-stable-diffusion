# SPEC: fast-stable-diffusion ノートブック群

- 作成日: 2026-04-27
- 最終更新: 2026-04-27
- 対象ファイル:
  - `fast_stable_diffusion_AUTOMATIC1111.ipynb`
  - `fast_stable_diffusion_ComfyUI.ipynb`
  - `fast-DreamBooth.ipynb`
- 動作環境: Google Colab（GPU必須）

---

# AUTOMATIC1111 ノートブック仕様

ファイル: `fast_stable_diffusion_AUTOMATIC1111.ipynb`

- 動作環境: Google Colab（GPU必須）

---

## 概要

AUTOMATIC1111 Stable Diffusion WebUI を Google Colab 上で起動するためのノートブック。
Google Drive にデータを保存し、セッションをまたいでモデル・設定を再利用できる。

---

## セル構成

### セル 1: Connect Google Drive

**目的**: Google Drive をマウントし、モデル等の保存先を確立する

| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `Shared_Drive` | string | `""` | 共有ドライブ名。空の場合は MyDrive を使用 |

**処理フロー**:
1. `google.colab.drive.mount('/content/gdrive')` でマウント
2. `Shared_Drive` が指定されていれば `Shareddrives/` 配下を使用
3. `mainpth` 変数（`MyDrive` or `Shareddrives/{名前}`）を設定

**副作用**: `mainpth` グローバル変数を設定

---

### セル 2: Install/Update AUTOMATIC1111 repo

**目的**: AUTOMATIC1111 WebUI リポジトリを Google Drive へクローン（初回）または更新（2回目以降）

**依存ライブラリ**: `git`, `six`, `tqdm`, `requests`, `base64`

**処理フロー**:
1. Google Drive 未接続の場合はローカルストレージを一時利用
2. WebUI リポジトリが未存在なら `git clone`
3. 存在する場合は `git reset --hard && git pull` で最新化

**保存先**: `/content/gdrive/{mainpth}/sd/stable-diffusion-webui/`

---

### セル 3: Requirements

**目的**: 必要な依存パッケージ・バイナリをインストールする

**主要インストール内容**:

| パッケージ | バージョン | 備考 |
|-----------|-----------|------|
| `gradio` | 削除→再インストール | Python 3.12 パス固定 |
| `wandb` | `0.15.12` | ピン留め |
| `pydantic` | `1.10.2` | ピン留め |
| `numpy` | `1.26` | ピン留め |
| `scipy` | `1.15.3` | ピン留め |
| `controlnet_aux` | 最新 | `--no-deps` |
| `diffusers` | 最新 | `--no-deps` |
| `libtcmalloc` | `gperftools-2.5` | メモリ最適化（初回のみビルド） |

**パッチ処理**:
- `warnings.py`: 警告テキスト出力を無効化
- `pytorch_lightning/__init__.py`: WandbLogger インポートを除去
- `wandb/sdk/lib/retry.py`: `ContextCancelledError` インポートを除去
- `pydantic/typing.py`: `recursive_guard` 引数対応パッチ

**⚠️ 注意**: Python パスが `/usr/local/lib/python3.12/` にハードコード。Python バージョン変更時は要修正。

---

### セル 4: Fix xformers ⭐ 2026-04-27 追加

**目的**: 現在の PyTorch バージョンに合った xformers を再インストールする

**背景**: Colab の PyTorch が `2.9.0 → 2.10.0` にアップデートされたことで既存の xformers バイナリと不一致が発生。

**処理**:
```python
!pip install xformers --upgrade -q
```

---

### セル 5: Model Download/Load

**目的**: Stable Diffusion モデルをダウンロードまたは指定する

| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `Use_Temp_Storage` | boolean | `False` | True: Colab一時領域 / False: Google Drive |
| `Model_Version` | select | `"SDXL"` | プリセットモデル選択 |
| `PATH_to_MODEL` | string | `""` | カスタムモデルのフルパス |
| `MODEL_LINK` | string | `""` | Civitai / HuggingFace / Google Drive リンク |

**対応モデル（プリセット）**:

| バージョン | ファイル名 | ソース |
|-----------|-----------|--------|
| `SDXL` | `sd_xl_base_1.0.safetensors` | HuggingFace |
| `1.5` | `v1-5-pruned-emaonly.safetensors` | HuggingFace |
| `v1.5 Inpainting` | `sd-v1-5-inpainting.ckpt` | HuggingFace |
| `V2.1-768px` | `v2-1_768-ema-pruned.safetensors` | HuggingFace |

**対応ダウンロードソース**: Civitai / Google Drive / HuggingFace / その他URL

---

### セル 6: Download LoRA

**目的**: LoRA モデルをダウンロードする

| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `LoRA_LINK` | string | `""` | Civitai / GDrive / その他リンク。空なら何もしない |

**保存先**: `.../models/Lora/`

---

### セル 7: ControlNet

**目的**: ControlNet 拡張機能とモデルをインストール・更新する

| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `XL_Model` | select | `"None"` | XL 用モデル選択（None/All/Canny/Depth/Sketch/OpenPose/Recolor） |
| `v1_Model` | select | `"None"` | v1.x 用モデル選択 |
| `v2_Model` | select | `"None"` | v2.x 用モデル選択 |

**拡張機能**: `sd-webui-controlnet`（Mikubill製）を GDrive にインストール

---

### セル 8: Start Stable-Diffusion

**目的**: WebUI を起動してブラウザからアクセス可能な URL を発行する

| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `Ngrok_token` | string | `""` | ngrok トークン。空なら Gradio 共有URLを使用 |
| `User` | string | `""` | Gradio Basic 認証ユーザー名（任意） |
| `Password` | string | `""` | Gradio Basic 認証パスワード（任意） |

**起動オプション**:
```
--api --disable-safe-unpickle --enable-insecure-extension-access
--no-download-sd-model --no-half-vae --xformers
--disable-console-progressbars --skip-version-check
```

**公開方式**:
- ngrok トークンなし → `--share`（Gradio 共有 URL）
- ngrok トークンあり → ngrok トンネル経由

---

## 既知の問題・制限事項

| # | 問題 | 影響 | 対応状況 |
|---|------|------|---------|
| 1 | Python パスが `python3.12` にハードコード | Python バージョン変更時に全パッチが失敗 | **対策案あり（下記参照）** |
| 2 | gdown v6 破壊的変更 | すべての gdown 呼び出しが失敗する | **重大・対策案あり（下記参照）** |
| 3 | `RuntimeError: dictionary changed size during iteration` | モデル切り替え初回に発生するがリトライで成功 | A1111側の問題・許容中 |

### 問題 2 詳細：gdown v6 破壊的変更

**gdown v6.0.0 リリース日**: 2026年4月中旬（Colab が最新版を自動使用する場合は影響あり）

| 項目 | 変更内容 | 該当ライン |
|------|--------|---------|
| `get_url_from_gdrive_confirmation` | v5 で deprecated、**v6 で完全削除** | L.222 |
| `gdown.download(..., fuzzy=True)` | `fuzzy` パラメータ **v6 で削除**（自動検知に） | L.380, 392, 441, 449 |
| `!gdown --fuzzy ...` | `--fuzzy` CLI フラグ **v6 で削除** | L.332 |
| `download()` 戻り値 | 失敗時に `None` → **`DownloadError` を raise** | 要レビュー |

**対策案**:
1. `from gdown.download import get_url_from_gdrive_confirmation` を削除 → `get_name()` の gdrive 処理を `gdown.parse_url()` で書き換え
2. `gdown.download(..., fuzzy=True)` → `fuzzy=True` を削除（v6 では URL から自動証別）
3. `!gdown --fuzzy ...` → `--fuzzy` を削除
4. `download()` の失敗判定: `if os.path.exists(model)` で確認しているため影響小小（そのまま容許）

**最も安全な短期対応**: 要件セルで `pip install gdown==5.2.1` をピン留め（v6 以前の最新安定版）

**実装タスク**: TODO.md T-009 として登録済み

**影響箇所**: Requirements セル内の `sed -i` / `rm -r` / `wget` コマンド（計 9 箇所）

```
/usr/local/lib/python3.12/dist-packages/gradio*        ← rm -r
/usr/local/lib/python3.12/dist-packages/tensorflow*    ← rm -r
/usr/lib/python3.12/warnings.py                        ← sed -i
/usr/local/lib/python3.12/dist-packages/pytorch_lightning/...
/usr/local/lib/python3.12/dist-packages/wandb/...
/usr/local/lib/python3.12/dist-packages/pydantic/...
/usr/local/lib/python3.12/dist-packages/gradio/blocks.py  ← wget
```

**対策案**: セル冒頭で Python バージョンを変数に格納し、`!` コマンド内で `$変数名` として展開する。
IPython/Jupyter では Python 変数を直接シェルコマンドに渡せる。

```python
# Requirements セル冒頭に追加
import sys
pyver = f"python{sys.version_info.major}.{sys.version_info.minor}"

# 以降の sed/rm/wget で python3.12 → $pyver に置き換え
# Before:
!sed -i '...' /usr/local/lib/python3.12/dist-packages/pydantic/typing.py
# After:
!sed -i '...' /usr/local/lib/$pyver/dist-packages/pydantic/typing.py
```

**実装コスト**: Requirements セルの先頭に 2 行追加 + 同セル内の `python3.12` を `$pyver` に置換（正規表現で一括可能）

**リスク**: 低。IPython の `$変数名` 展開は公式機能であり、Colab でも動作する。

**実装タスク**: TODO.md T-008 として登録済み

---

## 動作確認済み環境

| 日付 | PyTorch | CUDA | Python | 状態 |
|------|---------|------|--------|------|
| 2026-04-27 | 2.10.0+cu128 | 12.8 | 3.12.13 | ✅ 動作確認（xformers fix適用後） |

---

# ComfyUI ノートブック仕様

ファイル: `fast_stable_diffusion_ComfyUI.ipynb`

## 概要

ComfyUI（ノードベースの Stable Diffusion GUI）を Google Colab 上で起動するためのノートブック。
ngrok トンネル経由でブラウザアクセスを提供する。**ngrok トークンが必須。**

---

## セル構成

### セル 1: Connect Google Drive

A1111 版と同一仕様。`mainpth` 変数を設定する（`MyDrive` or `Shareddrives/{名前}`）。

---

### セル 2: Install/Update ComfyUI repo

**目的**: ComfyUI リポジトリを Google Drive へクローンまたは最新化する

**処理フロー**:
1. `TheLastBen/diffusers` をローカルにクローン
2. `/content/gdrive/{mainpth}/ComfyUI` を `git clone`（初回）or `git pull`（2回目以降）
3. キャッシュディレクトリを初期化し `TRANSFORMERS_CACHE` / `TORCH_HOME` を設定

**保存先**: `/content/gdrive/{mainpth}/ComfyUI/`

---

### セル 3: Requirements

**目的**: 依存パッケージをインストールする

| パッケージ | バージョン | 備考 |
|-----------|-----------|------|
| `diffusers` | 最新 | `-U` |
| `accelerate` | 最新 | `-U` |
| `transformers` | 最新 | `-U` |
| `av` | 最新 | |
| `comfyui-frontend-package` | 最新 | |
| `comfyui-workflow-templates` | 最新 | |
| `alembic` | 最新 | |
| `wandb` | `0.15.12` | ピン留め |
| `pydantic` | `2.10.5` | ピン留め（A1111版と異なり v2系） |
| `numpy` | `1.26` | ピン留め |
| `scipy` | `1.15.3` | ピン留め |
| `libtcmalloc` | `gperftools-2.5` | メモリ最適化（初回のみビルド） |

**パッチ処理**:
- `warnings.py`: 警告テキスト出力を無効化
- `jax/_src/deprecations.py`: AttributeError raise 抑制
- `pydantic/typing.py`: `recursive_guard` 引数対応

**A1111版との差分**:
- pydantic が **v2.10.5**（A1111は v1.10.2）
- tensorflow パッケージを `rm -r` で削除
- `wandb` の uninstall が先頭ではなく pip install 時に行う

**⚠️ 注意**: Python パスが `/usr/local/lib/python3.12/` に複数箇所ハードコード。

---

### セル 4: Model Download/Load

**目的**: ComfyUI 用モデルをダウンロードまたはカスタムリンクから取得する

| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `Use_Temp_Storage` | boolean | `False` | True: Colab一時領域 / False: Google Drive |
| `Model_Version` | select | `"SDXL"` | プリセットモデル選択 |
| `MODEL_LINK` | string | `""` | Civitai / HuggingFace / Google Drive リンク |

**対応モデル（プリセット）**:

| バージョン | ファイル名 | ソース |
|-----------|-----------|--------|
| `SDXL` | `sd_xl_base_1.0.safetensors` | HuggingFace / stabilityai |
| `1.5` | `v1-5-pruned-emaonly.safetensors` | HuggingFace / runwayml |
| `v1.5 Inpainting` | `sd-v1-5-inpainting.ckpt` | HuggingFace / runwayml |
| `flux` | `flux1-dev-fp8.safetensors` | HuggingFace / lllyasviel |

**保存先**: `/content/gdrive/{mainpth}/ComfyUI/models/checkpoints/`（または `/content/temp_models/`）

**対応ダウンロードソース**: Civitai / Google Drive / HuggingFace / その他URL

---

### セル 5: Download LoRA

**目的**: LoRA モデルをダウンロードする

| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `LoRA_LINK` | string | `""` | Civitai / GDrive / その他リンク。空なら何もしない |

**保存先**: `/content/gdrive/{mainpth}/ComfyUI/models/loras/`

---

### セル 6: Start ComfyUI

**目的**: ComfyUI サーバーを起動し ngrok トンネル経由でアクセス URL を発行する

| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `Ngrok_Token` | string | `""` | **必須**。空の場合は起動不可 |

**起動コマンド**:
```
python /content/gdrive/MyDrive/ComfyUI/main.py --listen --port 666
```

**ngrok 接続**: ポート `666` に ngrok トンネルを張り、コンソールに URL を表示

**⚠️ 注意**: A1111版とは異なり **ngrok トークンが必須**（Gradio 共有URLフォールバックなし）。
また `server.py` を sed で直接パッチして URL ログ出力をカスタマイズしているため、ComfyUI 本体のアップデートによって sed パターンが無効になるリスクがある。

---

## ComfyUI 既知の問題・制限事項

| # | 問題 | 影響 | 対応状況 |
|---|------|------|---------|
| 1 | Python パスが `python3.12` にハードコード | Python バージョン変更時にパッチ失敗 | **対策案あり（A1111 SPEC 参照）** |
| 2 | ngrok 必須（Gradio フォールバックなし） | ngrok トークンなしでは起動不可 | 仕様 |
| 3 | `server.py` に直接 sed パッチ| ComfyUI アップデートで無効化の可能性 | 要監視 |
| 4 | gdown v6 破壊的変更 | gdown 呼び出しが失敗する | **重大・対策案あり（A1111 SPEC 参照）** |

---

## ComfyUI 動作確認済み環境

| 日付 | 状態 |
|------|------|
| 2026-04-27 | 解析済み（動作確認は TODO） |

---

# DreamBooth ノートブック仕様

ファイル: `fast-DreamBooth.ipynb`

## 概要

Stable Diffusion モデルを DreamBooth 手法でファインチューニングするためのノートブック。
特定の人物・物体・スタイルを少数枚（10枚程度）の画像から学習できる。SD 1.5 / v2.1 をサポート。

---

## セル構成

### セル 1: Connect Google Drive

A1111/ComfyUI 版とほぼ同一。**`mainpth` は `MyDrive` 固定**（Shared Drive 切り替え機能なし）。

---

### セル 2: Dependencies

**目的**: DreamBooth 学習に必要な依存パッケージをインストールする

| パッケージ | バージョン | 備考 |
|-----------|-----------|------|
| `accelerate` | `0.12.0` | ピン留め（特定バージョン必須） |
| `gradio` | `3.16.2` | ピン留め（古いバージョン） |
| `wandb` | `0.15.12` | ピン留め |
| `pydantic` | `1.10.2` | ピン留め |
| `numpy` | `1.26` | ピン留め |
| `scipy` | `1.15.3` | ピン留め |
| `TheLastBen/diffusers` | main | git clone |
| `libtcmalloc` | `gperftools-2.5` | メモリ最適化 |

**パッチ処理**:
- `diffusers/utils/dynamic_modules_utils.py`: `cached_download` インポート除去
- `pytorch_lightning/loggers/__init__.py`: `WandbLogger` インポート除去
- `wandb/sdk/lib/retry.py`: `ContextCancelledError` 関連除去
- `pydantic/typing.py`: `recursive_guard` 引数対応

---

### セル 3 (markdown): Model Download ヘッダー

---

### セル 4: Model Download

**目的**: ファインチューニングのベースモデルをダウンロードまたは指定する

| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `Model_Version` | select | `"1.5"` | プリセットモデル選択 |
| `Path_to_HuggingFace` | string | `""` | HuggingFace の `profile/model` 形式 |
| `MODEL_PATH` | string | `""` | ローカルモデルファイルの絶対パス |
| `MODEL_LINK` | string | `""` | Civitai / GDrive / その他リンク |

**対応モデル（プリセット）**:

| バージョン | ダウンロード先 | 備考 |
|-----------|-----------|------|
| `1.5` | `v1-5-pruned-emaonly.safetensors` / HuggingFace | diffusers 形式に変換 |
| `V2.1-512px` | HuggingFace / stabilityai/stable-diffusion-2-1-base | sparse checkout |
| `V2.1-768px` | HuggingFace / stabilityai/stable-diffusion-2-1 | sparse checkout |

**HuggingFace プライベートモデル認証**:
`/content/gdrive/MyDrive/Fast-Dreambooth/token.txt` にトークンを記述すると自動認証。

**保存先**: `/content/stable-diffusion-v1-5/`（または v2-512, v2-768, stable-diffusion-custom）

---

### セル 5 (markdown): DreamBooth ヘッダー

---

### セル 6: Create/Load a Session

**目的**: 学習セッションを作成または再開する

| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `Session_Name` | string | `""` | セッション名（必須。スペース→アンダースコアに変換） |
| `Session_Link_optional` | string | `""` | 別のgdriveからセッションをインポートするリンク |

**セッション管理**:
- `SESSION_DIR` = `/content/gdrive/MyDrive/Fast-Dreambooth/Sessions/{Session_Name}/`
- 既存セッションがあれば訓練済みモデルを diffusers 形式に変換・ロード
- 新規の場合はディレクトリ作成のみ

---

### セル 7: Instance Images

**目的**: 学習用インスタンス画像をアップロード・管理する

| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `Remove_existing_instance_images` | boolean | `True` | 既存画像を削除してから追加 |
| `IMAGES_FOLDER_OPTIONAL` | string | `""` | フォルダパス指定（空ならアップロード） |
| `Smart_Crop_images` | boolean | `True` | 自動クロップ有効化 |
| `Crop_size` | select | `512` | クロップサイズ（512〜1024） |

**処理**:
- アップロードまたはフォルダ指定後、Smart Crop で自動リサイズ
- `.txt` ファイルはキャプションとして `CAPTIONS_DIR` に分類

**画像命名規則**: `{identifier} (N).{ext}` 形式（例: `phtmejhn (1).jpg`）

---

### セル 8: Captions（任意）

**目的**: インスタンス画像ごとにキャプションを手動作成・編集する GUI

- ipywidgets を使った画像選択UI
- テキストエリアで各画像のキャプションを編集・保存
- 顔学習時はキャプション不要

---

### セル 9: Training / Start DreamBooth

**目的**: DreamBooth ファインチューニングを実行する

| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `Resume_Training` | boolean | `False` | 前回モデルから継続訓練 |
| `UNet_Training_Steps` | number | `1500` | UNet の訓練ステップ数（0で無効化） |
| `UNet_Learning_Rate` | select | `2e-6` | UNet 学習率 |
| `Text_Encoder_Training_Steps` | number | `350` | テキストエンコーダの訓練ステップ数（0で無効化） |
| `Text_Encoder_Learning_Rate` | select | `1e-6` | テキストエンコーダ学習率 |
| `Offset_Noise` | boolean | `False` | スタイル学習時は有効推奨 |
| `External_Captions` | boolean | `False` | `.txt` キャプションを使用する |
| `Resolution` | select | `"512"` | 学習解像度（512〜1024） |
| `Save_Checkpoint_Every_n_Steps` | boolean | `False` | 中間チェックポイント保存 |
| `Save_Checkpoint_Every` | number | `500` | 何ステップごとに保存するか |
| `Start_saving_from_the_step` | number | `500` | 何ステップ目から保存開始するか |
| `Disconnect_after_training` | boolean | `False` | 訓練完了後に Colab を自動切断 |

**学習フロー**:
1. UNet のみ or テキストエンコーダ込みで訓練
2. 中間ステップ: `OUTPUT_DIR` に diffusers 形式で保存
3. 最終: diffusers → `.ckpt` に変換して `SESSION_DIR` に保存

**出力先**: `/content/gdrive/MyDrive/Fast-Dreambooth/Sessions/{Session_Name}/{Session_Name}.ckpt`

---

## DreamBooth 既知の問題・制限事項

| # | 問題 | 影響 | 対応状況 |
|---|------|------|---------|
| 1 | Python パスが `python3.12` にハードコード | Python バージョン変更時にパッチ失敗 | **対策案あり（A1111 SPEC 参照）** |
| 2 | `gradio==3.16.2` が古い | Gradio の新しいAPIと非互換の可能性 | 要監視 |
| 3 | accelerate==0.12.0 が古い | 新しい PyTorch との互換性リスク | 要監視 |
| 4 | gdown v6 破壊的変更 | gdown 呼び出しが失敗する | **重大・対策案あり（A1111 SPEC 参照）** |
| 5 | mainpth が MyDrive 固定 | Shared Drive を利用不可 | 仕様 |

**DreamBooth 固有の gdown v6 破壊的変更**:

| 項目 | 対象ライン |
|------|--------|
| `!gdown --fuzzy "$MODEL_LINK" -O $modelnm` | L.370 |
| `!gdown --folder --remaining-ok -O ...` | L.495（`--remaining-ok` も v6 で削除） |

---

## DreamBooth 動作確認済み環境

| 日付 | 状態 |
|------|------|
| 2026-04-27 | 解析済み（動作確認は TODO） |
