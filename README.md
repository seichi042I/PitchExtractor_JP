# JDC-PitchExtractor
このリポジトリには、[StarGANv2-VC](https://github.com/yl4579/StarGANv2-VC)と[StyleTTS](https://github.com/yl4579/StyleTTS)で使用されている音声変換(VC)とTTS用のディープニューラルピッチ抽出器のトレーニングコードが含まれています。StarGANv2-VCのF0ネットワークとStyleTTSのピッチ抽出器です。

## 前提条件
1. Python >= 3.7
2. Clone this repository:
```bash
git clone https://github.com/seichi042I/PitchExtractor_JP
cd PitchExtractor_JP
```

## Training
```bash
chmod +x run.sh
chmod +x preprocess.sh
./run.sh
```
このコマンドを実行するだけで事前準備から学習まで実行してくれます。 最大24GBのGPU VRAMを消費します。メモリ不足エラーが発生する場合は、Configs/config.ymlのbatch_sizeを減らして下さい。

### IMPORTANT: DATA FOLDER NEEDS WRITE PERMISSION
harvest` も `dio` も比較的遅いので、計算された F0 グラウンドトゥルースを保存しておく必要があります。meldataset.py](https://github.com/yl4579/PitchExtractor/blob/main/meldataset.py#L77-L89)では、各`.wav`ファイルに対して計算されたF0カーブ `_f0.npy`を書き込みます。これには data フォルダの書き込み権限が必要です。

### F0計算の詳細
[meldataset.py](https://github.com/yl4579/PitchExtractor/blob/main/meldataset.py#L83-L87)では、[PyWorld](https://github.com/JeremyCCHsu/Python-Wrapper-for-World-Vocoder)を使って、`harvest`を使った方法と`dio`を使った方法でF0カーブを計算しています。どちらの方法も音響ベースであり、特定の条件下では不安定です。harvest` は `dio` よりも速いが失敗が多いので、まず `harvest` を試す。harvest`が失敗した場合（ゼロでない値を持つフレームの数で判断）、`dio`を用いてグランドトゥルースのF0ラベルを計算する。dio`が失敗した場合、計算されたF0は`NaN`となり、0に置き換えられる。これはたまにしか起こらないはずであり、これらのサンプルはニューラルネットワークによってノイズとして扱われ、ディープラーニングモデルは多少ノイズの多いデータセットでも恩恵を受けることが分かっているため、トレーニングに影響を与えることはないはずである。しかし、多くのサンプルがこの問題を抱えている場合（例えば5%以上）、モデルが失敗したサンプルから学習しないように、それらをトレーニングセットから削除してください。

### データ補強
データ補強はこのコードには含まれていません。より良い音声変換結果を得るためには、[meldataset.py](https://github.com/yl4579/PitchExtractor/blob/main/meldataset.py)と[audiomentations](https://github.com/iver56/audiomentations)に独自のデータ補強を追加してください。

## References
- [keums/melodyExtraction_JDC](https://github.com/keums/melodyExtraction_JDC)
- [kan-bayashi/ParallelWaveGAN](https://github.com/kan-bayashi/ParallelWaveGAN)
- [改変元のリポジトリ](https://github.com/yl4579/PitchExtractor)
