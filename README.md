# Tensorflow

このリポジトリは墓村用のTensorflowのメモです。

## 環境構築

メモリ8G以上のマシン(VM)を用意すること。
Ubuntu 16.04(amd64)を準備して下記のコマンドで構築する。CentOSは面倒なのでやらないこと。

    $ sudo apt-get update
    $ sudo apt-get install -y python-pip3
    $ sudo pip3 install --pre tensorflow

標準のTensorflowはSSE/AVXに対応されておらず、糞遅い。
SSE/AVXに対応したバイナリを利用したい場合は、GitHubにAVE/SEEに対応したバイナリを
アップロードしているので、一度上記手順でtensorflowをインストールした後で、
下記の手順で置き換える。

    $ git clone https://github.com/gravevillage/tsflow-memo
    $ cd tsflow-memo/tensorflow_pkg
    $ sudo pip3 install tensorflow-1.3.0rc2-cp35-cp35m-linux_x86_64.whl

CUDAには対応していないので、CUDAを利用したい場合は要相談。

## サンプルの取得

Tensorflowのサンプルはhttps://github.com/tensorflow/modelsリポジトリに色々あるが、
ここでは、translate(翻訳)サンプルを利用する。オリジナルのサンプルは仏語から日本語へ
の翻訳のサンプルだが、他の言語や、質問から回答を得るような、2文間の対応関係の学習に
利用することができる(はずである)。

Gitから下記のコマンドでサンプルを取得する(1.3.0rc2で動作するようにバグ修正してある)。

    $ git clone https://github.com/gravevillage/tsflow-memo

下記のコマンドで、tesorflowのサンプルのディレクトリに移動して準備完了

    $ cd tsflow-memo/translate

## 学習

例えば、フランス語のテキストを1文1行でtext.frを用意する。書く行に対応する英訳を書いたテキストをtext.enとして用意する。
学習コマンドのサンプルは下記の通り。

    $ python3 translate.py --data_dir /mnt/data --train_dir /mnt/train --from_train_data /mnt/data/text.fr --to_train_data /mnt/data/text.en --en_vocab_size=10000 --fr_vocab_size=10000 --size=256 --num_layers=2 --steps_per_checkpoint=50 --max_train_data_size=50000 >& log &

QAとして利用する場合は、--from_train_dataで質問文を、--to_train_dataで回答文を記述したファイルを用意して実行する。

コマンドのオプションの意味は下記の通り。

```
オプション             説明
-------------------------------------------
--fr_vocab_size        入力データの語彙数
--en_vocab_size        出力データの語彙数
--num_layers           ニューラルネットワークの層数
--size                 ニューラルネットワークのサイズ
--data_dir             データ出力ディレクトリ(抽出した単語を加工したデータが出力される)
--train_dir            学習モデルを出力するディレクトリ
--from_train_data      学習に利用する入力データのリストファイル
--to_train_data        学習に利用する出力データのリストファイル
--max_train_data_size  トレーニング利用するデータサイズ。--from/to_train_dataで
                       指定したファイルを先頭からこのパラメータで指定した行数だけ学習に利用する。行
```

### テスト

学習したモデルを利用して、テストを行う。--decodeオプションを付けて下記のコマンドを実行

    $ python3 translate.py --decode --data_dir /mnt/data --train_dir /mnt/train  --size=256 --num_layers=2 

学習時に指定したニューラルネットのサイズ(--size=256)と階層(--num_layers=2)は同じにしないとエラーになるので注意すること。
例えば、学習時に入力にフランス語データ、出力に英語データを利用した場合、コンソール上でフランス語の文章を入力すると、
英語の出力を得る。
