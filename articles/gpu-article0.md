---
title: "最短でGPU機械学習環境を構築する！"
emoji: "🚀"
type: "tech"
topics: ["gpu"]
published: true
slug: gpu-article0
date: 2018-11-27T15:05:00.002Z
---






最短でGPU機械学習環境を構築する！



## 概要


日々の仕事でGPUの環境や、ローカルのCPU環境にDockerやAnacondaで環境を作ってきましたが、  

Ubuntuサーバにnvidia-dockerの組み合わせが個人的に一番楽だと思ったので、手順をメモしておきます。


### やること


UbuntuイメージがインストールされたGPUマシンにnvidia-docker2を入れて、Tensorflow Dockerイメージからコンテナを作成する。


### やらないこと


OSのインストール  

各コマンドの詳しい説明


## 構成


GCP


* ゾーン : us-west1-b
* マシンタイプ : n1-standard-8
* OS : Ubuntu 18.04 LTS
* GPU : NVIDIA Tesla K80 x 2


## GPUインスタンス立ち上げ


都合により省略  

今回はGCPを使いますが、UbuntuOSであれば基本的に問題は無いと思います。


## GPUドライバのインストール


#### [NVIDIA](https://developer.nvidia.com/cuda-downloads)からドライバをインストール


![写真]()  

必要な項目にチェックを付けて、ファイルのリンクをコピー  

GCPのターミナルでダウンロード



```
# 環境によってリンクは変化
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-repo-ubuntu1804_10.0.130-1_amd64.deb

```

ダウンロードの後は、表示コマンドを順番に入力



```
# こちらのコマンドも環境で差異があるので、ページを参照
sudo dpkg -i cuda-repo-ubuntu1804_10.0.130-1_amd64.deb
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
sudo apt update
sudo apt install cuda

```

#### Dockerのインストール


早速nvidia-docker 2をインストールしたいところですが、[installation](https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0))に従って、まずは[Docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/#set-up-the-repository)をインストール



```
sudo apt remove docker docker-engine docker.io

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
    
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88

# 環境によって違うので注意（エラーが出たので下記参照してコマンド変更）
#sudo add-apt-repository \
# "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
# $(lsb\_release -cs) \
# stable"
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic test"

sudo apt update
sudo apt install docker-ce

```

と、ここまで順調でしたが、ついにエラーが出てきました。



```
E: Package 'docker-ce' has no installation candidate

```

[こちら](https://unix.stackexchange.com/questions/363048/unable-to-locate-package-docker-ce-on-a-64bit-ubuntu)を参考にして上記のコマンドを変更したところ解消


#### nvidia-docker 2のインストール


[Installing version 2.0](https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)#installing-version-20)を参考にコマンドを実行



```
# まずリポジトリの追加
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION\_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt update
# インストール
sudo apt install nvidia-docker2
sudo pkill -SIGHUP dockerd

```

ようやく基本的な環境は整ったハズです！


## Dockerコンテナ作成


コンテナを作るために[Docker Hub](https://hub.docker.com/r/tensorflow/tensorflow/)からイメージファイルをpullします。  

今回はTensorflow公式イメージを使用しますが、他イメージも基本は同じです。



```
sudo docker pull tensorflow/tensorflow:1.11.0-devel-gpu-py3

```

ところでタグの意味は？と、思った方は[こちら](https://www.tensorflow.org/serving/docker#dockerfiles)を参照ください。  

`gpu-py3`が最小構成、`devel-gpu-py3`は依存関係も含まれるようです。


後は`--runtime=nvidia`を付けて`run`してあげるだけです。  

いつもこんな感じで使ってます。



```
# ディレクトリをマウントする際の絶対パスが面倒なので、
# いつも直下にディレクトリを作ってしまってます。
sudo mkdir /share
sudo docker run --name <コンテナ名> --runtime=nvidia -it -p 8888:8888 -p 6006:6006 -v /share:/work tensorflow/tensorflow:1.11.0-devel-gpu-py3

```

駆け足でしたが、無事に環境構築できたでしょうか？  

最後に`nvidia-smi`と打って、下記のようにGPUが認識されていれば完成です！  

忘れないうちにリポジトリの更新も行っておきましょう。



```
# apt update
# apt upgrade
# nvidia-smi

Wed Nov 21 16:20:41 2018
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 410.72       Driver Version: 410.72       CUDA Version: 10.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K80           Off  | 00000000:00:04.0 Off |                    0 |
| N/A   42C    P0    65W / 149W |      0MiB / 11441MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  Tesla K80           Off  | 00000000:00:05.0 Off |                    0 |
| N/A   62C    P0    86W / 149W |      0MiB / 11441MiB |    100%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+


```




