---
title: "「Zabbixサーバーが動作していません」と、なった時の対処法"
emoji: "🚨"
type: "tech"
topics: ["zabbix"]
published: true
slug: zabbix-article
date: 2019-07-11T17:09:00.002Z
---



※過去の[blogger記事](https://slowlog-tkc.blogspot.com)を移植しました


「Zabbixサーバーが動作していません」と、なった時の対処法



### まずはログ


`/etc/zabbix/zabbix_server.conf`を見ると、logの出力先が見つかると思います。



```
# デフォルトはこの場所
LogFile=/var/log/zabbix/zabbix_server.log

```

### 今回の原因


見てみると、こんなエラーらしきものが一定間隔で出力され続けていました。（私の場合）



```
[Z3001] connection to database 'zabbix' failed: [1045] Access denied for user 'zabbix'@'localhost' (using password: NO)

```

「zabbix」ユーザーがDBにアクセスできないよ、との事  

🤔  

どうやら、zabbixをアップデートしたら、configファイルが初期化されてしまった模様。


### 対処


`/etc/zabbix/zabbix_server.conf`のDBPasswordに、MySQLのzabbixデータベースに設定したパスワードを設定。  

zabbixサーバを再起動すると、直りました。


[こちら](https://www.atmarkit.co.jp/ait/articles/0909/29/news103_4.html)のサイトにお世話になりました。





