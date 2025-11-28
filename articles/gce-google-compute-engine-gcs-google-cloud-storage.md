---
title: "GCE(Google Compute Engine)にGCS(Google Cloud Storage)をマウントする"
emoji: "☁️"
type: "tech"
topics: ["gce", "google", "compute"]
published: true
slug: gce-google-compute-engine-gcs-google-cloud-storage
date: 2018-11-27T15:10:00.002Z
---




※過去の[blogger記事](https://slowlog-tkc.blogspot.com)を移植しました

GCE(Google Compute Engine)にGCS(Google Cloud Storage)をマウントする



## 環境


GCP


* ゾーン : us-west1-b
* マシンタイプ : n1-standard-8
* OS : Ubuntu 18.04 LTS


## Cloud Storage FUSEを使う


### Cloud Storage FUSEとは？



> 
> Cloud Storage [FUSE](http://fuse.sourceforge.net/) は、オープンソースの Fuse アダプタです。これにより、Cloud Storage バケットを、Linux または OS X システム上でファイル システムとしてマウントできます。  
> 
> <https://cloud.google.com/storage/docs/gcs-fuse>
> 
> 
> 


### インストール


[公式Github](https://github.com/GoogleCloudPlatform/gcsfuse/blob/master/docs/installing.md)を参考にしながら、下記コマンドを実行します。



```
export GCSFUSE_REPO=gcsfuse-`lsb\_release -c -s`
echo "deb http://packages.cloud.google.com/apt $GCSFUSE\_REPO main" | sudo tee /etc/apt/sources.list.d/gcsfuse.list
echo "deb http://packages.cloud.google.com/apt $GCSFUSE\_REPO main" | tee /etc/apt/sources.list.d/gcsfuse.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
sudo apt update
apt update
apt install gcsfuse

```

### マウントする


こちらも公式[README](https://github.com/GoogleCloudPlatform/gcsfuse/blob/master/README.md)を参考にしながら。


1. [Google Cloud Storage JSON API is enabled](https://cloud.google.com/storage/docs/json_api/#activating)が有効になっていないとダメのようなので、わからない場合は最初に確認しておいたほうが良さそうです。


2. `mnt_bucket`というバケットを`/path/to/mnt_dir`ディレクトリにマウントする



```
$ gcsfuse mnt_bucket /path/to/mnt_dir

....
Opening GCS connection...
Opening bucket...
Mounting file system...
File system has been successfully mounted.

```

こんな感じの出力が出れば成功です。  

めっちゃ簡単！  

ちなみに、この時`sudo`は付けてはいけないらしいです。




### アンマウントする


アンマウントは以下のコマンドで



```
$ fusermount -u /path/to/mnt_dir

```




