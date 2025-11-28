---
title: "Jupyter notebook で ROS を動かす"
date: 2019-05-25T17:11:00.002Z
slug: jupyter-notebook-ros
---






Jupyter notebook で ROS を動かす



ついにROSがjupyter notebook上で動くようになったらしい！  

<https://blog.jupyter.org/ros-jupyter-b7e82b5e1202>


ロボットからしばらく離れていたけど、何かやりたいなと燻っていたので、これをキッカケにもう一度ロボットに触れたいという思いで飛びつきたいと思います。（素人感）  

機械学習とロボットを結びつけたいとも思ってたので、これで少し結びつけやすそう？


# 使用環境


**OS**: Ubuntu 18.04 bionic  

**Kernel**: x86\_64 Linux 4.15.0-46-generic  

**CPU**: Intel Core i7-9700K @ 8x 4.9GHz  

**GPU**: GeForce RTX 2070  

**RAM**: 944MiB / 32034MiB


**Docker**: 18.09.3, build 774a1f4


ROSでGPUを使用できるかわかりませんが、一応GPUを積んでいるので、記載しています。


# 概要


### やること


基本的な環境はDockerで構築します。


1. Dockerfileから環境の構築
2. コンテナの作成
3. jupyter-rosの実行


### やらないこと


* Dockerのインストール、使用方法
* python等の基本的操作


今回は上記のような、各ツールの基本的な部分には触れません。  

Docker、python、jupyterの基本的な使用ができて、環境構築経験のある方向けの内容になります。  

Dockerや環境構築系は少し書いたこともありますが（[コチラ](https://slowlog-tkc.blogspot.com/2018/11/gpu_28.html)）、他の内容が濃いので、見てもわかりにくいかもしれません。


# 環境構築


[公式リポジトリ](https://github.com/RoboStack/jupyter-ros)を参考にしながら進めていきます。


### 1. Dockerfileから環境の構築


昔ははコマンドを打ってセットアップしていましたが、ROSもDockerで簡単にセットアップできるようです！  

最新のmelodicから`melodic-robot`イメージを使用します。


その他TAGは[docker hub](https://hub.docker.com/_/ros?tab=description)を参照して下さい。


[公式リポジトリ](https://github.com/RoboStack/jupyter-ros)に必要ライブラリが書いてあるので、まとめてDockerfileにしてしまいます。


ポイントしてはnodejs8です。10を入れると途中でスタックしました。


***注意：jupyterlab関係のinstallに、かなり手こずりました。  

nodejs等のインストールもあるので、Ubuntu18.04 以外の環境の動作が割とわかりません。  

jupyter notebookだけでいい人は、`#nstall nodejs npm`以下をコメントアウトして飛ばしてしまってもいいと思います。***



```
FROM ros:melodic-robot

WORKDIR /opt
RUN apt-get update -y
RUN apt-get upgrade -y
RUN apt-get install -y wget vim curl git

# install pip
RUN wget https://bootstrap.pypa.io/get-pip.py
RUN python get-pip.py

RUN pip install jupyter bqplot pyyaml jupyros jupyterlab
RUN jupyter nbextension enable --py --sys-prefix jupyros

# install nodejs npm
RUN apt-get remove -y --purge nodejs npm
RUN apt-get clean -y
RUN apt-get autoclean -y
RUN apt-get install -f -y
RUN apt-get autoremove -y
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash -
RUN apt-get install -y nodejs
RUN curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN apt-get update && apt-get install -y yarn

# install the extension for jupyterlab
RUN jupyter labextension install jupyter-ros

```

### 2. コンテナの作成


* jupyter用のポートフォワードを`8888`に設定
* ホストディレクトリのマウント


上記の辺りは好みで変更して下さい。


nvidia-docker2が入っているので、一応`--runtime=nvidia`していますが、使用しない場合は不要です。



```
# Dockerfileのあるディレクトリで
docker build ./ -t jupyter-ros
docker run --name jupyter-ros --runtime=nvidia  -it -p 8888:8888 -v /work:/work jupyter-ros bash

```

### 3. jupyter-rosの実行


まずは`jupyros`が無事にimportできるかテスト



```
# python
import sys
sys.path.append('/opt/ros/kinetic/lib/python2.7/dist-packages/')

import jupyros

```

無事実行できたら、次はデモコードを動かします。  

リポジトリを任意の場所にcloneして、jupyter notebookを実行します。  

実行時のオプションは、環境に合わせて下さい。



```
git clone https://github.com/RoboStack/jupyter-ros.git
cd jupyter-ros
jupyter notebook --ip=0.0.0.0 --allow-root --no-browser

```




