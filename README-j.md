# NFC Detect

NFC Detectは、ラベル付き音声データから音声分類モデルを学習するためのツールです。
このツールは、[Google Perch](https://github.com/google-research/perch/tree/main)や
[BirdNET](https://github.com/kahst/BirdNET-Analyzer)などの、
鳥の鳴き声データで学習された公開されている音声ニューラルネットワークモデルを活用します。
事前学習済みモデルを使用して音声入力から埋め込みを計算し、それを分類モデルの入力特徴量として使用します。

このプロジェクトの主な目的は、渡り鳥の夜間飛行時の鳴き声を検出することです。

## インストール

`poetry add`を使用して依存関係にnfc-detectを追加できます。

```
poetry add git+https://github.com/yoshikiyo/nfc-detect.git
```

## データセットの準備

データセットは音声ファイルとそれに関連付けられたアノテーションで構成されています。

* ファイル名
* アノテーション
  * ラベル
  * 開始時間
  * 持続時間

アノテーションデータはCSV形式である必要があります。各レコードには以下の4つのフィールドが必要です。他のフィールドは無視されます。

* `filename` (str): 音声ファイルへのパス（ディレクトリからの相対パス）
* `label` (str): セグメントのラベル
* `start` (float): `filename`フィールドで指定された音声ファイル内のセグメントの開始時間（秒）
* `duration` (float): セグメントの持続時間（秒）

以下は例です。
```
filename,label,start,duration
01.wav,call,10.1,0.5
01.wav,noise,23.7,0.7
...
02.wav,xxx,xx,xx
02.wav,xxx,xx,xx
```

次に、アノテーションと音声ファイルをTFRecord形式のデータセットに変換する必要があります。
以下のコマンドは、音声からアノテーションされたセグメントを抽出し、埋め込みモデルを適用して、
データをTFRecordファイルに保存します。

```
python3 -m nfcdetect prepare_data \
  --annotation_file <アノテーションCSVファイルへのパス> \
  --data_dir <音声ファイルを格納するディレクトリへのパス> \
  --embedding_model <birdnetまたはperch> \
  --output <出力ファイル名>
```

例：
```
python3 -m nfcdetect prepare_data \
  --annotation_file my_annotation.csv \
  --data_dir /path/to/audio/files \
  --embedding_model birdnet \
  --output my_data_train.tfrecords
```

## 学習

以下のように`train`コマンドでモデルを学習できます：
```
python3 -m nfcdetect train \
  --train_set <学習データとして使用するtfrecordsファイルへのパス> \
  --output <学習済みモデルを保存するファイル名> \
  --embedding_model <使用する埋め込みモデル> \
  --classifier_type <svcまたはlogistic_regression> \
  --regularization <正則化項（svcとlogistic_regressionで共通）>
```

例えば、以下のコマンドは`my_data_train.tfrecords`を使用してSVCモデルを学習します。成功すると、`/path/to/my_classifier.joblib`と`/path/to/my_classifier_config.json`の2つのファイルが生成されます。
```
python3 -m nfcdetect train \
  --train_set my_data_train.tfrecords \
  --output /path/to/my_classifier \
  --embedding_model birdnet \
  --classifier_type svc \
  --regularization 1.0
```

## 推論

学習済みモデルは`analyze`コマンドを使用して読み込み、推論に使用できます：

```
python3 -m nfcdetect analyze \
  --classifier_config /path/to/my_classifier_config.json \
  --input /path/to/input/audio_file.flac \
  --start 0.0 \
  --duration 60.0
```

これは、分類器を`audio_file.flac`の最初の60秒に1秒間隔で適用します。分類結果は現在のディレクトリの`nfc_output.csv`に書き込まれます。`--output`オプションを指定することで、結果を書き込むファイルを変更できます。
