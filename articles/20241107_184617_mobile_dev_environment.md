---
title: "外出時の荷物を極力減らすために何としてもPCを持ち歩きたくない、けど開発体験も落としたくないプログラマーの苦悩"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nginx", "vscode", "vscodeserver"]
published: true
---

## 概要
### なんの記事？
外出先でPCをちょっとだけ開く機会がある or あるかもしれないから くらいの理由で普段からPCを持ち歩いてるプログラマが、ほぼ手ぶらで外出できるようになる方法

### この記事を読んだらできること
自宅のPCでVSCode Serverを稼働させてから外出すれば、外出先からブラウザ操作で開発ができるようになる
ブラウザでVSCodeを操作するため、外出先が外部モニタを借りられるような場所であればスマホと入力機器があればある程度快適に開発ができるので、いつでもどこでもそこそこの開発体験があなたの下へ

![alt text](/images/image_pixel9.png)


### 前提
この記事の環境を全て再現するにあたって必要なものは以下の通り
基礎知識については、書かれたままをやるだけであればなくても問題はないと思われる

- 以下の基礎知識
  - Visual Studio Code
  - DNS
  - Docker
  - リバースプロキシ
- タブレット端末 or ミラーリング/外部出力が可能なスマートフォン を所持
- Docker Desktop導入済み
- 独自ドメイン所持
- 自宅の固定IP化済 or ダイナミックDNSの設定が自力でできる
- 自宅ルーターあり


## 背景
### なぜ荷物が重いとか思ってしまうのか
~~私がひ弱で怠惰だから~~
この記事の読者層的に、起業・開発問わず「外出時にノートPCは手放せない」という方ばかりなのじゃないかと思う。好きなタイミングで書きたいだけプログラム書いて休みたいだけ休む、みたいな社会人にあるまじきスタンスで仕事をしている私でさえ、外出先でサーバーが落ちただの不具合が出たから修正するだの、PCが手元にないとちょっと不安になる自体には散々出くわしているので、基本的に開発環境は持ち歩けるようにしている。

しかしながら、正直スマホと鍵だけ持てば事足りるような外出にさえ、そこそこ嵩張る上に最低 2.0 kg程度の荷物を毎回持ち運ぶなんて面倒くさすぎしどうにかならないものか？と昔から考えてはいた

### で、どうしたか
ということで、「どうせ、PC出やるよりも大変ひどい使い心地なんでしょう？」という憂いなく、極力荷物を少なくする方法を模索してみた

比較検討した内容は本記事内に詳細は省略するが、Codespacesやgithub.dev・スマホにVSCodeを入れる等々を横並びで検討し、現在の自分の環境下でお金も手間もかけずに実現する方法として、以下の方法に付いて調査・試行した

開発機のVSCodeで起動したVSCode Serverにスマホのブラウザから接続して、ブラウザ上からVSCodeを使う以下の2つの方法について試してみたので書いていくことにする
- Remote tunnel（vscode.devからGitHubの認証を使ったセキュアなトンネルを通して開発機に接続）
- Serve-web (LANやVPNを使用してブラウザから開発機に直接HTTP接続)

## VSCode Serverについて
### VSCode Serverとは？
ローカルのVSCodeクライアントやVSCode for the Webを介して、どこからでもリモートマシンに安全に接続することができる、そんな仕組み
詳しくは[こちら](https://code.visualstudio.com/docs/remote/vscode-server)

![alt text](/images/image.png)

### 起動コマンド

cli上でcodeコマンドのヘルプを見てみると、以下のようなSubommandsがあり、これらがVSCode Serverを使うためのコマンドとなる
```
$ code -h

Visual Studio Code 1.95.3
Usage: code [options][paths...]

(中略)

Subcommands
  tunnel       Make the current machine accessible from vscode.dev or other machines through a secure tunnel
  serve-web    Run a server that displays the editor UI in browsers.
```

では、それぞれで実際にスマートフォンから開発ができるかどうか確認してみる

### code tunnelを実行してみる

早速、開発機で以下のコマンドを実行してみる

``` zsh
code tunnel
```


``` zsh
*
* Visual Studio Code Server
*
* By using the software, you agree to
* the Visual Studio Code Server License Terms (https://aka.ms/vscode-server-license) and
* the Microsoft Privacy Statement (https://privacy.microsoft.com/en-US/privacystatement).
*
[2024-12-** **:**:**] info 
Open this link in your browser https://vscode.dev/tunnel/*******

```

スマートフォンからhttps://vscode.dev/tunnel/******* にブラウザからアクセスすると、code tunnelを実行したフォルダ内のファイルをvscode上で参照できることがわかる

![alt text](/images/Screenshot_20241208-203018_Chrome.png)

当然、開発サーバーを立ち上げる事もできる
![alt text](/images/zenn_preview.png)

立ち上げた開発サーバーは自動でweb上で公開される（公開先はportタブ転送押されたアドレスを確認できる）
![alt text](/images/Screenshot_20241208-215837_Chrome.png)

アクセスすると「開発者用トンネルに接続しようとしています」という警告ページが初回だけ出るので、続行を押すと開発サーバーにもアクセスできることがわかる
![alt text](/images/Screenshot_20241208-204538_Chrome.png)


大体のことはこれだけで問題なくできるし、機能面で不便になったと感じるところもあまり感じられなかった。しかし、私の環境では**VSCode内のターミナルに一文字打つごとに少しのラグが発生したり色々な動きがほんの少し遅く、集中が乱されそうで勘弁してもらいたい**というのが正直な感想
（とはいえ、環境によっては高速に動作すると思うので、これで満足の場合は続きは興味があれば見る程度でよいと思われる）

個人的にはもう少し早く動いてくれたりしないだろうか？ということで、今度はtunnelではなく直接接続を試してみる


### code serve-web

WANで試すための設定はまだしていないので、LAN内から試すところから始める
開発機で以下のコマンドを実行する（192.168.***.***のところは、開発機のLAN内のIPアドレスで埋める）

``` zsh
code serve-web --without-connection-token --accept-server-license-terms --host=192.168.***.***
```

``` zsh
*
* Visual Studio Code Server
*
* By using the software, you agree to
* the Visual Studio Code Server License Terms (https://aka.ms/vscode-server-license) and
* the Microsoft Privacy Statement (https://privacy.microsoft.com/en-US/privacystatement).
*
Web UI available at http://192.168.***.***:8000
```

http://192.168.***.***:8000にブラウザからアクセスすると、code tunnelのときと同様ブラウザ版VSCodeが立ち上がり、開発機内を参照できることがわかる。
開発サーバーの起動等々も同様に試してみたが、機能面では拡張機能のRemote Containerが使えなくなった分、dev container等を利用する際には不便かもしれないが**速度面ではtunnelと比較するとかなり早く、ローカルで開発するのと遜色のないレベル**で快適にコーディングできた

個人的には開発環境としてこちらのほうがありがたかったので、ここからはこの環境をある程度セキュアな状態でWANに公開する方法を書いていく


## ルーターのポート開放設定
すみません、ここから雑です（近日中に追記します）

### ルーターと開発機のIPアドレスを設定する
LAN内の開発機のIPアドレスは固定しておきましょう
固定IPの場合は、ルーターのWAN側IPアドレスを確認しておきましょう。
そうでない場合はダイナミックDNSの設定が必要なので、無料のサービスを使って設定してしまいましょう

### ポートマッピング（ポートフォワーディング）の設定
80番ポートと443番ポートを開けて開発機の同ポートに繋ぐ
テストのために8000番ポートも同様にしてみる

### WANからIPアドレスで直接アクセス 
アクセスするとlocalhostと同じように開けることを確認する
※危ないのでこのままで使うのはおすすめはしない。8000番ポートは必ず閉じる

## リバースプロキシの設定
### リバースプロキシとは
外側から自宅LAN内に入ってくる際に必ず通ってくるプロキシのこと。
Nginxとか有名
80とか443番ポートから入ってきたリクエストをURLで振り分けたり、色々な設定が可能

今回は[Nginx-Proxy-Manager](https://nginxproxymanager.com/)というGUIでNginxの設定やSSLの設定が可能なものを使う

### 独自ドメインのサブドメインを取得
サブドメインのAレコードを自宅の固定IPに設定する

### Nginx Proxy Managerのセットアップ

任意のディレクトリに[公式ページのQuick Setup](https://nginxproxymanager.com/guide/#quick-setup)のcompose.ymlファイルを作成

``` yml
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

そのディレクトリで以下を実行してコンテナを起動（Docker Desktopが）
``` zsh
docker compose up -d
```

開発機で http://localhost:81 にアクセスすると、Nginx Proxy Managerが立ち上がります

### リバースプロキシの設定

Add Proxy Hostから、先に作成したサブドメインとhttp://host.docker.internal:8000 を紐づける
SSLの設定も同時にできるためLet's encryptでSSL証明書も取得する

### BASIC認証

そのままではURLを知っている誰でもが開発機の中をVSCodeで除き放題になってしまうので、BASIC認証を付ける
Nginx Proxy ManagerからACCESS制御も可能
VPN等を使ってIP制限もかけたらなお良い（今回は触れない）


## まとめ（まだまとまっていないけど）
アドカレのために投稿しましたが、近日中にちゃんと追記します。
VSCode Serverを開発機で起動し、Nginx Proxy Manager等を利用しつつ割と簡単に快適な開発環境を作れているんじゃないでしょうか、多分。

