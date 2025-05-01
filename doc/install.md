# NFC Detect インストールガイド

## 前提条件

- Python 3.x
- Poetry（Pythonパッケージ管理ツール）
- Apple Silicon (M1/M2) Macの場合は、適切な環境設定が必要

## インストール手順

1. リポジトリのクローン
```bash
git clone git@github.com:yoshikiyo/nfc-detect.git
cd nfc-detect
```

2. Poetry環境のセットアップ
```bash
poetry shell
poetry install
```

3. モデルファイルのダウンロード
- 以下のURLからモデルファイルをダウンロード
- https://drive.google.com/file/d/1qPhnZErZInYyeQUdyUeUDOdVKnURZtDl/view?usp=sharing
- ZIPファイルには以下の2つのファイルが含まれています：
  - .jsonファイル（設定ファイル）
  - .joblibファイル（モデルファイル）
- 解凍後、2つのファイルを同じディレクトリに配置

## Apple Silicon (M1/M2) Macでの注意点

1. ターミナルの実行環境確認
```bash
arch
```
- `arm64`と表示されることを確認
- `i386`と表示される場合は、Rosetta環境で動作しているため、native環境に切り替えが必要

2. Python環境の確認
- ターミナルで`python`を実行し、native環境で動作していることを確認
- GNU coreutilsがインストールされている場合、PATHから外すことを推奨

## 使用方法

1. 基本的な使用方法
```bash
python3 analyze.py --input=<音声ファイル> --classifier_config=<birdnet_svc_repeat_config.jsonへのパス>
```

2. 出力
- 同じディレクトリに`nfc_output.csv`が生成されます
- ファイルには時刻とNFC検出確率の値が含まれます
- 出力例：
```
position,prob
60.100,0.092339
61.100,0.116616
62.100,0.270325
...
```

## トラブルシューティング

1. **Illegal instruction 4エラー**
   - Apple Silicon Macで発生する可能性があります
   - ターミナルがnative環境で動作していることを確認
   - Pythonがnative環境で動作していることを確認

2. **依存関係のインストールエラー**
   - Poetryのキャッシュをクリアして再試行
   ```bash
   poetry cache clear --all pypi
   poetry install
   ```

3. **モデルのダウンロードエラー**
   - インターネット接続を確認
   - プロキシ設定を確認（必要な場合）

## アンインストール

Poetryを使用してアンインストールする場合：
```bash
poetry remove nfc-detect
```
