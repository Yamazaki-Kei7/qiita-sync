# はじめに

<!--
title:   GDALでCSVファイルをGeoPackageファイルに変換する
tags:    GIS,GDAL,python
private: true
-->

以下の流れで GDAL を使用し、CSV ファイルを GeoPackage ファイルに変換するテクニックを身につけたため、備忘録として残します。

経緯

1. ogr2ogr コマンドで CSV ファイルを GeoPackage ファイルに変換
1. 変換されたが、属性情報が全て文字列型となった
1. VRT ファイルを用いて属性情報の型を指定して変換する手法を試す
1. 変換成功。ただし、属性情報を VRT ファイルに直接記述するのは手間がかかる
1. 属性情報の型を Python の pandas で取得し、VRT ファイルに書き込む方法を採用
1. CSV ファイルを GeoPackage ファイルにうまく変換できるようになった

# 実行手順

## 1. 使用するツール

- M2MAC
- GDAL
- python
- pandas

## 2. データの準備

以下のサイトからデータ「ケース 01\_堤防破堤」をダウンロードして、解凍します。
<https://www.geospatial.jp/ckan/dataset/dataset>

## 3. CSV ファイルを GeoPackage ファイルに変換

### 3.1. 変換手順

以下のコマンドで変換してみます。

Shift-JIS の CSV ファイルは ogr2ogr コマンドで読み込めないため、まず UTF-8 に変換します。nkf コマンドを使用して以下のように変換します。

```terminal
nkf -w --overwrite 浸水メッシュ_最大クラスケース01_満潮位_堤防破堤_12h_01系.csv
```

次に、以下のコマンドを実行して GeoPackage ファイルに変換します。

```terminal
FILE_NAME="浸水メッシュ_最大クラスケース01_満潮位_堤防破堤_12h_01系"
LAYER_NAME="浸水_01"

ogr2ogr -f GPKG "${FILE_NAME}.gpkg" "${FILE_NAME}.csv" -nln $LAYER_NAME -oo X_POSSIBLE_NAMES=緯度 -oo Y_POSSIBLE_NAMES=経度
```

### 3.2. 変換結果

変換された GeoPackage の属性情報を確認します。

```terminal
ogrinfo -al -so 浸水メッシュ_最大クラスケース01_満潮位_堤防破堤_12h_01系.gpkg

> INFO: Open of `浸水メッシュ_最大クラスケース01_満潮位_堤防破堤_12h_01系.gpkg'
>       using driver `GPKG' successful.
>
> Layer name: 浸水_01
> Geometry: Point
> Feature Count: 360680
> Extent: (128.395388, 27.019122) - (130.280255, 33.376882)
> Layer SRS WKT:
> (unknown)
> FID Column = fid
> Geometry Column = geom
> 経度: Real (0.0)
> 緯度: Real (0.0)
> 地殻変動後の標高（m）: String (0.0)
> 浸水深（m）: String (0.0)
> 隆起量（m）: String (0.0)
> 到達時間_01cm（秒）: String (0.0)
> 到達時間_30cm（秒）: String (0.0)
> 到達時間_01m（秒）: String (0.0)
> 到達時間_02m（秒）: String (0.0)
> 到達時間_03m（秒）: String (0.0)
> 到達時間_05m（秒）: String (0.0)
> 到達時間_10m（秒）: String (0.0)
> 到達時間_20m（秒）: String (0.0)
> 到達時間_30m（秒）: String (0.0)
> 到達時間_40m（秒）: String (0.0)
> 到達時間_最高水位（秒）: String (0.0)
```

変換後、すべての属性情報が文字列型になってしまいました。例えば、標高や浸水深などは数値型にしたい場合があります。

## 4. VRT ファイルを使って属性情報の型を指定して変換

### 4.1. 変換手順

以下の内容で VRT ファイル「浸水メッシュ*最大クラスケース 01*満潮位\_堤防破堤\_12h_01 系.vrt」を新規作成します。
座標系や、XY 座標を示す列、各列の属性情報を以下のように記述します。

```xml
<OGRVRTDataSource>
    <OGRVRTLayer name="浸水メッシュ_最大クラスケース01_満潮位_堤防破堤_12h_01系">
        <SrcDataSource>浸水メッシュ_最大クラスケース01_満潮位_堤防破堤_12h_01系.csv</SrcDataSource>
        <GeometryType>wkbPoint</GeometryType>
        <LayerSRS>EPSG:6668</LayerSRS>
        <GeometryField encoding="PointFromColumns" x="緯度" y="経度"/>
        <Field name="経度" type="Real" />
        <Field name="緯度" type="Real" />
        <Field name="地殻変動後の標高（m）" type="Real" />
        <Field name="浸水深（m）" type="Real" />
        <Field name="隆起量（m）" type="Real" />
        <Field name="到達時間_01cm（秒）" type="Real" />
        <Field name="到達時間_30cm（秒）" type="Real" />
        <Field name="到達時間_01m（秒）" type="Real" />
        <Field name="到達時間_02m（秒）" type="Real" />
        <Field name="到達時間_03m（秒）" type="Real" />
        <Field name="到達時間_05m（秒）" type="Real" />
        <Field name="到達時間_10m（秒）" type="Real" />
        <Field name="到達時間_20m（秒）" type="Real" />
        <Field name="到達時間_30m（秒）" type="Real" />
        <Field name="到達時間_40m（秒）" type="Real" />
        <Field name="到達時間_最高水位（秒）" type="Real" />
    </OGRVRTLayer>
</OGRVRTDataSource>
```

今回はすべての属性を `Real`に指定しましたが、以下のような型が指定できます。

- Integer
- Real
- String
- Date
- Time
- DateTime
- Binary
- IntegerList
- RealList
- StringList

VRT ファイルを使って、GeoPackage ファイルに変換します。

```terminal
ogr2ogr -f GPKG $"{FILE_NAME}.gpkg" "${FILE_NAME}.vrt"
```

### 4.2. 変換結果

変換された GeoPackage の属性情報を確認します。
すべての属性情報が数値型である `Real`になっていることが確認できます。

```terminal
ogrinfo -al -so 浸水メッシュ_最大クラスケース01_満潮位_堤防破堤_12h_01系.gpkg

> INFO: Open of `浸水メッシュ_最大クラスケース01_満潮位_堤防破堤_12h_01系.gpkg'
>      using driver `GPKG' successful.
>
> Layer name: 浸水メッシュ_最大クラスケース01_満潮位_堤防破堤_12h_01系
> Geometry: Point
> Feature Count: 360680
> Extent: (128.395388, 27.019122) - (130.280255, 33.376882)
> Layer SRS WKT:
> GEOGCS["JGD2011",
>    DATUM["Japanese_Geodetic_Datum_2011",
>        SPHEROID["GRS 1980",6378137,298.257222101,
>            AUTHORITY["EPSG","7019"]],
>        AUTHORITY["EPSG","1128"]],
>    PRIMEM["Greenwich",0,
>        AUTHORITY["EPSG","8901"]],
>    UNIT["degree",0.0174532925199433,
>        AUTHORITY["EPSG","9122"]],
>    AXIS["Latitude",NORTH],
>    AXIS["Longitude",EAST],
>    AUTHORITY["EPSG","6668"]]
> Data axis to CRS axis mapping: 2,1
> FID Column = fid
> Geometry Column = geom
> 経度: Real (0.0)
> 緯度: Real (0.0)
> 地殻変動後の標高（m）: Real (0.0)
> 浸水深（m）: Real (0.0)
> 隆起量（m）: Real (0.0)
> 到達時間_01cm（秒）: Real (0.0)
> 到達時間_30cm（秒）: Real (0.0)
> 到達時間_01m（秒）: Real (0.0)
> 到達時間_02m（秒）: Real (0.0)
> 到達時間_03m（秒）: Real (0.0)
> 到達時間_05m（秒）: Real (0.0)
> 到達時間_10m（秒）: Real (0.0)
> 到達時間_20m（秒）: Real (0.0)
> 到達時間_30m（秒）: Real (0.0)
> 到達時間_40m（秒）: Real (0.0)
> 到達時間_最高水位（秒）: Real (0.0)
```

## 5. 属性情報の型を python の pandas で取得して、VRT ファイルに書き込む

VRT ファイルに属性の型をべた書きするのは面倒です。

python の pandas を使って、CSV ファイルの属性情報の型を取得して、VRT ファイルに書き込みます。

### 5.1. 属性情報の型を取得する python スクリプト

以下の Python スクリプトを作成し、CSV ファイルの属性情報の型を取得して VRT ファイルに書き込みます。

```python:create_vrt.py
import sys

import pandas as pd


def create_vrt(filename, x_col, y_col, epsg=4326):
    df = pd.read_csv(filename)
    field_type_str = ""
    for col, dtype in df.dtypes.items():
        col = col.replace('"', """)
        if dtype == "float64":
            field_type = "Real"
        elif dtype == "int64":
            field_type = "Integer"
        else:
            field_type = "String"
        field_type_str += f'<Field name="{col}" type="{field_type}" />\n'

    # Create VRT file
    vrt_filename = filename.replace(".csv", ".vrt")
    with open(vrt_filename, "w") as f:
        f.write(f"""<OGRVRTDataSource>
    <OGRVRTLayer name="{filename.replace('.csv', '')}">
        <SrcDataSource>{filename}</SrcDataSource>
        <GeometryType>wkbPoint</GeometryType>
        <LayerSRS>EPSG:{epsg}</LayerSRS>
        <GeometryField encoding="PointFromColumns" x="{x_col}" y="{y_col}"/>
        {field_type_str}
    </OGRVRTLayer>
</OGRVRTDataSource>""")
    print(f"Created {vrt_filename}")

if __name__ == "__main__":
    # Check if sufficient arguments are provided
    if len(sys.argv) < 4:
        print("Usage: python script.py <filename> <x_col> <y_col> [epsg]")
        sys.exit(1)

    filename = sys.argv[1]
    x_col = sys.argv[2]
    y_col = sys.argv[3]
    epsg = int(sys.argv[4]) if len(sys.argv) > 4 else 4326  # Default to EPSG:4326 if not provided

    create_vrt(filename, x_col, y_col, epsg)

```

今回は、整数値と浮動小数点数の属性情報を `Integer` と `Real` に変換し、それ以外は `String` に変換しています。

### 5.2. 属性情報の型を取得する python スクリプトの実行

以下のコマンドで、python スクリプトを実行します。

```terminal
python create_vrt.py 浸水メッシュ_最大クラスケース01_満潮位_堤防破堤_12h_01系.csv 緯度 経度 4326
```

### 5.3. 実行結果

VRT ファイルに属性情報の型が正しく書き込まれていることが確認できます。

```xml
<OGRVRTDataSource>
    <OGRVRTLayer name="浸水メッシュ_最大クラスケース01_満潮位_堤防破堤_12h_01系">
        <SrcDataSource>浸水メッシュ_最大クラスケース01_満潮位_堤防破堤_12h_01系.csv</SrcDataSource>
        <GeometryType>wkbPoint</GeometryType>
        <LayerSRS>EPSG:4326</LayerSRS>
        <GeometryField encoding="PointFromColumns" x="緯度" y="経度"/>
        <Field name="経度" type="Real" />
<Field name="緯度" type="Real" />
<Field name="地殻変動後の標高（m）" type="Real" />
<Field name="浸水深（m）" type="Real" />
<Field name="隆起量（m）" type="Real" />
<Field name="到達時間_01cm（秒）" type="Real" />
<Field name="到達時間_30cm（秒）" type="Real" />
<Field name="到達時間_01m（秒）" type="Real" />
<Field name="到達時間_02m（秒）" type="Real" />
<Field name="到達時間_03m（秒）" type="Real" />
<Field name="到達時間_05m（秒）" type="Real" />
<Field name="到達時間_10m（秒）" type="Real" />
<Field name="到達時間_20m（秒）" type="Real" />
<Field name="到達時間_30m（秒）" type="Real" />
<Field name="到達時間_40m（秒）" type="Real" />
<Field name="到達時間_最高水位（秒）" type="Real" />

    </OGRVRTLayer>
</OGRVRTDataSource>
```

### 5.4. GeoPackage ファイルに変換

得られた VRT ファイルを使用して、最終的に GeoPackage ファイルに変換します。

```terminal
ogr2ogr -f GPKG $"{FILE_NAME}.gpkg" "${FILE_NAME}.vrt"
```

## 6. まとめ

- 大容量の CSV ファイルを Shapefile や GeoPackage ファイルに変換する際、処理速度やメモリ使用量を考慮すると、GDAL の ogr2ogr を使うのが適切だと思います。
- ogr2ogr を使うと属性情報が文字列型になる問題がありますが、VRT ファイルを用いることで解決可能です。
- Python の pandas を活用して、属性情報の型を取得し、VRT ファイルに書き込むことで、より効率的に変換作業を行うことができました。
