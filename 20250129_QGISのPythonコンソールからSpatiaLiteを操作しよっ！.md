<!--
title:   QGISのPythonコンソールからSpatiaLiteを操作しよっ！
tags:    GIS,QGIS,SQlite,SpatiaLite
private: true
-->

# はじめに

個人的な作業メモです。
なお、Windows かつ、QGIS のバージョンは 3.34.15 です。

# メモ

## はじめかた

まずは QGIS を開き、Python コンソールを起動します。
そして、エディタを開きましょう。

![1738140475400](image/20250129_QGISのPythonコンソールからSpatiaLiteを操作しよっ！/1738140475400.png)

エディタに以下を記述して、拡張機能である SpatiaLite を読み込みます。

```python
import sqlite3

db_path = r"C:hoge\db.sqlite"
con = sqlite3.connect(db_path)

# 拡張を有効
con.enable_load_extension(True)
cur = con.cursor()
cur.execute("SELECT load_extension('mod_spatialite')")

# メタデータの初期化
try:
    cur.execute("SELECT InitSpatialMetadata()")
except sqlite3.OperationalError as e:
    # すでに初期化済みの場合はエラーが出ることがある
    print("Spatial metadata initialization might already be done:", e)
```

これで、Cursor から、SpatiaLite を操作できるようになります。

## テーブルの作成

SpatiaLite の`AddGeometryColumn`関数を使って、ジオメトリカラムを追加します。
これにより、対象のテーブルにジオメトリカラムを追加することができます。

`AddGeometryColumn`関数の Syntax は以下の通りです。

```sql
AddGeometryColumn(
    table String,
    column String,
    srid Integer,
    geom_type String
    [ , dimension String [ , not_null Integer ] ]
  ) : Integer
```

```python
TABLE_NAME_POINT = "point_table"
# テーブルの作成
cur.execute(f"DROP TABLE IF EXISTS {TABLE_NAME_POINT}")  # 既存テーブルを削除する
cur.execute(f"CREATE TABLE {TABLE_NAME_POINT} (id INTEGER PRIMARY KEY, name TEXT)")
# ジオメトリカラムを追加（ここがSpatiaLiteの特徴）
cur.execute(f"""
    SELECT AddGeometryColumn('{TABLE_NAME_POINT}', 'geom', 4326, 'POINT', 'XY')
""")
```

上の例では、ポイントデータを格納するテーブルを作成しています。
他には、以下のような`geom_type`が指定できます。

- **ポイントデータ**
  - POINT
  - POINTZ
  - POINTM
  - POINTZM
- **ラインデータ**
  - LINESTRING
  - LINESTRINGZ
  - LINESTRINGM
  - LINESTRINGZM
- **ポリゴンデータ**
  - POLYGON
  - POLYGONZ
  - POLYGONM
  - POLYGONZM
- **マルチポイントデータ**
  - MULTIPOINT
  - MULTIPOINTZ
  - MULTIPOINTM
  - MULTIPOINTZM
- **マルチラインデータ**
  - MULTILINESTRING
  - MULTILINESTRINGZ
  - MULTILINESTRINGM
  - MULTILINESTRINGZM
- **マルチポリゴンデータ**
  - MULTIPOLYGON
  - MULTIPOLYGONZ
  - MULTIPOLYGONM
  - MULTIPOLYGONZM
- **ジオメトリコレクションデータ**
  - GEOMETRYCOLLECTION
  - GEOMETRYCOLLECTIONZ
  - GEOMETRYCOLLECTIONM
  - GEOMETRYCOLLECTIONZM
- **ジオメトリ**
  - GEOMETRY
  - GEOMETRYZ
  - GEOMETRYM
  - GEOMETRYZM

## CRUD 操作

### Create

データの登録は以下のように行います。
位置情報は SpatiaLite の`GeomFromText`関数を使って、WKT 形式で指定しています。

`GeomFromText`関数の Syntax は以下の通りです。

```sql
GeomFromText( wkt String [ , SRID Integer] ) : Geometry
```

```python
# データの登録
cur.execute(f"""
    INSERT INTO {TABLE_NAME_POINT} (id, name, geom)
    VALUES (
        1,
        'Test Point',
        GeomFromText('POINT(139.767052 35.681167)', 4326)
    )
""")
```

### Read

特に記述することなし。。

### Update

データの更新は以下のように行います。
UPDATE 文の`SET`句で、更新したいカラムを指定します。
更新対象は`WHERE`句で指定します。

```python
# 例1: nameだけを更新する場合
cur.execute(f"""
    UPDATE {TABLE_NAME_POINT}
    SET name = ?
    WHERE id = ?
""", ("Updated Point Name", 1))
```

位置情報を更新する時は、登録の時と同様に、`GeomFromText`関数を使って、WKT 形式で指定します。

```python
# 例2: geom も更新する場合
cur.execute(f"""
    UPDATE {TABLE_NAME_POINT}
    SET name = ?,
        geom = GeomFromText(?, 4326)
    WHERE id = ?
""", ("Updated Point Name 2", "POINT(140.000 36.000)", 1))
```

### Delete

データの削除は以下のように行います。
DELETE 文の`WHERE`句で、削除したいデータを指定します。

```python
# データの削除
cur.execute(f"""
    DELETE FROM {TABLE_NAME_POINT}
    WHERE id = ?
""", (1,))
```

## 他のファイル形式からのデータ取り込み

### GeoJSON からのデータ取り込み

SpatiaLite の`ImportGeoJSON`GeoJSON からデータを取り込むことができます。

`ImportGeoJSON`関数の Syntax は以下の通りです。

```sql
ImportGeoJSON( filename Text , table Text ) : Integer

-- または

ImportGeoJSON( filename Text , table Text [ , geo_column Text [ , spatial_index Boolean [ , srid Interger [ , colname_case Text ]]]] ) : Integer

```

注意点として、潜在的なセキュリティ問題があるため、デフォルトでは`ImportGeoJSON`関数は無効になっています。
環境変数`SPATIALITE_SECURITY=relaxed`を明示的に設定することが必要ですが、利用の際は気をつける必要があります。

```python
# SpatiaLite で ImportGeoJSON 関数を使うには、セキュリティ上の理由により以下の設定が必要
import sqlite3
import os
os.environ["SPATIALITE_SECURITY"] = "relaxed"

db_path = r"C:hoge\db.sqlite"
con = sqlite3.connect(db_path)

# 拡張を有効
con.enable_load_extension(True)
cur = con.cursor()
cur.execute("SELECT load_extension('mod_spatialite')")

# ~~ 省略 ~~

geojson_path = r"C:\hoge\hoge.geojson"
TABLE_NAME_GEOJSON = "geojson_table"
# GeoJSON の取り込み
cur.execute(f"""
    SELECT ImportGeoJSON(
        '{geojson_path}',     -- filename
        '{TABLE_NAME_GEOJSON}',  -- table
        'geom',               -- geo_column
        1,                    -- spatial_index (TRUE)
        4326,                 -- SRID
        'NONE'                -- colname_case
    );
""")
```

### Shapefile からのデータ取り込み

SpatiaLite の`ImportSHP`GeoJSON からデータを取り込むことができます。
こちらも、環境変数`SPATIALITE_SECURITY=relaxed`を明示的に設定することが必要です。

`ImportSHP`関数の Syntax は以下の通りです。

```sql
ImportSHP( filename Text , table Text , charset Text ) : Integer

-- または

ImportSHP( filename Text , table Text , charset Text [ , srid Integer [ , geom_column Text [ , pk_column Text [ , geometry_type Text [ , coerce2D Integer [ , compressed Integer [ , spatial_index Integer [ , text_dates Integer [ , colname_case Text [ , update_statistics Integer [ , verbose Integer ] ] ] ] ] ] ] ] ] ] ] ) : Integer

```

## 空間解析

## 空間インデックスの作成

作業メモ
現状のスクリプト

```python
import sqlite3
import os
os.environ["SPATIALITE_SECURITY"] = "relaxed"

db_path = r"C:\Users\yamakei\Desktop\test\db.sqlite"
con = sqlite3.connect(db_path)

# 拡張を有効
con.enable_load_extension(True)
cur = con.cursor()
cur.execute("SELECT load_extension('mod_spatialite')")

# メタデータの初期化
try:
    cur.execute("SELECT InitSpatialMetadata()")
except sqlite3.OperationalError as e:
    # すでに初期化済みの場合はエラーが出ることがある
    print("Spatial metadata initialization might already be done:", e)

# テーブルの作成
TABLE_NAME_POINT = "point_table"
# テーブルの作成
cur.execute(f"DROP TABLE IF EXISTS {TABLE_NAME_POINT}")  # 既存テーブルを削除する
cur.execute(f"CREATE TABLE {TABLE_NAME_POINT} (id INTEGER PRIMARY KEY, name TEXT)")
# ジオメトリカラムを追加（ここがSpatiaLiteの特徴）
cur.execute(f"""
    SELECT AddGeometryColumn('{TABLE_NAME_POINT}', 'geom', 4326, 'POINT', 'XY')
""")

# データの登録
cur.execute(f"""
    INSERT INTO {TABLE_NAME_POINT} (id, name, geom)
    VALUES (
        1,
        'Test Point',
        GeomFromText('POINT(139.767052 35.681167)', 4326)
    )
""")

# ==== 更新 (UPDATE) ====
# 例1: nameだけを更新する場合
cur.execute(f"""
    UPDATE {TABLE_NAME_POINT}
    SET name = ?
    WHERE id = ?
""", ("Updated Point Name", 1))

# 例2: geom も更新する場合
cur.execute(f"""
    UPDATE {TABLE_NAME_POINT}
    SET name = ?,
        geom = GeomFromText(?, 4326)
    WHERE id = ?
""", ("Updated Point Name 2", "POINT(140.000 36.000)", 1))

# データの削除
cur.execute(f"""
    DELETE FROM {TABLE_NAME_POINT}
    WHERE id = ?
""", (1,))

# GeoJSONの取り込み
geojson_path = r"C:\Users\yamakei\Desktop\test\P29-21_10.geojson"
TABLE_NAME_GEOJSON = "geojson_table"
cur.execute(f"""
    SELECT ImportGeoJSON(
        '{geojson_path}',     -- filename
        '{TABLE_NAME_GEOJSON}',  -- table
        'geom',               -- geo_column
        1,                    -- spatial_index (TRUE)
        4326,                 -- SRID
        'NONE'                -- colname_case
    );
""")

con.commit()
con.close()
```
