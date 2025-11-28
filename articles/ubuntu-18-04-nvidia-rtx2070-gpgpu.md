---
title: "Ubuntu 18.04 で NVIDIA RTX2070 のGPGPU機械学習環境を作る"
emoji: "🧠"
type: "tech"
topics: ["ubuntu", "nvidia", "rtx2070"]
published: true
slug: ubuntu-18-04-nvidia-rtx2070-gpgpu
date: 2019-06-14T14:36:00.001Z
---



※過去の[blogger記事](https://slowlog-tkc.blogspot.com)を移植しました


Ubuntu 18.04 で NVIDIA RTX2070 のGPGPU機械学習環境を作る



環境構築しかブログに書けないのは何故なのか。。。  

他の作業やってる時って、色々やるすぎてまとめられない。  

という言い訳。。。  

今年はちゃんとアウトプットしたいと思います。


## 環境


* OS : Ubuntu 18.04 Desctop
* GPU : NVIDIA RTX 2070


## やること


* NVIDIAドライバのインストール




---


以下の項目は、以前書いたことがあるので、[こちらの記事](https://blog.ai-platform.jp/2018/11/gpu_28.html)を見てください。


* Dockerのインストール
* NVIDIA Dockerのインストール
* Dockerイメージを使用してtensorflow-gpuが動くことを確認


## はじめに


GPGPU環境を作りますが、Dockerを使うことを前提としています。  

経験上、今までTensorFlow、Kerasを使用する場合、Dockerイメージを使用すれば、マシン本体に入れるNVIDIA DRIVERのバージョン、CUDAのバージョン等は最新で問題なく動きました。  

なので、今回は最新のNVIDIA DRIVERをインストールして、DockerでTensorFlow等にバージョンを合わせた環境を作っていく、という考えで作業していきます。


### 映像出力をオンボードへ


機械学習でGPUを使用するマシンという前提なので、映像出力をGPUからオンボードへ切り替えましょう。  

そうすることで、画面描画等の無駄なリソースを使わなくて済みます。


各マザーボードによって設定方法は違うので、各メーカーの説明を参考にしてください。  

ちなみに、ASUSは[こんな感じ](https://www.asus.com/jp/support/FAQ/1017796/)です。


### CUIから作業をする


あとで書きますが、ドライバインストールの際にCUIに移行します。  

そのため、最初からCUI環境で作業をオススメします。


※Ubuntu18とそれ以前の切り替え操作は若干違っているので注意！  

私の環境では「Ctrl + Alt + F3」でした。


## NVIDIAドライバのインストール


[こちら](https://gihyo.jp/admin/serial/01/ubuntu-recipe/0454)の記事を参考にしながらインストールしていきたいと思います。  

インストールする方法はいくつかあるようで、大きくは下記３つのようです。


1. Ubuntuの公式リポジトリからインストールする方法


2. [Graphics Drivers Team](https://launchpad.net/~graphics-drivers/+archive/ubuntu/ppa)のPPAからインストールする方法


3. [NVIDIA公式のサイト](https://www.nvidia.co.jp/Download/index.aspx?lang=jp)からDL、インストールする方法




できれば1. 2.の手法でインストールした方がいいと思います。  

というのも、ドライバの更新があった場合、パッケージマネージャがよしなに更新をしてくれるハズなので。


ただ、残念ながらRTXシリーズが出たばかりなので、1.のUbuntuリポジトリには対応ドライバがありませんでした。


Graphics Drivers Teamを見てみたところ、公式サイトの最新版と対応するパッケージがあったので、こちらからインストールしたいと思います。


**※と、思いましたが、こちらのPPAからも最新版が落とせない様子だったので3. の公式からインストールしたいと思います。**


### セキュアブートは無効化しておく


無効にしておかないといけないらしいですが、私の場合は設定していないので、飛ばします。


### PPAからインストール（しようとしたけど、断念）


しようと思ったけどお目当てのドライバ(410番台)は無かった。。。



```
$ sudo add-apt-repository ppa:graphics-drivers/ppa
$ sudo apt update
$ sudo apt upgrade
$ sudo apt search "^nvidia-[0-9]{3}$"
.
.
.
nvidia-390/bionic 390.87-0ubuntu0~gpu18.04.1 amd64
  Transitional package for nvidia-driver-390


```

### NVIDIA公式からDL、インストール


[公式サイト](https://www.nvidia.co.jp/Download/index.aspx?lang=jp)で、自分の環境に合うドライバを選択しましょう。


![nvidia_driver_download_page](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiAL_z6MjKsDdYft-hCRkjTAKwW8VAj4Z8nksytZBDHrC9ZgUb5K1w8oPM6OIrBkeyUs7RVLqrhTKew3ibpVf7CPDsqUe20bh4lbNoqeLoR80eTzud_KNBifg3gtmYb96-ZcisF-RiTxZmD/)  

選択したらダウンロードします。  

GUIならそのままクリック、CUIの場合は[DOWNLOAD]ボタンを押した先の[DOWNLOAD]のURLからwget等で任意の場所にダウンロードしましょう。


[公式のインストールガイド](http://us.download.nvidia.com/XFree86/Linux-x86_64/410.93/README/installdriver.html)に沿ってインストールを実行していきます。


### Nouveauの無効化



> 
> If you’re installing on a system that is set up to use the Nouveau driver, then you should first disable it before attempting to install the NVIDIA driver.
> 
> 
> 


ともあるので、Nouveauも無効にします。  

コイツの無効化の記事は氾濫していて、イマイチ何が正攻法なのかわからなかったので、少し調べてみました。  

結果、[NVIDIA CUDA TOOLKIT DOCUMENTATION](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#runfile-nouveau)内に記述を見つけたので、NVIDIAが公式に発信している手法を使いたいと思います。


Nouveauが有効になっているか、下記のコマンドで確認。



```
$ lsmod | grep nouveau

nouveau              1716224  0
mxm_wmi                16384  1 nouveau
ttm                   106496  1 nouveau
i2c_algo_bit           16384  2 i915,nouveau
drm_kms_helper        172032  2 i915,nouveau
drm                   401408  5 drm_kms_helper,i915,ttm,nouveau
wmi                    24576  4 intel_wmi_thunderbolt,wmi_bmof,mxm_wmi,nouveau
video                  45056  2 i915,nouveau


```

こんな感じのログが出てきたら有効になっているようです。


無効にするため、設定ファイルを作成して、その中に以下のコマンドで設定を記述していきます。



```
# 設定の記述
$ sudo touch /etc/modprobe.d/blacklist-nouveau.conf
$ sudo sh -c "echo 'blacklist nouveau' >> /etc/modprobe.d/blacklist-nouveau.conf"
$ sudo sh -c "echo 'options nouveau modeset=0' >> /etc/modprobe.d/blacklist-nouveau.conf"

# 設定の適用
$ sudo update-initramfs -u

```

再起動して、設定が適用できたか確認します。  

下記コマンドをもう一度打って、何も出て来なければ成功です。



```
$ lsmod | grep -i nouveau

```

### Xサーバの停止



> 
> Before you begin the installation, exit the X server and terminate all OpenGL applications
> 
> 
> 


と、書いてあるので、デスクトップ環境から抜ける、もしくはSSHなどで作業しましょう。


Xサーバ（GUI）環境を停止させます。  

ディストリビューションによって、動作しているディスプレイマネージャが違うと思います。  

適当なGUIサービスを止めましょう。



```
# サービスを確認する場合は
$ sudo service --status-all
# 対象のサービス停止
$ sudo service gdm3 stop

```

### （ようやく）NVIDIA DRIVERインストール


ダウンロード先のディレクトリに移動しで、コマンドを実行しましょう。  

ダウンロードしたインストールファイルの名前に適宜変更してください。



```
$ cd yourdirectory
$ sudo sh NVIDIA-Linux-x86_64-410.93.run

```

即死しました



> 
> The distribution-provided pre-install script failed! Are you sure you want to continue?
> 
> 
> 


と出ています。  

pre-install scriptとは一体。。。？  

経験上、このまま[continue installation]しても上手く入るのですが、ブログにするのに、エラー出っぱなしは気持ち悪いので、原因を調べました。


結果から書くと、このままエラーを無視で大丈夫です。  

無視でもダメな場合はログの内容を確認してみてください。


少し長い＆根本解決無しなので、興味無い方は読み飛ばしてください。




---


`/var/log/nvidia-installer.log`のログに出力されていた`/usr/lib/nvidia/pre-install`を見てみると



```
#!/bin/sh

# Trigger an error exit status to prevent the installer from overwriting
# Ubuntu's nvidia packages.

exit 1

```

、、、絶対にエラー吐くマンでした。  

これはPPAを追加した時にでもできたもの？


[NVIDIA Forum](https://devtalk.nvidia.com/default/topic/970641/linux/the-distribution-provided-pre-install-script-failed-are-you-sure-you-want-to-continue-/)で、「ドライバーちゃんと消してる？」となっていますが、そもそも今回が最初のインストール  

一応PPAを削除して、ついでにパッケージを整理します。



```
$ sudo add-apt-repository --remove ppa:graphics-drivers/ppa
$ sudo apt autoremove
$ sudo apt autoclean

```

まだファイルは無くならない。


今度は[Ubuntuのフォーラム](https://askubuntu.com/questions/206283/how-can-i-uninstall-a-nvidia-driver-completely)から、NVIDIA DRIVERの完全削除を調べて実行。



```
$ dpkg -l | grep -i nvidia

```

何も出てこない。。。  

なにやらubuntu-desktopを入れるとnvidia-commonが入るとか言ってるけど、そんなものは無かった。


念の為削除コマンドしてみるが、もちろん削除されるものは何も無い。



```
$ sudo apt-get remove --purge '^nvidia-.*'

```

そもそも、`/usr/lib/nvidia/`の下に`pre-install`しか無いので、必要無い気しかしません。


すると、`pre-install`はエラーを吐かせ続けるので[削除するという記事](https://blog.techlab-xe.net/archives/4382)を見つけました。


`pre-install`のコメントはパッケージの上書き防止となっていますが、パッケージは何も無いことは確認したので、挙動を確認したい気持ちもあり、バックアップを取って削除することにしました。


最初のエラーが出なくなったので、結果的に、このファイルを無視する方向でよかったみたいです。




---


最初のエラーを無視 or ファイル削除で切り抜けたものの、即エラー



> 
> ERROR: Unable to find the development tool `cc` in your path; please make sure that you have the package ‘gcc’ installed. If gcc is installed on your system, then please  
> 
> check that `cc` is in your PATH.
> 
> 
> 


次はgccが無いと言っているので、入れて再実行。



```
$ sudo apt install gcc

```

ハイ、、、



> 
> ERROR: Unable to find the development tool `make` in your path; please make sure that you have the package ‘make’ installed. If make is installed on your system, then please  
> 
> check that `make` is in your PATH.
> 
> 
> 


次はmakeですね。



```
$ sudo apt install make

```

そして再実行すると



> 
> WARNING: Unable to find a suitable destination to install 32-bit compatibility libraries. Your system may not be set up for 32-bit compatibility. 32-bit compatibility files  
> 
> will not be installed; if you wish to install them, re-run the installation and set a valid directory with the --compat32-libdir option.
> 
> 
> 


「32bit対応できないので、よろしく」  

ということなので、もちろんそのまま「OK」



```
An incomplete installation of libglvnd was found. All of the essential libglvnd libraries are present, but one or more optional components are missing. Do you want to install
  a full copy of libglvnd? This will overwrite any existing libglvnd libraries.

```

「libglvndが不完全なインストール状態だけど、インストールする？しない？」  

ということなので、`Install and overwrite existing files`を選択



> 
> Would you like to run the nvidia-xconfig utility to automatically update your X configuration file so that the NVIDIA X driver will be used when you restart X? Any  
> 
> pre-existing X configuration file will be backed up.
> 
> 
> 


「X （GUI）を再起動したらNVIDIA X driverが使用される様にしますか？」


これは正直すごく悩ましい。。。  

GPUに画面描画の仕事をさせるつもり無いんですが、GUIで認識されないと何か不都合があったりする？  

そもそも、オンボードグラフィックになってるし、NOでいい？  

でも、いざ画面で何かを使う時が来る？  

悩んだ結果「YES」にしました。


過去の経験上、YESにしてインストールできて、不都合なかったので。


### インストールできたか確認


下記のコマンドを打って表示が出れば完了です！  

お疲れ様でした。



```
$ nvidia-smi
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 410.93       Driver Version: 410.93       CUDA Version: 10.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce RTX 2070    Off  | 00000000:01:00.0 Off |                  N/A |
|  0%   38C    P0    40W / 175W |      0MiB /  7952MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+


```




