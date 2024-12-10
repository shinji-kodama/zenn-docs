---
title: "PCを持ち歩かず、外出時にスマホで快適にコーディングする"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode", "vscodeserver", "android", "nginx", "proxy"]
published: true
---

## 概要
普段使用している開発機でVSCode Serverを起動しておけば、ブラウザから開発機内のデータとVSCodeを使ったコーディングが可能になるため、Androidのデスクトップモードを使う等でかなり快適な開発環境を維持する事が可能

下の画像はAndroidをデスクトップモードで外部モニタに接続し、ブラウザ版VSCodeから自宅のMac miniに繋いで開発しているところ
![alt text](/images/image_smart_connect.png)

外出先でPCを開く機会があるかも…程度の理由で普段からPCを持ち歩いているプログラマが、上記を使ってほぼ手ぶらで外出できるようになった

本記事では、その方法を紹介する


## 背景
### 目的
基本的に開発環境は常に持ち歩くようにしている開発者は多いと思うが、正直スマホと鍵だけ持てば事足りるような外出にさえ、そこそこ嵩張る上に付属品等含めると最低 2.0 kg程度の荷物を毎回持ち運ぶなんて面倒過ぎるしどうにかならないものか？と昔から考えていた。
最近、外部出力できるスマートフォンを入手したため、これを機に極力荷物を少なくする方法を模索してみた。

### 紹介対象
調査する中で、ブラウザからの接続で使えるVSCode Serverの存在(※)と、Androidに「デスクトップモード」なる外部モニタ接続によりPCのように操作できるモードの存在を知り、これらを組み合わせて快適な外での開発ができるのではないかと考え、その利用方法についてまとめた。

具体的には、メイン開発機のVSCodeで起動したVSCode Serverにスマホのデスクトップモードでブラウザから接続し開発機内のデータとVSCodeを使って開発を行う。
開発機のVSCodeの使い方には以下の2つの方法があったため、それぞれについて紹介する
- Remote tunnel（vscode.devからGitHubの認証を使ったセキュアなトンネルを通して開発機に接続）
- Serve-web (LANやVPNを使用してブラウザから開発機に直接HTTP接続)

※ デスクトップ板VSCode or SSH接続が必須だと勘違いしていた

### 前提条件
この記事の環境を全て再現するにあたって必要なものは以下の通り

- 以下の基礎知識 （書かれたままをやるだけであれば知らなくても問題なし）
  - Visual Studio Code
  - Docker
  - リバースプロキシ
  - 簡単なネットワークの知識
- タブレット端末 or ミラーリング/外部出力が可能なスマートフォン
- Docker Desktop 導入済み
- 独自ドメイン所持
- 拠点固定IP化済 or ダイナミックDNSの設定が自力でできる
- 拠点にルーターあり


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

### code tunnel を実行してみる

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

スマートフォンからhttps://vscode.dev/tunnel/******* にブラウザからアクセスすると、`code tunnel`を実行したフォルダ内のファイルをvscode上で参照できることがわかる

![alt text](/images/Screenshot_20241208-203018_Chrome.png =300x)

当然、開発サーバーを立ち上げる事もできる
![alt text](/images/zenn_preview.png)

立ち上げた開発サーバーは自動でweb上で公開されるため、portタブにある転送されたアドレスにアクセスすれば良い
![alt text](/images/Screenshot_20241208-215837_Chrome.png)

アクセスすると「開発者用トンネルに接続しようとしています」という警告ページが初回だけ出るので、続行を押すと開発サーバーにもアクセスできることがわかる
![alt text](/images/Screenshot_20241208-204538_Chrome.png =300x)


大体のことはこれだけで問題なくできるし、機能面で不便になったと感じるところもあまり感じられなかった。

しかし、私の環境では**VSCode内のターミナルに一文字打つごとに少しのラグが発生したり色々な動きがほんの少し遅く**、個人的にはもう少し早く動いてくれたりしないだろうか？と思っており、今度は`code tunnel`ではなく直接接続を試してみる


### code serve-web を実行してみる

WANへの公開設定は未実施なので、LAN内で試すところから始める
開発機で以下のコマンドを実行する（`192.168.***.***`のところは、開発機のプライベートIPアドレスで埋める）

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

`http://192.168.***.***:8000` にブラウザからアクセスすると、`code tunnel`のときと同様ブラウザ版VSCodeが立ち上がり、開発機内のデータを参照できることがわかる。

開発サーバーの起動等も同様に試してみた（画像は省略）が、機能面では拡張機能のRemote Containerが使えなくなった分、Dev Container等を利用する際には不便だが**速度面ではtunnelと比較すると早く、ローカルでの開発と遜色ないレベル**で快適にコーディングできた


### 両者の比較
試す以外に調べた内容をもとに、両者の比較をしてみた

| 特徴                     | code tunnel                                      | code serve-web                                |
|--------------------------|--------------------------------------------------|-----------------------------------------------|
| **接続方法**             | インターネットを介してリモートアクセスが可能     | LAN内またはWAN設定により接続が可能            |
| **セットアップの容易さ** | ほぼ自動設定で簡単                               | IPアドレス指定やLAN設定が必要でやや複雑       |
| **速度**                 | 環境によってはラグが発生する                     | 高速でローカルに近いレスポンス                |
| **セキュリティ**         | マイクロソフトのセキュアトンネルを利用           | 手動で設定が必要                              |
| **拡張機能のサポート**   | Remote Containerなどの拡張機能をサポート         | Remote Container等の閣僚機能が利用不可        |
| **開発サーバー**         | 自動でポートフォワーディングを行いインターネット上で公開可能 | 手動でWAN公開設定を行う必要がある |


- **code tunnel**は、外出先やネットワークをまたいで作業する必要がある場合に適しており、自動的にセキュアなトンネルを設定してくれるため設定の手間が少なく便利。一方で、速度が遅く感じられることがある
- **code serve-web**は、速度重視のローカル環境やLAN内での開発に最適。ただし、セキュリティ設定や環境構築に多少の手間がかかる

速度以外は圧倒的に`code tunnel`のほうが優秀そうなのでそちらを使うことを強くおすすめしますが、個人的に**速度の問題は大きい**ので、ここからは`code serve-web`をある程度セキュアな状態でWANに公開する設定方法を書いていく

ここまで読んで「もう`code tunnel` でいいや」となった方には、あとはAndroidのデスクトップモードの使い方だけ参照するとよいかも


## LAN外からcode serve-webで起動したVSCode Serverにアクセス
まず、`code serve-web`で起動したVSCode ServerをLANの外からアクセスするための設定をしてみる

### 前提
前述の通り、筆者の拠点は固定IPであるため、その前提で話を進めていく
もし、固定IPを取得しない場合はダイナミックDNS等で対応が可能

### 事前準備
頻繁に使うわけでもないのであれば、DHCPにして毎回確認しても別によいのだが、簡単のためにLAN内での開発機のプライベートIPアドレスを固定にする

参考：
- Windowsの場合: [Windows11のIPアドレスを固定に設定する手順](https://www.aterm.jp/support/qa/qa_external/00244/win11.html)
- Macの場合: [MacでDHCPまたは手動IPアドレスを使用する](https://support.apple.com/ja-jp/guide/mac-help/mchlp2718/mac)

### ポートマッピング（ポートフォワーディング）の設定
ブラウザからルーターのアドレス（例: http://192.168.0.1 ）にアクセスし、[ポートマッピング](https://www.aterm.jp/function/wg1800hp/guide/list/main/m01_m15.html)の設定をする。

80番ポートと443番ポートへのアクセスを、固定した開発機のプライベートIPアドレスの同ポートにマップするよう設定を追加。設定方法はルーターのメーカーによって異なるが、今回はAterm製品の機能詳細ガイドを参考にしてほしい。

https://www.aterm.jp/function/wg1800hp/guide/web/main/8w_m6.html

上記ページに則って説明をすると、具体的には以下のようなエントリを3件追加して保存する。

```
- LAN側ホスト：固定した開発機のIPアドレス
- プロトコル：TCP
- ポート番号：80 （or 443 or 8000）
- 優先度：（重複していない数字なら何でもok）
```

### LAN外からIPアドレスで直接アクセス 

開発機が置いてある拠点のグローバルIPアドレスにアクセスして確認してみる。

グローバルIPアドレスが不明な場合は、開発機でこちらのページにアクセスすることで確認可能。
https://www.cman.jp/network/support/go_access.cgi

確認後、開発機で以下のコマンドを実行 (hostには開発機のプライベートIPアドレスを使用)

``` zsh
code serve-web --without-connection-token --accept-server-license-terms --host=192.168.***.***
```

VSCode Serverが起動してから、**スマートフォンのWiFiをoffにして** WANから`http://グローバルIPアドレス:8000` にアクセスするとブラウザ版VSCodeが開く。

現状では**グローバルIPを知っている人間誰にでもアクセスが可能**であるため、ここからは少しでもセキュアにするための設定を行っていく。

:::message 
ここでの接続テストが完了したら、ルーターの8000番ポートは必ず閉じておく
:::

## code serve-webをセキュアに使うための設定
以下を設定していく
- リバースプロキシを設置
- SSL化
- BASIC認証
- IPアドレス制限

### 事前準備（サブドメインの設定）
所持している独自ドメインからVSCode Server用と開発サーバー用のサブドメインを設定する
基本的には、独自ドメインを管理しているDNSでサブドメインのAレコードを開発機の置いてある拠点のグローバルIPアドレスに設定するだけでok

- VSCode Server用： `code.example.com`
- 開発サーバー用：  `serve.example.com`

### リバースプロキシの設定
サブドメインへのアクセスがあった際に振り分ける設定をする
具体的には、設定した2つのサブドメインをVSCode Serverを公開するポートと開発サーバーのポートを紐づける

:::details リバースプロキシとは
LANの外側から自宅LAN内にアクセスする際に通るプロキシ
皆様御存知のNginxもwebサーバ兼リバースプロキシとして使われることが多い

詳しくはこちらから
https://wa3.i-3-i.info/word1755.html
:::

今回は**Nginx-Proxy-Manager**というGUIで色々な設定が可能なツールを使う
https://nginxproxymanager.com/

#### Nginx Proxy Managerのセットアップ

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

Docker Desktopが起動していることを確認し、そのディレクトリで以下のコマンドを実行してコンテナを起動

``` zsh
docker compose up -d
```

開発機で http://localhost:81 にアクセスすると、以下のページが表示される

![](https://nginxproxymanager.com/screenshots/login.png)


Emailとパスワードを求められますが、初回は以下のデフォルトのAdmin Userでログインできる。

``` txt
Email:    admin@example.com
Password: changeme
```

#### URLでVSCode Serverのポートにアクセス可能に設定

ログインするとパスワードの再設定を求められるため対応後、[Add Proxy Host]から、先に作成したサブドメインと`http://host.docker.internal:****`(※) を紐づける。サーバー起動時にポートを指定すればよいため任意のポート番号で良いが、今回は localhost:30001,30002 でVSCode Serverと開発サーバーをそれぞれ起動する想定で、以下のように設定する

- code.(独自ドメイン)  → host.docker.internal:30001 
- serve.(独自ドメイン) → host.docker.internal:30002 

![alt text](/images/image_add_proxy_host.png)

Websocket Supportはonにしておく

![alt text](/images/image_new_proxy_host.png)

これで、サブドメインを使ったhttp通信が可能になる


※ `host.docker.internal`: Docker Desktop利用時に使用可能な、コンテナ内からlocalhostにアクセスできるちょっと特殊なドメイン

#### SSL証明書の取得
Nginx Proxy Managerでは、DNS設定が完了していれば Proxy host設定時に[SSL]タグから Let's EncryptのSSL証明書も同時に取得できるのでやっておく
![alt text](/images/image_ssl.png)

これも、code.(独自ドメイン)と serve.(独自ドメイン)の両方ともやっておく

#### アクセス制限用の設定を作成
現状では、URLを知っている人間全員が開発機内をVSCodeでアクセスできてしまうので、アクセス制限をつける。NginxのBasic認証やhttp_accessによるip制限と同様の設定が可能なので、それぞれ設定を加えていく。

まず、以下の画像のように[Access Lists] -> [Add Access List] をクリックする
![alt text](/images/image_access_lists.png)

Detailsタグが開いているので、任意の名前をつける。

今回は、「Basic Auth VSCode」という名前で作成

##### Basic認証
Basic認証を付ける場合は[Authorization]タグから、UsernameとPasswordを設定。

![alt text](/images/image_basic_auth.png)

##### IPアドレス制限
VPN等を使ってIP制限もかけられる場合は、そちらも[Access]から制限をかける

![alt text](/images/image_allow_ip.png)

Nginxでの設定方法と同様のため、以下の記事のallow/deny ディレクティブについてを参考にすると良い

https://cn.teldevice.co.jp/blog/p36525/

#### VSCode Serverへのアクセス制限を適用する
上で作成したアクセス制御設定を、code.(独自ドメイン)のアクセスに適用する。

[Hosts] > Proxy Hosts にアクセスし、対象のレコードの最右にある三点リーダから編集が可能


以下の画像の通り、Access Listに先ほど設定したアクセス制限用の設定が選択可能となっているので選択して確定させる

![alt text](/images/image_apply_basic_auth.png)


### VSCode ServerにURLでアクセスしてみる
以下のコマンドで、VSCode serverを起動する
`host`は`localhost`(=`127.0.0.1`)、`port`は先にNginx Proxy Managerで設定した`30001`で起動しています

```
code serve-web --without-connection-token --accept-server-license-terms --host=127.0.0.1 --port=30001
```

スマートフォンのWiFiを無効にし、WANから `https://code.(独自ドメイン)`にアクセスすると以下のようにBasic認証を求められるようになる
![alt text](/images/image_basic_auth_phone.png =400x)

設定したUsernameとPasswordを入力すると、以下のようにVSCodeが開く
![alt text](/images/image_vscode_phone.png =300x)

適当なhtmlのファイルがあるディレクトリを開き、Live Serverの公開port設定を30002番ポートに変更（`settings.json`に`liveServer.settings.port=33002`と記述）してからopen with live serverを選択すると
![alt text](/images/image_open_with_live_server.png =300x)

以下のように、`https://serve.(独自ドメイン)` にアクセスすることで起動したLive Serverの画面をスマホで閲覧することが可能
![alt text](/images/image_rubik.png =400x)


### 注意点
今回は簡単な開発サーバーを起動しただけなので追加の設定は不要であったが、redirectやcallback用のURL（開発環境では大体`localhost:<port>`）が環境変数に入っている場合、用途によってサブドメインに書き換えが必要な場合がある。

現状動いている開発環境が動かない場合はそこでハマっている可能性が高いため、VSCode Serverでの開発用の`.env`ファイルを作成し、環境によって読み込むファイルを変えると便利



## スマートフォンの設定
VSCode Serverの設定は完了したので、ここからはスマートフォン側の設定を行う

### 前提

外部モニタに接続して開発をしようと思っても、端末自体が以下のどちらかに対応している必要がある

- DisplayPort Altモードでの有線接続
- Miracastによる無線接続（動作は重く、モニタ側の対応も必須）

### 開発者オプションを有効にする

デスクトップモードは開発者オプション内にある設定項目なので、まずは以下の操作で開発者オプションを有効化する

```
設定 > システム > 端末情報 > 「ビルド番号」を連打 
```

### デスクトップモードを有効にする

開発者オプションを開き、以下の項目をonにする
- デスクトップモードに強制的に切り替え（必須）
- フリーフォームウィンドウの有効化（推奨。アプリをウィンドウで表示できるようになる）

![alt text](/images/image_dev_option.png =300x)

### 外部モニタに繋いでみる

Displayport Alt Mode対応のUSB type-Cケーブルでディスプレイに接続してミラーリングすると、全画面表示になっている。また、例えばChromeを開くと、コレまでに開いたタブがPC版のように上部に並ぶようになっている。
![alt text](/images/image_pixel9.png)

### kiwiブラウザを導入する
スマートフォンでの開発でも、当然開発サーバーで動作確認を行いながら作業を進めていくが、mobile版のChromeを始めとしたブラウザにはDeveloper toolやchrome拡張機能が利用できない。kiwiブラウザを使うと、mobile環境でもchromeとほぼ同等にそれらの機能を利用できるため、Google Playからインストールして利用することをおすすめする

https://play.google.com/store/apps/details?id=com.kiwibrowser.browser&hl=ja

### おすすめスマホ
余談だが、SamsungのDeXやMotorolaのSmartConnectで有線接続に対応しているスマートフォンを使うと、このような設定をせずにPCライクにスマートフォンを使うことができる。実は、本記事一番上の画像はSmartConnectのもの。UI上も少しだけ使いやすいので、興味があると調べてみると良いかも。


## お片付け
これで、開発機で起動したサーバーをつかって、スマホでコーディング刷る環境が整った
本記事を終える前に、外部へ公開している起動しているサーバーを落としておく

### Nginx-Proxy-Manager
セットアップの際に作成した、compose.ymlがあるディレクトリで以下のコマンドを実行し、Nginx Proxy Managerを停止する
``` zsh
docker compose down
```

### code serve-web or code tunnel
実行中のターミナルで `ctrl + c` を押し、プロセスを終了する


## まとめ
PCを持ち歩かず、外出先でスマートフォンでコーディングする方法について紹介した。

### スマートフォンでの作業について
開発機でVSCode Serverを起動することで、PCを持ち歩かずとも問題なく作業ができるところまで環境を構築することができた。VSCode Serverには2種類の起動方法があるので、それぞれの特性に合わせた使い方をしてほしい。（正直、長々とセキュアに使うための方法も書いたが、`code serve-web`はLAN内で使うのが良いと思う）

### 対策前後の普段の荷物の変化
対策前からポーチに入れて持ち歩いているものがどれだけ低減した重量の和は 約1.24kg。思ったより減った。よい。
- スマートフォン
- ~~ミニPC（EM680）~~ → - 570g
- 折りたたみキーボード（MOBO KEYBOARD 2）
- 無線マウス（logicool M350s）
- ~~充電器（100w）~~ → - 178g
- usb-cケーブル 3本
- ~~Androidタブレット（Fire Max 11, PCモニタ兼用）~~ → - 489g
- タブレットスタンド

### 今後について
昨今、XREAL等の軽量なAR Glassも登場しており、外出先でもさらに便利に色々なことができるようになっていると思う。現状ではコレくらいがリーズナブルかと思われる範囲に検討したが、更に良い技術やガジェットが出てきた際にはもっと荷物が減らせないか試すことにしたい。
（tap strapみたいなのでいいやつ、出てこないかな）

以上
