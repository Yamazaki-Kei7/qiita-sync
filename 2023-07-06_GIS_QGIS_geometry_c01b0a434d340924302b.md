<!--
title:   QGISで横断線をサクッと作成する
tags:    GIS,QGIS,geometry
id:      c01b0a434d340924302b
private: false
-->
# はじめに
QGISを使用していると、作成したラインデータから等間隔の横断線を作成したい・・・ということが良くあります。

今回は、以下の記事を参考に横断線を作成してみました。
https://geoobserver.wordpress.com/2023/03/27/qgis-tipp-flussbreiten-ermitteln-aber-wie/

なお、説明図の背景地図として[地理院タイル](https://maps.gsi.go.jp/development/ichiran.html)を加工して利用しています。

# 作成手順
## フロー図
`mermaid
graph TB
  A[/ラインデータ/] --> B["（プロセシング）ジオメトリに沿った等間隔点群"] --> C["（プロセシング）式によるジオメトリ"]
`
## やりかた
### ラインデータの準備
まずは横断線を作成するためのラインデータを作成(またはダウンロード）します。
今回は1本だけ河川に沿って作成しました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/9b03d4ed-3528-6723-4574-1a1bdc1946a1.png)


## ライン沿いに等間隔のポイントを作成する
プロセシング「ジオメトリに沿った等間隔点群」で横断線を作成するのに必要なポイントデータを作成します。
今回は500m間隔で横断線を作成したいので、パラメータ「距離」に`500`を指定します。
![スクリーンショット 2023-06-21 17.34.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/be760a68-4896-185e-4061-46551843a460.png)

等間隔にポイントを作成することができます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/36eecb97-864d-7a22-684d-21858b30b9b9.png)

### 横断線を作成する
作成したポイントを使って横断線を作成します。
プロセシング「式によるジオメトリ」に以下の式を記載することで、横断線がサクッと作成できます。
今回はポイントから左右に800m延伸した横断線を作成したいので、`800`という数字を使用しています。
`90`という数字は、ポイントの向きから垂直方向に向けてラインを作成する・・・ということを意味しています。

```sql:ジオメトリの式
extend(
   make_line(
      $geometry,
       project (
          $geometry,
          800,
          radians("angle"-90))
        ),
   800,
   0
)
```

![スクリーンショット 2023-06-21 17.53.57.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/d2e8f2d1-602f-be41-e264-f9c5ed43ccf7.png)


プロセシングを走らせると、簡単に横断線を作成することができました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/0abc4696-6b3e-7189-6062-3cce7e8e6737.png)

作成した横断線は断面図の作成に利用するなど、色々と活用することができます。

# おまけ
ポイント（ラインから等間隔に作成したデータ）のスタイル設定を変えるだけで、横断線を**見かけ上**作成できます。

レイヤのシンボロジは以下の手順で変更します。

 - ポイントのシンボルレイヤ型を`ジオメトリジェネレータ`に設定する
 - ジオメトリ型は`ラインストリング/マルチラインストリング`にする
 - プロセシング「式によるジオメトリ」で使用した式を「式ダイアログ」に入力する

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/0eea9f19-24a6-e4c4-170e-dc09008f4906.png)

こうすることで、角度や横断線の長さを変更した時、リアルタイムで地図に反映することができます。
便利ですね〜。