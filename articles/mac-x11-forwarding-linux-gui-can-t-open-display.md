---
title: "title: MacからX11 forwardingでLinux GUIを表示する（Can't open display対処）"
emoji: "💻"
type: "tech"
topics: ["mac", "x11", "forwarding"]
published: true
slug: mac-x11-forwarding-linux-gui-can-t-open-display
date: 2019-07-07T04:44:00.002Z
---






Mac から X11 forwarding を使用してLinux サーバの GUI アプリを表示する（Can't open display トラブルシュート編）



## トラブルシュート編


### 概要


この設定の際になかなか悩まされたので、誰かの役に立てればと備忘録を残します。


何をしようとしていたかというと、  

Mac（クライアント）からUbuntu（ホスト）にX11 forwardingを使用してSSH接続し、  

ホスト側で実行したGUIアプリ（今回はxeyes）を、クライアント側の画面で表示させようとしていました。


### 現象


どうしても下記のエラーが出て接続できない。。。



```
$ xeyes 
Error: Can't open display: localhost:10.0

```

いろんなブログを参考にしながらいろんな設定をひたすら確認。。。  

[こちらのブログ](http://luozengbin.github.io/blog/2014-06-21-%5B%E3%83%A1%E3%83%A2%5D%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%88x%E3%81%AE%E6%8E%A5%E7%B6%9A%E6%96%B9%E6%B3%95.html)を見て順に設定を確認していた所、紹介されている出力とは明らかに違う項目を発見



```
$ xauth list
xauth:  /home/tkc/.Xauthority not writable, changes will be ignored

```

どうも **.Xauthority** というファイルがおかしい？という事に気付きました。  

よくよく見ると、ログインメッセージに以下のようなエラーも出ていました。



```
/usr/bin/xauth:  /home/tkc/.Xauthority not writable, changes will be ignored
/usr/bin/xauth:  /home/tkc/.Xauthority not writable, changes ignored

```

### 原因と対処


[このブログ](https://blog.goo.ne.jp/halcali/e/8f225dffd190d75d72c1f1959c8e6b35)や、[このフォーラム](https://superuser.com/questions/315050/xauth-x11-ssh-forwarding-errors-with-xauthority-file-not-writable)を見て対処できました。（ここまで約2週間。。。）



```
$ ssh　-X xxx@yyy
$ ls -la ~/ | grep Xauth*
drwxr-xr-x  2 root root  4096  6月 22 00:13 .Xauthority/

```

rootで作成されてて、しかもディレクトリ？？  

自分で作成した覚えや、操作をした記憶はないですが、どうもこれが原因のようです。


ログイン時に自動で生成されるようなので、削除して再ログインします。  

すると、下記のようなメッセージと共に、ファイルができていると思います。



```
# 削除 & ログアウト
$ sudo rm -r ~/.Xaut*
$ exit

```


```
# 再ログイン
$ ssh -X xxx@yyy

#ログインメッセージ
…
/usr/bin/xauth:  file /home/xxx/.Xauthority does not exist

# 確認
$ ls -la ~/ | grep Xauth*
-rw-------  1 xxx  xxx     52  7月  7 12:07 .Xauthority
$ xauth list
ubuntu/unix:10  MIT-MAGIC-COOKIE-1  51d737b446197f2381e724388b8dc2f6

```

この後にx11アプリを実行したところ、無事にクライアント側ディスプレイに表示できました。





