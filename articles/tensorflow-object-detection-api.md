---
title: "Tensorflow Object Detection APIの実行環境を作る"
emoji: "📷"
type: "tech"
topics: ["tensorflow", "object", "detection"]
published: true
slug: tensorflow-object-detection-api
date: 2019-02-07T14:37:00.001Z
---






Tensorflow Object Detection APIの実行環境を作る



## 概要


超便利なGoogle製物体検知API  

Tensorflowがわからなくてもすぐに使えてしまうお手軽物体検知の導入のメモです。


## 構成


GCP


* ゾーン : us-west1-b
* マシンタイプ : n1-standard-8
* OS : Ubuntu 18.04 LTS
* GPU : NVIDIA Tesla K80 x 2  

Driver Version: 410.72  

CUDA Version: 10.0


python 3.5.2  

tensorflow-gpu 1.11.0


## 環境構築


### Tensorflow Object Detection APIのクローン


ソースのクローンですが、**少しコツがあります**。  

そのままクローンしてしまうと、Object Detection APIと関係ないソースまでクローンしてしまいます。  

そこで、`Sparse Checkout`というテクニックを使って、指定のディレクトリだけをクローン、Git管理できるようにします。



```
# cloneしないので.gitが直下にできてしまうため
# 予めディレクトリを作成
cd <path>
mkdir TensorflowObjectDetectionAPI
cd TensorflowObjectDetectionAPI

# Sparse Checkoutの設定
git init
git config core.sparsecheckout true
git remote add origin https://github.com/tensorflow/models.git
echo -e "research/object\_detection\nresearch/slim" > .git/info/sparse-checkout
git pull origin master

```

### Tensorflowのインストール


installationには以下のようにありますが、個人的にオススメはDocker HubのTensorflowイメージを使ったDocker環境の作成です。  

他の環境を汚さず済んで、コマンド一発でリセット、新しいバージョンの導入などが行えます。  

Dockerコンテナの構築方法は[こちら](https://blog.ai-platform.jp/2018/11/gpu.html){:target="\_blank"}の記事を参考にしてください。


Dockerを使用しない場合は、環境に合わせて下記のようにTensorFlowをインストール



```
# For CPU
pip install tensorflow
# For GPU
pip install tensorflow-gpu

```

### 依存ライブラリのインストール



```
sudo apt install protobuf-compiler python-pil python-lxml python-tk
pip install --user cython contextlib2 jupyter matplotlib

```

### COCO APIのインストール


必須ではないようですが、TensorboardでmAP（適合率）やAR（再現率）を見れるようになるので、入れておいて損は無いと思います。



```
cd <path>
git clone https://github.com/cocodataset/cocoapi.git
cd cocoapi/PythonAPI
make
cp -r pycocotools <path>/TensorflowObjectDetectionAPI/research/
# インストールファイルはもういらないので、削除しておく
cd ../../
rm -rf ./cocoapi/

```

### Protobufのコンパイル


[ココ](https://github.com/protocolbuffers/protobuf/releases/latest)から、最新のビルド済みzipファイルを適当な場所にDL＆解凍しておきましょう。



```
unzip protoc-3.6.1-linux-x86_64.zip
ls

bin/
include/
protoc-3.6.1-linux-x86_64.zip
readme.txt

```

解凍後、上記のようなファイルができると思います。  

`readme.txt`にもありますが、パスの通った場所にファイルを置いてあげましょう。



```
sudo mv bin/protoc /usr/local/bin/
sudo mv include/google/ /usr/local/include/

```

余分なファイルは適当に消して、本題のprotocコマンドを入力します。



```
cd TensorflowObjectDetectionAPI/research/
protoc object_detection/protos/*.proto --python_out=.

```

一瞬で終わりますが、`object_detection/protos/`内に`.py`ファイルができていればOKです。


### PYTHONPATHの追加



```
# TensorflowObjectDetectionAPI/research内で
echo "export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim">>~/.bashrc
source ~/.bashrc

```

これでTensorflow Object Detection APIの環境ができました！


## テスト


以下のテストコードを動かして`OK`が出れば成功です！  

お疲れ様でした！



```
cd object_detection
python builders/model_builder_test.py
......................
----------------------------------------------------------------------
Ran 22 tests in 0.232s

OK

```




