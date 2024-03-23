<!--
title:   QGISプラグインのテスト
tags:    Python,QGIS,pytest,テスト
id:      12bef24d138220393e4a
private: true
-->
# pytestでテストする
## 各種Fixtureの説明
### 1. `qgis_app` Fixture
  `python
  def test_with_qgis_app(qgis_app):
      # このテストでは、フルに設定された QgsApplication のインスタンスが利用可能です。
      # QgsApplication に関連する操作やテストをここで行うことができます。
  `

### 2. `qgis_bot` Fixture
  `python
  def test_with_qgis_bot(qgis_bot):
      # QgisBot を使用して、QGIS との一般的なインタラクションを行います。
      # 例えば、QGIS 内で特定のアクションを実行するためのメソッドを呼び出すことができます。
  `

### 3. `qgis_canvas` Fixture
  `python
  def test_with_qgis_canvas(qgis_canvas):
      # このテストでは、QgsMapCanvas オブジェクトが利用可能です。
      # マップキャンバスに関連する操作やレンダリングのテストが可能です。
  `

### 4. `qgis_parent` Fixture
  `python
  def test_with_qgis_parent(qgis_parent):
      # qgis_canvas の親として使用される QWidget が利用可能です。
      # これは、ウィジェット階層や親子関係に関連するテストに有用です。
  `

### 5. `qgis_iface` Fixture
  `python
  def test_with_qgis_iface(qgis_iface):
      # スタブ化された QgsInterface が利用可能です。
      # QGIS のインターフェースに関連する機能やテストを行うことができます。
  `

### 6. `qgis_new_project` Fixture
  `python
  def test_with_qgis_new_project(qgis_new_project):
      # 新しいプロジェクト環境を提供します。
      # このフィクスチャは、QgsProject にレイヤーを追加するなどのテストに適しています。
  `

### 7. `qgis_processing` Fixture
  `python
  def test_with_qgis_processing(qgis_processing):
      # 処理フレームワークが初期化されています。
      # `processing.run(...)` を呼び出すコードのテストに使用できます。
  `

### 8. `qgis_version` Fixture
  `python
  def test_with_qgis_version(qgis_version):
      # QGIS のバージョン番号を整数で取得できます。
      # バージョンに依存するテストや条件分岐に有用です。
  `

### 9. `qgis_world_map_geopackage` Fixture
  `python
  def test_with_qgis_world_map_geopackage(qgis_world_map_geopackage):
      # QGIS に付属の world_map.gpkg のパスが利用可能です。
      # この GeoPackage ファイルを使用してテストを行うことができます。
  `

### 10. `qgis_countries_layer` Fixture
  `python
  def test_with_qgis_countries_layer(qgis_countries_layer):
      # Natural Earth の国々のレイヤーが QgsVectorLayer として利用可能です。
      # このレイヤーを使って、地理データに関連するテストを行うことができます。
  `

  これらのフィクスチャは、QGIS の様々な機能やコンポーネントをテストする際に非常に便利です。フィクスチャを使用することで、テストコードのセットアップとクリーンアップを簡素化し、テストの焦点を実際の機能性やパフォーマンスに集中させることができます。

### 具体的な例
`python
def test_qgis_functionality(qgis_app, qgis_iface, qgis_countries_layer):
    # このテスト関数では、qgis_app, qgis_iface, qgis_countries_layer の3つのフィクスチャが使用されます。

    # qgis_app は QGIS アプリケーションインスタンスで、アプリケーションが正しく起動されているかをチェックします。
    assert qgis_app is not None, "QGIS application instance is not initialized."

    # qgis_iface は QGIS インターフェースで、特定のUI要素が存在するかを確認します。
    assert qgis_iface.mainWindow() is not None, "QGIS interface's main window is not available."

    # qgis_countries_layer は国々のデータレイヤーを提供し、レイヤーが正しくロードされているかをチェックします。
    assert qgis_countries_layer.isValid(), "Countries layer is not valid."
    assert qgis_countries_layer.featureCount() > 0, "Countries layer has no features."

    # さらに、レイヤーの中に特定の国が含まれているかをチェックすることもできます。
    countries = [f['NAME'] for f in qgis_countries_layer.getFeatures()]
    assert "Japan" in countries, "Japan is not in the countries layer."

    # また、QGIS インターフェースを介してマップキャンバスにレイヤーを追加することもできます。
    mapCanvas = qgis_iface.mapCanvas()
    mapCanvas.setLayers([qgis_countries_layer])
    mapCanvas.zoomToFullExtent()

    # マップキャンバスにレイヤーが正しく追加されたことを確認します。
    assert len(mapCanvas.layers()) == 1, "Map canvas does not have the expected number of layers."

```

## サンプル
### プラグインの起動を確認する

```python:プラグインの起動テスト.py
from qgis.core import QgsApplication
from qgis.gui import QgisInterface

from hogehoge_plugin import Hoge

def test_open_and_cancel_hoge(qgis_app: QgsApplication, qgis_iface: QgisInterface):
    plugin = Hoge()
    plugin.show()

    assert plugin.isVisible() is True # プラグインが表示されているかチェック

    plugin.closeButton.click()
    assert plugin.isVisible() is False # キャンセルボタンがクリックされたことで、非表示になったかチェック
```

### テストサンプル
`python
import pytest
from qgis.testing import start_app
from hogehoge_plugin import Hoge
from qgis.core import QgsProject

@pytest.fixture(scope="class") # テストクラスごとに一度だけ実行
def qgis_app():
    start_app() # QGISアプリケーションを起動

@pytest.fixture(scope="class") # テストクラスごとに一度だけ実行
def plugin(qgis_app):
    plugin = Hoge() # ここでプラグインが読み込まれる
    yield plugin # ここでテストが実行される
    del plugin # ここでテストが終了

def test_plugin(qgis_countries_layer):
    project = QgsProject.instance()
    project.addMapLayer(qgis_countries_layer)
    assert project.count() == 1
    assert project.mapLayersByName(qgis_countries_layer.name()) == [qgis_countries_layer]
`

### 機能をテストする
`python
import pytest
from qgis.testing import start_app
from hogehoge_plugin import Hoge
from qgis.core import QgsProject

@pytest.fixture(scope="class") # テストクラスごとに一度だけ実行
def qgis_app():
    start_app() # QGISアプリケーションを起動

@pytest.fixture(scope="class") # テストクラスごとに一度だけ実行
def plugin(qgis_app):
    plugin = Hoge() # ここでプラグインが読み込まれる
    yield plugin # ここでテストが実行される
    del plugin # ここでテストが終了

def test_hogehoge(plugin):
    project = QgsProject.instance() # プロジェクトを取得

    assert ○○
`


# qgis.testingでテストする

`python
from qgis.testing import unittest, start_app

class TestDockWidget(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        start_app()
        cls.plugin = DockWidget()
        cls.iface = cls.plugin.iface

    @classmethod
    def tearDownClass(cls):
        del cls.plugin
`
- `setUpClass`メソッドは、テストケースが実行される前に一度だけ呼び出されます。
- このメソッドでは、`start_app`関数を呼び出してQGISアプリケーションを開始し、`DockWidget`のインスタンスを作成しています。
- また、作成したプラグインの`iface`属性をテストクラスの`iface`属性に設定しています。これにより、テストメソッドからプラグインとそのインターフェースに簡単にアクセスできるようになります。
- `tearDownClass`メソッドは、テストケースが実行された後に一度だけ呼び出されます。このメソッドでは、`plugin`属性を削除して、テストで使用したリソースを解放しています。これにより、テストが終了した後もシステムのパフォーマンスに影響を与えないようにしています。
- このように、`setUpClass`と`tearDownClass`メソッドは、テストの前後で一度だけ実行する必要があるセットアップとクリーンアップのロジックを記述するために使用されます。これらのメソッドを使用することで、テストコードの冗長性を減らし、テストの可読性と保守性を向上させることができます。