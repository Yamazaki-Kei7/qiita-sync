<!--
title:   Mergin Maps CE (Community Edition) をEC2にデプロイする
tags:    AWS,QGIS,merginmaps
id:      96bc6f6876bbe33f7144
private: false
-->
:::note info
これは MIERUNE AdventCalendar 2023 9日目の記事です! 昨日は@darshuさんによる [地形で栓抜きを作れと言われたら](https://qiita.com/darshu/items/d9e71725a26112979eb5) でした。
:::

# Mergin Maps
<img width="300" src="https://assets-global.website-files.com/64a7b431134d96b4b5f9c5b9/6513f4efd71c4c7e7977cdbc_webapp_tablet.webp">

Mergin Mapsは、 Lutra Consulting Limitedによって開発された地理空間データの収集および共有プラットフォームです。

地理空間データの収集を効率的に行えるシステム・アプリには、「Qfield」や「ArcGIS Field Maps」などがありますが、Mergin Mapsは以下のような特徴を持っています。

1. QGISとの統合
    - Mergin MapsはQGISの機能をモバイルデバイス向けに拡張し、利用者が簡単に調査プロジェクトを作成し、最新の情報をすべてのチームメンバーと共有できます​。
    - QGISのMergin Mapsプラグインは、QGIS内でMergin Mapsプロジェクトを直接操作することを可能にします。PCへのプロジェクトのダウンロード、プロジェクトの変更、およびクラウドへの同期を行えます。
1. データ収集
    - モバイルデバイスを使って地理空間データを簡単に記録でき、紙のフィールドノートからの手書き転写、写真の整理などの手間を省くことができます。モバイルデバイスでは専用のアプリを使用します。

Mergin Mapsは、Mergin Maps cloudを使用することで、データを複数端末間でシームレスに同期します。Mergin Maps cloudは、SaaSとして利用することもできますが、利用者が用意したサーバ（例えば、AWS EC2）にMergin Maps cloudを構築し、利用することもできます。

## SaaSとしてのMergin Maps
Mergin MapsをSaaSとして利用する場合、以下のプランが選択できます。
金額や利用可能なサービス内容については、[HP](https://merginmaps.com/pricing)を確認してください。
- Individual
- Professional
- Team

## Private Hosting
SaaSとしてMergins Mapsを利用する場合、データ容量などに制限がつきますが、自分でサーバを用意することで、無制限にデータを保存することが可能になります。
Private Hostingには以下2つのプランがあり、利用できるサービスに違いがあります。詳しくは[HP](https://merginmaps.com/pricing-for-ce-and-ee)を確認してください。
- Mergin Maps CE (Community Edition）（無料）
- Mergin Maps EE (Enterprise Edition)（有料）

今回はAWSのEC2にMergin Maps CEを構築してみます。

# EC2にMergin Maps CEをデプロイする手順
![merginmaps-merginmaps.drawio (1).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/f48cf57b-c3d0-74bc-e1f2-39a26fd7533e.png)

システム構成としては、上図のようになります。
以下の作業が完了していることを前提として、解説していきます。


- ACMでSSL証明書を発行する。
- EC2インスタンスを起動する。
- ELBを生成し、
    - セキュリティグループをHTTPS：443およびHTTP：80ポートへの接続を許可する。
    - ターゲットグループに起動したEC2インスタンスを配置する。
    - ACMで発行したSSL証明書を設定する。
- EC2インスタンスのセキュリティグループに、上記ELBから8080ポートへの接続を許可する（使用端末からのSSH接続も合わせて許可する）。
- Route53パブリックホストゾーンを作成する。
- Route53でELBのドメイン名を指すAレコード（エイリアス）を作成する。


上記の準備が完了したら、
まずは、SSHクライアントを使用して、EC2インスタンスに接続しておきます。

## EC2に必要なソフトをインストールする
EC2内のパッケージをupdateしておきます。
`
$ sudo yum update -y
`
### Dockerのインストール
まずはDockerをインストールします。

```
$ sudo yum -y install docker
```
Dockerがインストールできたら、Dockerを起動します。
```
$ sudo systemctl start docker
```
Dockerが起動していることを確認します。active(running)と表示されていれば、正常に起動しています。
```
$ systemctl status docker
```

#### Dockerの自動起動を有効化
Dockerの自動起動を有効にします。
これで、EC2を再起動しても自動でDockerが起動します。
`
$ sudo systemctl enable docker
`

#### ec2-userにdockerグループを追加
Dockerをインストールすると、OSにdockerグループが自動作成されます。
そのdockerグループにec2-userを追加します。
これでec2-userはsudoなしでdockerコマンドを実行できます。
`
$ sudo usermod -a -G docker ec2-user
`
いったんEC2から`exit`コマンドでログアウトし、再度EC2にec2-userでSSH接続します。
それ以降の接続では、ec2-userにdockerグループのアクセス権限が付与された状態になります。

### Docker Compose(docker-compose)のインストール
はじめに`/usr/local/lib/`配下にディレクトリ`/usr/local/lib/docker/cli-plugins/`を作成します。このあとの手順でDocker Composeのバイナリファイルを格納します。
`
$ sudo mkdir -p /usr/local/lib/docker/cli-plugins
`
変数`VER`にDocker Composeのバージョンを代入します。バージョンの数字は最新安定版のものに変えてください（最新安定版のバーションは[docker docsの公式サイト](https://matsuand.github.io/docs.docker.jp.onthefly/compose/install/)で確認してください）。
`
$ VER=2.4.1
`
GitHubにあるDocker Composeのバイナリファイルを、上記で作成した`/usr/local/lib/docker/cli-plugins/`にダウンロードします。
`
$ sudo curl \
  -L https://github.com/docker/compose/releases/download/v${VER}/docker-compose-$(uname -s)-$(uname -m) \
  -o /usr/local/lib/docker/cli-plugins/docker-compose
`
ダウンロードしたdocker-compose（バイナリファイル）に実行権限を付与します。
`
$ sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
`
次にシンボリックリンクを設定します。
`
$ sudo ln -s /usr/local/lib/docker/cli-plugins/docker-compose /usr/bin/docker-compose
`
docker-composeコマンドが使用できることを確認します。
`Docker Compose version v2.4.1`が表示されていればDocker Composeのインストールは成功です。
`
$ docker-compose --version
Docker Compose version v2.4.1
`

### Gitのインストール
最後にGitをインストールします。
`
$ sudo install -y git
`

## Mergin Mapsのデプロイ
### GitでCloneする
Mergin Mapsの[レポジトリ](https://github.com/MerginMaps/mergin)をCloneします。
`
$ git clone https://github.com/MerginMaps/mergin.git
`

### Dockerコンテナを起動する
以下のコマンドを実行し、Dockerコンテナを起動させます。
5つのコンテナが起動することを確認します。
`
$ cd mergin
$ mkdir -p projects
$ sudo chown -R  901:999 ./projects/
$ sudo chmod g+s ./projects/
$ docker-compose -f docker-compose.yml up
[+] Running 5/5
 ⠿ Container merginmaps-proxy   Started                                                                    2.0s
 ⠿ Container merginmaps-db      Started                                                                    2.0s
 ⠿ Container merginmaps-redis   Started                                                                    2.0s
 ⠿ Container merginmaps-server  Started                                                                   10.8s
 ⠿ Container merginmaps-web     Started                                                                    0.8s
....
`
サーバーを初めて起動する場合は、データベースを初期化し、スーパーユーザーを作成する必要があります。管理者のユーザー名、パスワード、電子メールを設定します。
`
$ docker exec merginmaps-server flask init-db
$ docker exec merginmaps-server flask user create <username> <password> --is-admin --email <email>
`

## アプリケーションの確認

Route53で作成したレコード名を使用して、ELBへHTTPS接続してみます。

接続に成功すると、以下の画面が表示されます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/769bbc2c-c525-a6a8-ab51-522a3467ac0a.png)

登録した「管理者のユーザー名」と「パスワード」を入力し、サインインします。
サインインするとDashboad画面に遷移します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/6815e632-28ac-4929-55c5-5af46b5ddd64.png)

### QGISからプロジェクトを作成し、Mergin Mapsへ同期する
QGISを起動し、プラグイン「Mergin」をインストールします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/6e601083-0f12-a482-2c8a-a8107e392e1f.png)

ブラウザパネルに`Mergin Maps`が表示されることを確認し、右クリックして表示される`Configure`をクリックします。
<img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/17bec1a3-bf4b-ef4b-7d7e-2a46cdb924f4.png">

`Custom Mergin Server`にチェックをいれることで、自分で立ち上げたMergin MapsのServerとQGISを連携することができます。

`Username`と`Password`、`Custom Mergin Server`には、Webブラウザでアクセスする時に使用したものを入力します。

入力して「OK」し、ブラウザパネルの`Mergin Maps`を右クリックすると、選択できる項目がふえていることが確認できます。

それでは、新しくプロジェクトを作成してみます。`Create new project`をクリックします。
<img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/35ec326c-a4d9-9b24-8e34-2ec0b0ef47cc.png">

`New basic QGIS project`をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/8711db5a-b622-b3d4-9f32-659ccd677efa.png)

`Project Name`とServerと同期させるローカルフォルダを選択し、「Finish」をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/ef2a01f3-5a15-20ef-b615-5f55150b1c50.png)

デフォルトで用意されているプロジェクトが作成されました。
調査時のメモや写真が登録できるポイントレイヤ`Survey`とラスタータイル`OpenMapTiles (OSM)`がレイヤパネルに確認できます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/9fc4ba47-7c12-0d4c-2258-b458cb03bdea.png)

先月、福井に行ったので福井駅前のメモを登録してみます。
PCから写真を登録する時は、写真ファイルをプロジェクトフォルダの中にコピー・移動させてから登録してください（そうしないと、写真ファイルがEC2のServerに同期されないため）。
<img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/8874e826-e43e-b2ba-aba6-c336060f2306.png">

データが登録できたら、ブラウザパネルから`Mergin Maps`> `sample`を右クリックし、`Synchronize`をクリックします。
<img width="300" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/b234fef1-31e0-65ea-034c-93431a0cdc21.png">

「Sync」をクリックします。
<img width="300" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/28429f27-bbe6-84b3-3860-e555fdec7a4c.png">

ブラウザからEC2で立ち上げたMergin Mapsを確認します。
画面左のProjectsをクリックすると、先ほど作成したsampleプロジェクトが同期されていることが確認できます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/d45017dd-ea15-6b12-da3c-c5c23caa0fe7.png)

写真ファイルを保存しておいた`DCIM`フォルダを見てみると、写真も同期できたことが確認できます。
<img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/19688ab3-760b-8bbb-af7b-21bbf0325761.png">

### モバイルアプリMergin Mapsでプロジェクトを操作する
　モバイル端末（iOS、Androidのどちらでも良い）でMergin Mapsのアプリをダウンロードしておきます。

アプリを起動し、画面右上のアカウントボタンをタップし、ログイン画面に遷移します。
ログイン画面では「ユーザ名」と「パスワード」を入力し、画面下の鉛筆マークの右側にRoute53で作成したレコード名を入力します。
入力できたらサインインします。
<img width="200" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/57f86652-4364-b125-8932-b56cfde93615.jpeg">

サインに成功すると、プロジェクト一覧にQGISで作成・同期したプロジェクトが表示されていることが確認できます。
<img width="200" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/0b8c73e1-63ea-4b31-ce8e-38f14fa12e74.png">

プロジェクト名をタップすると、モバイル端末からQGISで作成したプロジェクトを閲覧することができます。もちろん編集も可能です。
<img width="200" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/a412da75-8014-f10d-8de7-a6c9efde5681.png">

地物の属性情報も閲覧することができました。
<img width="200" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/3b98cc1b-7da1-0bb0-9882-a01335fd9e2d.jpeg">

# 終わりに
AWSのEC2にMergin Mapsのクラウド環境を構築できました。
QGISおよびモバイルアプリ「Mergin Maps」から上記クラウドのデータにアクセス・同期することも確認できました。

Mergin Maps CEはサーバさえ用意できれば、無料で利用できます。
とりあえず試してみたい人は使ってみると良いのではないでしょうか。

Mergin Mapsによる「地理空間データの収集および共有」が気に入った会社は、セキュリティアップデート等の追加機能がついた有料版Mergin Maps EEを契約し、Mergin Mapsのエコシステムを維持するための資金提供をしていただくのが望ましいと思います。

# 参考
- [【AWS】EC2にDockerとDocker Composeをインストール](https://kacfg.com/aws-ec2-docker/)
- [httpで公開したWebサーバーをhttps通信できるようにする](https://zenn.dev/nozomi_iida/articles/20220216_to_https_with_aws)
- [パブリックALB(HTTPS) → プライベートEC2構築](https://qiita.com/tannh/items/f1dbe7bfc2dca955fda4)
- [Mergin Maps Community Edition
](https://merginmaps.com/docs/dev/mergince/#mergin-maps-community-edition)


:::note info
明日は@soramiさんによる記事です！お楽しみに！！
:::