---
title: "JETSON NANO セットアップ：ハード編"
emoji: "🤖"
type: "tech"
topics: ["jetson", "nano"]
published: true
slug: jetson-nano-article
date: 2020-06-07T14:39:00.001Z
---

※過去の[blogger記事](https://slowlog-tkc.blogspot.com)を移植しました

JETSON NANO セットアップ：ハード編




## 
概要


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgypt37e_jWD9s5vUWEaKbQVb2Y8iigZMrtpteQ2-4Xvm0XDkMEK2p2DR1HJPtzirg1ClsV6jnWJ46VHN0eOQl_cDHp2cIaIDQ-EiQVGqX-Pv_YU-w87PDwYIJXiobndFhcmMj0A16nLSck/s640/IMG20200527013206.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgypt37e_jWD9s5vUWEaKbQVb2Y8iigZMrtpteQ2-4Xvm0XDkMEK2p2DR1HJPtzirg1ClsV6jnWJ46VHN0eOQl_cDHp2cIaIDQ-EiQVGqX-Pv_YU-w87PDwYIJXiobndFhcmMj0A16nLSck/s1600/IMG20200527013206.jpg)  
  

眠っていたJETSON NANO をセットアップしたのですが、手順は簡単でも、  

最小の構成ではあまりイケていなかったので（形から入るタイプ）  

手軽かつ、実用的な範囲の構成でセットアップした手順をまとめました。  


基本的には[NVIDIA公式ページ](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit)に書かれている手順プラスα な内容です。  

### 
やること


* セットアップの構成説明
* ハードウェアの取り付け


### 
やらないこと


* Jetsonのセットアップ  

（セットアップ手順は[公式ページ](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit)を参考にしてください。）
* 詳細な手順説明
* 用語、部品の説明


## 
1. ) 最小構成


JETSON NANO を購入しただけでは起動すらできず、電源やmicroSD を買い足して上げる必要があります。  

必要最低限のモノと、私が購入時に用意したモノを以下にまとめました。  


> 
> 1. NVIDIA公式の最小構成
> 2. 注意点
> 3. 私の構成
> 
> 
> 


1. **JETSON NANO**  

NVIDIA の各代理店から購入可能です。  

[マクニカ](https://amzn.to/2ZyC9Og)さんはAmazon店も  

2. **電源**  

後述しますが、私はMicro-USB電源を使用していません。  

ラズパイの電源が使用できるので、既にお持ちの方や、GPUはそこまで使用しない場合はこの電源で問題無いと思います。  

	1. Micro-USB (5V⎓2A)
	2. ラズパイの電源など、上記スペックの電源も使用可能だが、ACアダプタであれば4Aまで使用することができる。
	3. ACアダプタ（後述）
3. **microSD カード**  

Docker や 機械学習モデルを ゴニョゴニョしていると、64GB では不安かなと思ったのと、  

比較的安かったので128GBにしました。  

	1. 16GB UHS-1
	2. SDカードの規格はとても多いので、推奨されているUHS-1以下を意図せず選ばないよう注意  
	
	また、OSだけで13GBほど使用しているようなので、16GBだとかなりギリギリ
	3. [Transcend microSDカード 128GB UHS-I U3 Class10](https://amzn.to/36oQPB9)
4. **キーボード + マウス**  

特に注意書きはありませんが、有線接続できるものが無難と思います。（私の無線マウスは動きました）  

	1. 特になし
	2. USB有線接続が無難
	3. 家にあった[キーボード](https://amzn.to/3e73bR0)と[マウス](https://amzn.to/36rXL0c)
5. **ディスプレイ + 接続のためのケーブル**  

	1. HDMI 又は DisplayPort 接続できるもの
	2. 特になし
	3. 自宅にころがっていたHDMIケーブルと、ディスプレイ


## 
2. )お手軽追加構成（やったこと）


1. **ケース**  

基盤むき出し＆机に直置きでは何かと不安なので、[アクリルケース](https://amzn.to/3f2pLuD)に入れました。  

JetsonNanoが入っていた箱に簡易台座もありましたが、場所も取る（カタチから入る）ので  

2. **冷却ファン**  

他Jetsonシリーズは、学習や推論をさせるとかなりの高音になるので、Nanoにも冷却用ファンを用意しました。  

ケースを買った時に付属していたファンは風が弱かったのと、PWM制御ができるので[Noctuaのファン](https://amzn.to/2AIGW5A)を新しく購入。  

[![](//ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=B071FNHVXN&Format=_SL250_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=slowlog-22&language=ja_JP)](https://www.amazon.co.jp/gp/product/B071FNHVXN/ref=as_li_ss_il?ie=UTF8&psc=1&linkCode=li3&tag=slowlog-22&linkId=d71f49c2ae8ba857f87b782a7c030437&language=ja_JP)![](https://ir-jp.amazon-adsystem.com/e/ir?t=slowlog-22&language=ja_JP&l=li3&o=9&a=B071FNHVXN)
3. **ACアダプタ**  

こちらも機械学習用途で、マシンパワーを最大限しようしたかったため、USBではなく[ACアダプタ](https://amzn.to/3h5hwiY)を用意。  


ACアダプタから電源を取るには、ジャンパピンで↓のJ48をショートさせてあげましょう。  

![](https://a70ad2d16996820e6285-3c315462976343d903d5b3a03b69072d.ssl.cf2.rackcdn.com/eea60bd8b76d5bee61801abff56226c9)  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj6pay85VaDhgE12J69FM5TXX1iCJL87JvKYmeJcDZwWWpthNA_A_bpMMYh5RCxkm8LMjNHfAlJfslCO0mDi3EPWhpfrlJtuStC2zfPGtAqb1cZxlV-TLSk25JmrC2aSCyVhqShdcuDl2sB/s320/jetsonNano_jumperPin.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj6pay85VaDhgE12J69FM5TXX1iCJL87JvKYmeJcDZwWWpthNA_A_bpMMYh5RCxkm8LMjNHfAlJfslCO0mDi3EPWhpfrlJtuStC2zfPGtAqb1cZxlV-TLSk25JmrC2aSCyVhqShdcuDl2sB/s1600/jetsonNano_jumperPin.jpg)
4. **無線LANアダプタ**  

有線だと取り回しが悪いので、無線化しました。  

本当はm.2のwifiモジュールを入れたかったのですが、安いのと、現状必要性があまりないので[TP-Link TL-WN725N](https://amzn.to/2MEfjwZ)を使用しています。  



## 
3. ) +できたらやりたい追加（やれていないこと）


1. **起動スイッチ**  

JetsonNano は Button header ピンがあるので、電源スイッチを設置可能です。  

ケーブルの抜き差しで電源ON OFFをするのはかっこ悪いので、電源ボタンを追加したい。  

2. **wifiモジュール**  

現状はUSB端子を１つ消費しているので、m.2の無線LANモジュールに移行したい。  

なぜなら、アンテナがかっこいいから（）  






