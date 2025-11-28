---
title: "Ubuntu18.04のCPU、GPU 温度を、Zabbix4.0から監視する"
emoji: "📈"
type: "tech"
topics: ["ubuntu18", "cpu", "gpu"]
published: true
slug: ubuntu18-04-cpu-gpu-zabbix4-0
date: 2019-02-05T16:05:00.002Z
---






Ubuntu18.04のCPU、GPU 温度を、Zabbix4.0から監視する



外から自宅のGPU環境を使用する場合、いつの間にかマシンの温度が上がりすぎ、死亡、最悪火事、なんて事にはしたくないので、温度を監視したいと思います。


やはり、皆考えることは同じようで、[先駆者の方々](http://nonbiri-tereka.hatenablog.com/entry/2017/10/24/081504)を参考にさせてもらいながら、最新の手法に則って導入したいと思います。


## 環境


* OS: Ubuntu 18.04
* CPU: インテル
* GPU: NVIDIA


## やること


Ubuntuサーバの温度監視をZabbixを使って行う。  

できればアラートも上げたい。


1. Zabbixインストール
2. Zabbix監視設定


## Zabbix4.0のインストール


[Zabbix公式](https://www.zabbix.com/documentation/4.0/manual/installation/install_from_packages/debian_ubuntu)のドキュメントを参考にしていきます。


### Zabbixリポジトリの追加



```
# この先rootユーザーでの実行なので、rootに切り替えます
$ sudo su

# wget https://repo.zabbix.com/zabbix/4.0/ubuntu/pool/main/z/zabbix-release/zabbix-release\_4.0-2+bionic\_all.deb
# dpkg -i zabbix-release\_4.0-2+bionic\_all.deb
# apt update

```

### サーバー/プロキシ/フロントエンドのインストール



```
# apt install zabbix-server-mysql
# apt install zabbix-proxy-mysql
# apt install zabbix-frontend-php

```

### データベースを作成



> 
> ZabbixサーバーとZabbix proxyには別々のデータベースが必要です。同じデータベースを使用することはできません。したがって、それらが同じホストにインストールされている場合、それらのデータベースは異なる名前で作成する必要があります。
> 
> 
> 


と、注意書きがありますが、Zabbix proxyは使用しないので、特に気にしません。



```
shell> mysql -uroot -p<password>
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by '<password>';
mysql> quit;

```

### データインポート



```
# zcat /usr/share/doc/zabbix-server-mysql/create.sql.gz | mysql -uzabbix -p zabbix

```

### Zabbixサーバー用DBの設定


ところどころコメントアウトしてあるので、下記のように編集します。



```
# vi /etc/zabbix/zabbix\_server.conf
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=<password>

```

### Zabbixサーバーの起動



```
# service zabbix-server start
# update-rc.d zabbix-server enable

# ついでにapacheもリスタート
# service apache2 restart

```

### apacheの設定


`/etc/apache2/conf-enabled/zabbix.conf`で、コメントアウトされているタイムゾーンを編集する。



```
# vi /etc/apache2/conf-enabled/zabbix.conf
# php\_value date.timezone Europe/Riga
php_value date.timezone "Asia/Tokyo"

```

下記にアクセスできていることが確認できれば、無事設定できています。  

http://<サーバのIP>/zabbix


![zabbix_setup.php](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiiLRPHrXih7gWGfU5QFCC1FXGSj9b-pbF6MTjlwUYX2R6AWZzjDoWpg0ACooe6wujE0WmJ4hFJKKlyqU78f1KRPNqRycDpqugGXtiX98aY3MViv3qipK1KGEx0KnsSa3gRL9ItxYCr4S0N/ "zabbix_setup.php")


以降の設定ですが、特に難しさは無い（画面に従うだけな）ので、詳しく書かれている[公式ページ](https://www.zabbix.com/documentation/4.0/manual/installation/install#installing_frontend)を参考にしてください。


無事に設定が完了しして、`User:Admin Pass:zabbix`（デフォルト）でログインすると、こんな画面が出てきます。  

![zabbix_home](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgQ0lCvDFHJ2fjRWpid73cZ2CVvGiKZAAHijb-BjjVGJ3yr6j_iBR-EROdsg21ZB_T3VTvaq2qSA8GliiyuRP_xv2AuIOF5qMyi2SQStGh7VY9h_Y2clEL2mLb_BuUJ47CiCFSLv4IH5fyp/ "zabbix_home")


さっそく「エージェントと通信できません」問題が発生しているので、解消したいと思います。


## Zabbix Agentインストール


サーバ内の情報を監視するのがサーバー、その情報を集めるのがAgentですね。  

今回は、Zabbixサーバ自体を監視するので、どちらも同じマシンにインストールする必要があります。



```
# apt install zabbix-agent
# service zabbix-agent start
# ついでに自動起動の設定
# systemctl enable zabbix-agent.service

```

しばらく待つと警告は消えてると思います。  

インストール等は終わりです。  

お疲れ様でした。


## ZabbixでCPU、GPUの温度監視設定


先駆者の方がそのすべてを記してくれています。~~（ちからつきた）~~


**GPU**  

<http://nonbiri-tereka.hatenablog.com/entry/2017/10/24/081504>


**CPU**  

<https://qiita.com/DaisukeMiyamoto/items/468edc5e5c927480a3d7>


ただし、CPUに関しては4コアまでしか表示されないので、8コアに対応させ、  

更に、閾値以上の温度でアラートが上がるよう、トリガーを追加しました。  

よりコアの多い方等、[こちら](https://github.com/TakeshiKishita/utils/blob/master/zabbix/cpu_temperature.xml)を参考にしてみてください。





