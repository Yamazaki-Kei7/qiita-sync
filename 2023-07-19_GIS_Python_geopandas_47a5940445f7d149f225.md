<!--
title:   GeoPandasで空間結合を行い、ポリゴン内に含まれるポイントの属性情報をポリゴンの属性にカンマ区切りで付与する
tags:    GIS,Python,geopandas
id:      47a5940445f7d149f225
private: false
-->
# はじめに
PythonのGeoPandasライブラリは、地理空間データの操作を効率的に行えます。
ArcGISやQGISなどでデータ処理するより、高速（であることが多い）かつ履歴が残るため、私は好んで使用しています。
今回は、GeoPandasを使って以下の処理をしてみます。

- ①行政界ポリゴンと②学校ポイントを準備する
- 各行政界に存在する学校の名前を把握する
    - ①行政界ポリゴンに新しく属性を追加し、対象の行政界に含まれる②学校の名前をカンマ区切りで入力する

使用したPythonとGeoPandasのバージョンは以下の通りです。
Python: 3.10.9
GeoPandas: 0.9.0


# データの準備
今回使用するデータは、[国土数値情報ダウンロードサイト](https://nlftp.mlit.go.jp/)から群馬県のデータをダウンロードして使用します。

- 行政区域データ
https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-N03-v3_1.html
- 学校データ
https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-P29-v2_0.html

# コード
## 必要なライブラリのインポート
`python:python
import geopandas as gpd
import pandas as pd
import matplotlib.pyplot as plt
`

## データの読み込みと確認

行政界のポリゴンデータを読み込みます。
`python:python
admin_area = gpd.read_file("data/N03-20230101_10_GML/N03-23_10_230101.shp", encoding="shift-jis")
admin_area.head()
`
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/36b34a1a-ddcf-5fb8-d7a7-417a2d1d7436.png)


学校のポイントデータを読み込みます。
`python:python
school_point = gpd.read_file("data/P29-21_10_GML/P29-21_10.shp", encoding="shift-jis")
school_point.head()
`
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/8b84778f-49de-f751-af7a-fd9df8e69a3b.png)

## データをプロットして確認する
それぞれのデータの位置関係を確認してみます。

`python:python
fig, ax = plt.subplots(figsize=(10, 10))
admin_area.plot(ax=ax)
school_point.plot(ax=ax, color="red", markersize=1)
`
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/e66ea087-b9e9-271f-5b60-e15436b65d18.png)

## 2つのデータを空間結合する
行政界ポリゴンと学校ポイントは`geopandas.sjoin`関数を使って、空間結合させます。
https://geopandas.org/en/stable/docs/reference/api/geopandas.sjoin.html

`sjoin`関数の特徴は以下の通りです。
- pandasの`merge`関数とほぼ同じように使用できる
- 引数の`op`に空間的位置関係を指定する（GeoPandasのバージョンによっては`op`ではなく`predicate`）
- `op`のデフォルト値は`intersects`

[GeoPandasのドキュメント](https://geopandas.org/en/stable/docs/user_guide/mergingdata.html#binary-predicate-joinsintersects)を参照すると、 `op`で指定できる空間的位置関係は以下の通りです。
- contains
- within
- touches
- crosses
- overlaps

以上を踏まえ、行政界ポリゴンと学校ポイントを空間結合します。
空間結合した後は、`groupby`関数を活用しつつ、学校の名前を行政界単位でまとめます。
まとめた結果を`merge`関数を使って行政界ポリゴンに結合させて完成です。
`python:python
# 行政界ポリゴンと学校の間の空間結合を行う
admin_with_schools = gpd.sjoin(admin_area, school_point, how='left', op='contains')

# 学校の名前をカンマ区切りで結合する
schools_by_admin = admin_with_schools.groupby('OBJECTID')['P29_004'].apply(lambda x: ', '.join(x.dropna())).reset_index()

# 行政界ポリゴンの任意の列に学校の名前を入力する
admin_area['school_names'] = admin_area.merge(schools_by_admin, left_on='OBJECTID', right_on='OBJECTID', how='left')['P29_004']

# 結果を表示する
admin_area.head()
`
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/81adf928-0837-62e9-8a6a-a8281e130dce.png)

## 処理した結果の確認
処理した結果をQGISで確認してみます。
上手に処理できました！！
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/8ef42f51-b1b2-7842-bdbc-8f2113df7616.png)