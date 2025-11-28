---
title: "Kaggleカーネル環境再現"
emoji: "📊"
type: "tech"
topics: ["kaggle"]
published: true
slug: kaggle-article
date: 2019-05-26T03:55:00.001Z
---



※過去の[blogger記事](https://slowlog-tkc.blogspot.com)を移植しました


Kaggleカーネル環境再現



## 概要


[KaggleカーネルのDockerイメージ](https://github.com/Kaggle/docker-python)を、ただ使っただけだとGPUを認識してくれなかったので、メモ


## 環境


**OS**: Ubuntu 18.04 bionic  

**Kernel**: x86\_64 Linux 4.15.0-47-generic  

**GPU**: GeForce RTX 2070  

**Docker**: 18.09.5, build e8ff056


## やること


* KaggleカーネルのDockerイメージの作成
* コンテナ作成
* コンテナ内からGPUが認識できているか確認


## やらないこと


NVIDIA Docker 環境の構築など  

[過去記事](https://slowlog-tkc.blogspot.com/2018/11/gpu_28.html)を参考にしてみてください


## カーネル環境構築


とても簡単です。  

やることは以下3点


* GitHubリポジトリのクローン
* build
* 環境変数変更


[README.md](https://github.com/Kaggle/docker-python)にも書いてありましたが、最後の環境変数の変更がないと、GPUを認識してくれないようです。  

(自分は見落として、色々と時間を無駄にしました…)



```
git clone https://github.com/Kaggle/docker-python.git
cd docker-python/
# 実行
./build --gpu

# コンテナ作成
docker run --runtime nvidia --rm -it kaggle/python-gpu-build /bin/bash

```

しばらく時間はかかりますが、これだけでカーネル環境が手元にできます！  

が、下記にあるように、環境変数を変更しないと、うまくGPUを認識したくれませんでした。



> 
> To ensure your container can access the GPU, follow the instructions posted [here](https://github.com/Kaggle/docker-python/issues/361#issuecomment-448093930).
> 
> 
> 


対処法は簡単で、変数を以下のように変更するだけ。



```
(container) $ export LD_LIBRARY_PATH=/usr/local/cuda/lib64

```

## 備考


もともと変数には以下の文字列が入ってると思います。  

それを変更するので、もしかしたら不具合がどこかで生じるカモしれません。



```
LD_LIBRARY_PATH=/usr/local/nvidia/lib64:/usr/local/cuda/lib64:/usr/local/cuda/lib64/stubs

```

なぜか余分なものが含まれている。と、先程のissueでは言っていて、公式もこれをREADME.mdに追記、自分もこの変更でGPUを認識してくれたので、今の所大丈夫かなと思っています。





