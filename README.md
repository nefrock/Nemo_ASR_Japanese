# Nemo_ASR_Japanese
Prepare Japanese dataset for nvidia NeMo ASR models





#### 注意点

- 特殊記号 ('÷', '―', '△', '○', '々', '〇', '〒', '％', '＆', '＋', '／') や英字を含む文はデータから排除する

- CSJでは数字は漢数字で表記しており,統一するため他のコーパスで英数字を含む文はデータから排除する
- 英字を含む文はデータから排除する
- 句読点やカギカッコは無視する
- spectrogram augmentationを行うと落ちるので,行わないようにした



## 学習済みモデル

trained_models/QuartzNet5x5/2021-02-03_15-16-12:  CSJで学習

trained_models/QuartzNet5x5/2021-02-04_01-44-40:  CSJ+JNAS+JVSで学習

trained_models/QuartzNet15x5/2021-01-29_07-37-24: CSJで学習 かなりアンダーフィット気味



#### 推論

- 学習中のモデル(.ckpt形式)で推論したい: inference/inference_ckpt.ipynb
  学習開始時に読み込んだconfigファイルが必要. 学習開始時に結果保存ディレクトリにコピーするようにしておくといいかもしれない (TODO)
- 学習済みのモデル(.nemo形式)で推論したい: inference/inference_nemo.ipymb



#### 対応コーパスとその配置

- CSJ:  /opt/storage/datasets/audio/japanese/CSJ
- JNAS: /opt/storage/datasets/audio/japanese/JNAS
- JVS: /opt/storage/datasets/audio/japanese/jvs_ver1

NeMo本体は/src/NeMoに置いてあるとする.

- DNS: /opt/storage/datasets/audio/noise/DNS-Challenge (https://github.com/microsoft/DNS-Challenge)

#### 使い方

0. conf以下を/src/NeMo/examples/asr/confにコピーする
   conf/config-*-template.yamlを予め環境に合わせて編集しておくと良い.
   16GBのメモリを持つGPUの場合, QuartzNet5x5でバッチサイズ32, QuartzNet15x5でバッチサイズ14が安全.

1. prepare_manifests以下のprepare_{csj, jvs, jnas}.ipynbを実行することで,コーパスのトップディレクトリ直下にmix_train_manifest.json, mix_val_manifest.jsonが生成される.  ここでmixは漢字かな混じりのこと. jsonの各行に

   > {
   >                 "audio_filepath": path,
   >                 "duration": duration,
   >                 "text": text
   >             }

   という形式で各サンプルのメタデータが格納される.
   CSJは各サンプルが長いので台本ファイルの行区切りをもとにwavファイルを分割して保存する.

2. prepare_conf/gather.ipynbを実行すると,  
   NeMo/examples/asr/conf/mix_train_manifest.json, NeMo/examples/asr/conf/mix_val_manifest.json　が生成される.

3. prepare_conf/generate_conf.ipynbを実行すると,
   準備したコーパスに対応したconfigファイルが生成される.

4. NEMO/examples/asr/speech_to_text.py, L.69, config_nameを生成したconfigの名前に書き換えてから,python3 speech_to_text.pyを実行する. 学習結果はNEMO/examples/asr/nemo_examples以下に保存される.