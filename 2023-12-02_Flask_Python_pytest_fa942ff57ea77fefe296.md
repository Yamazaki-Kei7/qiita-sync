<!--
title:   テスト駆動開発でブログアプリのバックエンドを実装してみる
tags:    Flask,Python,pytest
id:      fa942ff57ea77fefe296
private: false
-->
:::note info
これは [MIERUNE AdventCalendar 2023](https://qiita.com/advent-calendar/2023/mierune) 5日目の記事です!
昨日は@northprintさんによる [deck.glをTypeScriptで組むときのポイント](https://qiita.com/northprint/items/7f1a5f535676c7205107) でした。
:::

# はじめに
最近、テスト駆動開発（TDD: Test-Driven Development）に興味を持ってきました。
私はpythonを使っているので、pythonでテスト駆動開発の一覧の流れを体験してみたいと思います。

テスト駆動開発をやる「お題」としては、[Modern Test-Driven Development in Python](https://testdriven.io/blog/modern-tdd/)に沿って基本的には進めることとし、プラスαとしてVisual Studio Code（VSCode）の拡張機能なども有効活用してみたいと思います。


## テスト駆動開発とはなんぞや
テスト駆動開発は、ソフトウェア開発プロセスの一つで、まず「テストケース」を作成し、そのテストがパスするようにコードを書いていく手法です。
テストを書くことで、開発者は機能の要件を明確に理解し、より効率的なコーディングが可能になる・・・と言われています。

テスト駆動開発の進め方・リズムについては、書籍「[テスト駆動開発](https://amzn.asia/d/hRP0A47)」にて、以下のように要約されています。

> 1. まずはテストを1つ書く
> 2. すべてのテストを走らせ、新しいテストの失敗を確認する
> 3. 小さな変更を行う
> 4. すべてのテストを走らせ、すべて成功することを確認する
> 5. リファクタリングを行なって重複を除去する


## テストで重要なこと
テスト自体を書く時は、以下3つの項目を意識して書くと良いそうです。
① テストは短く要点を絞って書く
② それぞれの動作を1回だけテストする
③ テストは独立している

また、「① テストは短く要点を絞って書く」時は、**GIVEN-WHEN-THEN**のテストケースを書くと、良いテストを書くことができます。

> 1. GIVEN: テストの前提条件を記述する
> 2. WHEN: テスト対象の操作を記述する
> 3. THEN: テスト結果の期待値を記述する


# 基本的なテストの実践
最初に基本的なテストのやり方ついて試してみます。

まずは、Visual Studio Code（VSCode）を起動します。
起動したら、任意の場所に新しいフォルダを作成し、そこに移動します。
`terminal
$ mkdir testing_project
$ cd testing_project
`

移動したらpoetryを使ってプロジェクトを初期化し、pytestをインストールします。
`terminal
$ poetry init
$ poetry add pytest
`

次に、テストを書くためのフォルダを作成します。
`
── sum
│   ├── __init__.py
│   └── another_sum.py
└── tests
    ├── __init__.py
    └── test_sum
        ├── __init__.py
        └── test_another_sum.py
`

テストを書く前に、テスト対象の関数を作成します。
another_sum.pyに以下のコードを追加しておきます。
`python: another_sum.py
def another_sum(a, b):
    return a + b
`

テスト対象の関数は、足し算をする関数です。これをテストするためのコードを書いてみます。
test_another_sum.pyに以下のコードを追加します。
`python:test_another_sum.py
def test_anoter_sum():
    assert another_sum(2, 3) == 5
`

次に、空のconftest.pyファイルを「tests」フォルダー内に追加します。このファイルは、 pytestフィクスチャの保存に使用していきますが、一旦パスしても良いです。

最後に、VSCodeの「拡張機能」から「Python Test Explorer for Visual Studio Code」を検索して、インストールします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/7a17b3da-c58e-0b78-d385-9477200fb611.png)

インストールできたら、「コマンドパレット」に`test`を入力し、「Python:テストを構成する」をクリックします。

コマンドパレットは、Windows／Linuxでは［Ctrl］＋［Shift］＋［P］キー、macOSでは［Command］＋［Shift］＋［P］キーを押すとウィンドウ上部に表示されます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/78ef4a41-0276-bc8a-cde8-605d6348ae83.png)

「Python:テストを構成する」をクリックすると、テストのフレームワークを選択する必要があるので、pytestを選択します。

テスト対象のフォルダを聞かれるので、今回はフォルダ「testing_project」を選択します。

ここまででプロジェクト構造は以下のようになります。

```
.
├── poetry.lock
├── pyproject.toml
├── sum
│   ├── __init__.py
│   └── another_sum.py
└── tests
    ├── __init__.py
    ├── conftest.py
    └── test_sum
        ├── __init__.py
        └── test_another_sum.py
```

早速テストを実行してみます。
VSCodeの左側のパネルからフラスコマークのアイコン「テスト」をクリックします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/549fb08a-ee0b-3e5b-8e5c-f64152ca7b36.png)

正しくテスト対象のファイルと関数が認識されていると、「テストエクスプローラー」に認識されたテストが表示されます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/e1b60545-d6f0-03eb-9d68-89d1ff498941.png)

「テストの実行」ボタンをクリックし、テストを実行します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/346b2e25-7899-12fc-23d7-09122c739369.png)

テスト結果が表示されました。
緑色のチェックマークが付いていれば、テストに通っています。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/4fb1fe0d-f88b-4e05-2da3-f03a94688bc9.png)

テストを失敗させてみたいので、
test_another_sum.pyのコードを修正してみます。
`diff_python:test_another_sum.py
def test_anoter_sum():
-    assert another_sum(2, 3) == 5
+    assert another_sum(2, 3) == 4
`

再度「テストの実行」ボタンをクリックし、テストを実行すると失敗したことが確認できました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/2c084f8d-939e-07f3-8b4c-aa4e7c6baba6.png)

テストに失敗した理由は、該当するテストコードで確認できます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/85473df2-bd82-2e84-bd2a-1f4968b98f6f.png)

テストは以上のように実施することができます。

次からは、実際のアプリケーションの作成を通して、テスト駆動開発を実践していきたいと思います。

# 実際のアプリケーションでテスト駆動開発をしてみる
pytestによるテスト駆動開発で、簡単なブログアプリのバックエンドを構築してみます。

使用するフレームワーク等は以下の通りです。
- Flask
- SQLite

作成するアプリの要求事項は以下としてみます。
- 記事が作成できる
- 記事を取得できる
- 記事を一覧表で表示できる

## 準備
それでは、新しくプロジェクトを作成します。

```
$ mkdir blog_app
$ cd blog_app
```

poetryで仮想環境を作成し、pytestとpydanticをインストールします。
`terminal
$ poetry init
$ poetry add pytest "pydantic[email]"
`

次に、フォルダ`blog_app`内に以下のファイルとフォルダを作成します。
`
.
├── blog
│   ├── __init__.py
│   ├── app.py
│   └── models.py
├── poetry.lock
├── pyproject.toml
└── tests
    ├── __init__.py
    ├── conftest.py
    └── pytest.ini
`

models.pyに以下のコードを追加して、`pydantic`で新しく`Article`モデルを定義します。`pydantic`に関する説明は割愛しますが、`pydantic`を使用することでデータ型の検証を簡単に行うことができます。

```python:models.py
import os
import sqlite3
import uuid
from typing import List

from pydantic import BaseModel, EmailStr, Field


class NotFound(Exception):
    pass


class Article(BaseModel):
    id: str = Field(default_factory=lambda: str(uuid.uuid4()))
    author: EmailStr
    title: str
    content: str

    @classmethod
    def get_by_id(cls, article_id: str) -> "Article":
        con = sqlite3.connect(os.getenv("DATABASE_NAME", "database.db"))
        # 列名をキーとして値にアクセスできるようにする
        con.row_factory = sqlite3.Row

        cur = con.cursor()
        cur.execute("SELECT * FROM articles WHERE id=?", (article_id,))

        record = cur.fetchone()

        if record is None:
            raise NotFound

        article = cls(**record)
        con.close()

        return article

    @classmethod
    def get_by_title(cls, title: str) -> "Article":
        con = sqlite3.connect(os.getenvi("DATABASE_NAME", "database.db"))
        con.row_factory = sqlite3.Row

        cur = con.cursor()
        cur.execute("SELECT * FROM articles WHERE title=?", (title,))

        record = cur.fetchone()

        if record is None:
            raise NotFound

        article = cls(**record)
        con.close()

        return article

    @classmethod
    def list(cls) -> List["Article"]:
        con = sqlite3.connect(os.getenv("DATABASE_NAME", "database.db"))
        con.row_factory = sqlite3.Row

        cur = con.cursor()
        cur.execute("SELECT * FROM articles")

        records = cur.fetchall()
        articles = [cls(**record) for record in records]
        con.close()

        return articles

    def save(self) -> "Article":
        with sqlite3.connect(os.getenv("DATABASE_NAME", "database.db")) as con:
            cur = con.cursor()
            cur.execute(
                "INSERT INTO articles" \
                "(id,author,title,content)"\
                "VALUES(?, ?, ?, ?)",
                (self.id, self.author, self.title, self.content),
            )
            con.commit()

        return self

    @classmethod
    def create_table(cls, database_name="database.db") -> None:
        conn = sqlite3.connect(database_name)

        conn.execute(
            "CREATE TABLE IF NOT EXISTS articles"\
            "(id TEXT, author TEXT, title, TEXT, content TEXT)"
        )
        conn.close()
```

## テスト駆動開発の実践
### 要求事項「記事が作成できる」のテスト
「tests」フォルダに「test_article」フォルダを作成します。
次に、 test_commands.pyというファイルを「test_article」フォルダに追加します。
`
.
├── blog
│   ├── __init__.py
│   ├── app.py
│   └── models.py
├── poetry.lock
├── pyproject.toml
└── tests
    ├── __init__.py
    ├── conftest.py
    ├── pytest.ini
    └── test_article
        ├── __init__.py
        └── test_commands.py
`
次に、test_commands.pyに以下のテストコードを追加します。

```python:test_commands.py
import pytest

from blog.models import Article
from blog.commands import CreateArticleCommand, AlreadyExists

def test_create_article():
    """
    GIVEN 「author, title, content」のプロパティを持つ
            CreateArticleCommandが与えられ
    WHEN executeメソッドが呼び出されたとき
    THEN データベースには同じ属性を持つ新しい記事が存在しなければならない
    """
    cmd = CreateArticleCommand(
        author="test@test.com",
        title="新しい記事です",
        content="すばらしい記事です",
    )

    # データベースに新しい記事が作成される
    article = cmd.execute()
    # データベースから同じ属性を持つ記事の取得を試みる
    db_article = Article.get_by_id(article.id)

    assert db_article.id == article.id
    assert db_article.author == article.author
    assert db_article.title == article.title
    assert db_article.content == article.content

def test_create_article_already_exists():
    """
    GIVEN 「author, title, content」のプロパティを持つ
            CreateArticleCommandが与えられ
    WHEN executeメソッドが呼び出されたとき
    THEN データベースには同じ属性を持つ新しい記事が存在してはいけない
    """

    # データベースに新しい記事を作成する
    Article(
        author="test@test.com",
        title="新しい記事です",
        content="すばらしい記事です",
    ).save()
    # 同じ属性を持つ記事を作成しようとする
    cmd = CreateArticleCommand(
        author="test@test.com",
        title="新しい記事です",
        content="すばらしい記事です",
    )

    # すでに記事が存在するため、例外が発生するかを確認する
    with pytest.raises(AlreadExists):
        cmd.execute()
```

作成したテストにより、以下のユースケースをカバーすることができます。
- 有効なデータに対して記事を作成する必要がある
- 記事のタイトルは一意である必要がある

ただ、今の段階だと`blog.commands`で呼び出す関数が作成できていなので、commands.pyファイルを「blog」フォルダーに追加します。
`python:commands.py
from pydantic import BaseModel, EmailStr

from blog.models import Article, NotFound


class AlreadyExists(Exception):
    pass


class CreateArticleCommand(BaseModel):
    author: EmailStr
    title: str
    content: str

    def execute(self) -> Article:
        try:
            Article.get_by_title(self.title)
            raise AlreadyExists
        except NotFound:
            pass

        article = Article(
            author=self.author,
            title=self.title,
            content=self.content
        ).save()

        return article
`

ここまでできたら、テストを実行して、テストが失敗することを確認します。
コマンドパレットから「Python:テストを構成する」でpytestを選択し、フォルダ「blog_app」をクリックします。

正しくテスト対象のファイルと関数が認識されたらテストを実行します。
実行すると失敗したことが確認できます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/687ad6ef-3136-b2e1-469d-12ec61834a04.png)

`test_create_article`のエラー内容を確認すると、データベースが作成されていないため、エラーが発生していることがわかります。
`terminal
>       cur.execute("SELECT * FROM articles WHERE title=?", (title,))
E       sqlite3.OperationalError: no such table: articles
`

テストを実施するために、以下について対応していきます。
- テスト開始前にデータベースを新規作成する
- テスト実施後にデータベースをクリアする

### Test Fixtures
pytest fixturesを使用すると、各テストの後にデータベースをクリアし、各テストの前に新しいデータベースを作成できます。

fixturesはデコレータ `@pytest.fixture`で装飾された関数です。

fixturesは通常、conftest.pyに記述しますが、実際のテストファイルにも追加できます。これらの関数はデフォルトで各テストの前に実行されます。

pytest fixturesでは、以下のようなオプションが利用できます。
- pytest fixturesを使用するとテスト内の戻り値を使用することができる
- データベースの作成やモジュールのモックなどの副作用を実行できる
- `yield`を使用することでテストの前後にfixturesを一部実行できる

それでは、次のfixturesをconftest.pyに追加します。
これにより、各テストの前に新しいデータベースが作成され、テスト後に削除されます。
`python:conftest.py
import os
import tempfile

import pytest

from blog.models import Article

# 各テストの前後にデフォルトで自動的に使用されるようにautouseをTrueにする
@pytest.fixture(autouse=True)
def database():
    _, file_name = tempfile.mkstemp()
    os.environ["DATABASE_NAME"] = file_name
    Article.create_table(database_name=file_name)
    yield # ここでテストが実行される
    os.unlink(file_name)
`

:::note warn
今回は全てのテストケースでデータベースを使用するため、`autouse=True`のフラグを使用しています。

一方、データベースへのアクセスが必要ない場合は、テストマーカーで自動化を無効にすることができます。
自動化を無効化するテストマーカーは`@pytest.mark.noautofixt`を使用します。
:::


それではテストを再度実行します。
無事にテストに通りました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/6a37a054-7cb8-a991-7748-d7debac434cf.png)


### 要求事項「記事を一覧表で表示できる」のテスト
今度は「記事を一覧表で表示できる」機能をテストします。

コマンドの代わりにクエリを使用するので、test_queries.pyという新しいファイルを「test_article」フォルダーに追加します。
`python:test_queries.py
from blog.models import Article
from blog.queries import ListArticlesQuery


def test_list_articles():
    """
    GIVEN データベースに2つの記事が保存されている
    WHEN execute methodが呼ばれた時
    THEN 2つの記事のリストが返される
    """
    Article(
        author="test@test.com",
        title="新しい記事です",
        content="すばらしい記事です",
    ).save()
    Article(
        author="test@test.com",
        title="他の記事です",
        content="さらにすばらしい記事です",
    ).save()

    query = ListArticlesQuery()

    assert len(query.execute()) == 2
`

それではテストを実行してみます・・・が、クエリの実装をしていないため、テストに失敗します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/7e7ff2a4-a04e-4200-fd3b-082df15be99f.png)

念の為、ターミナルからもテストを実行し、確認してみます。
`terminal
$ poetry run python -m pytest tests
____________________ ERROR collecting test_article/test_queries.py _____________________
ImportError while importing test module '/Users/yamakei/Documents/Study/Pythonによる最新のテスト駆動開発/blog_app/tests/test_article/test_queries.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
../../../../anaconda3/lib/python3.10/importlib/__init__.py:126: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
tests/test_article/test_queries.py:2: in <module>
    from blog.queries import ListArticlesQuery
E   ModuleNotFoundError: No module named 'blog.queries'
=============================== short test summary info ================================
ERROR tests/test_article/test_queries.py
!!!!!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!!!!!
=================================== 1 error in 0.04s ===================================
`

テストを通すために作業をしていきます。

queries.pyファイルを「blog」フォルダに追加します。

```python:queries.py
from typing import List
from pydantic import BaseModel

from blog.models import Article


class ListArticlesQuery(BaseModel):

    def execute(self) -> List[Article]:
        articles = Article.list()

        return articles
```

それでは再度テストを実行してみます。
結果を見ると、全てのテストが通ったことを確認できます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/1f695c4a-ce93-d39f-2722-d14f03144117.png)


### 要求事項「記事を取得できる」のテスト
次に「記事を取得できる」機能をテストします。

IDによって1つの記事を取得できるかテストしていきます。
新しいテストをtest_queries.pyに追加します。

```diff_python:test_queries.py
from blog.models import Article
- from blog.queries import ListArticlesQuery
+ from blog.queries import ListArticlesQuery, GetArticleByIDQuery

def test_list_articles():
    """
    GIVEN データベースに2つの記事が保存されている
    WHEN execute methodが呼ばれた時
    THEN 2つの記事のリストが返される
    """
    Article(
        author="test@test.com",
        title="新しい記事です",
        content="すばらしい記事です",
    ).save()
    Article(
        author="test@test.com",
        title="他の記事です",
        content="さらにすばらしい記事です",
    ).save()

    query = ListArticlesQuery()

    assert len(query.execute()) == 2

+ def test_get_article_by_id():
+     """
+     GIVEN データベースに記事が保存されている
+     WHEN GetArticleByIDQueryのexecute methodがID付きで呼ばれた時
+     THEN 同じIDの記事が返される
+     """
+     article = Article(
+     author="test@test.com",
+     title="新しい記事です",
+     content="すばらしい記事です",
+     ).save()
+
+     query = GetArticleByIDQuery(
+         id=article.id
+         )
+
+     assert query.execute().id == article.id
```

テストを実行し、失敗することを確認します。
`GetArticleByIDQuery`がまだ実装できていないのでテストには通りません。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/5b5ac084-9a98-ae72-f1dd-7a041769f432.png)


テストを通すため、querys.pyに以下を追記します。

```diff_python:querys.py
from typing import List
from pydantic import BaseModel

from blog.models import Article


class ListArticlesQuery(BaseModel):

    def execute(self) -> List[Article]:
        articles = Article.list()

        return articles


+ class GetArticleByIDQuery(BaseModel):
+     id: str
+
+     def execute(self) -> Article:
+         article = Article.get_by_id(self.id)
+
+         return article
```

再度テストを実行してみると、テストに通りました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/2589917f-2bf6-7cc6-01e8-ce0f28b25b99.png)

### ここまでの振り返り
ここまで要求事項を満たすためのテストコードを作成し、それが通るように開発してきたので、以下の要求事項を満たしていることが保証できています。
- 記事が作成できる
- 記事を取得できる
- 記事を一覧表で表示できる

次からは、Flask RESTful APIを介して、上記機能をWebで公開できるようにしていきます。


### Flaskを使用してAPIを公開する
Flaskを使用してAPIを公開するための3つのエンドポイントは以下になります。

1. `/create-article/` - 新しく記事を作成する
1. `/article-list/` - 全ての記事を取得する
1. `/article/<article_id>/` - 記事を1つだけ取得する


まず「test_article」フォルダ内に「schemas」フォルダを作成し、そこに2つのJSONスキーマ、Article.jsonとArticleList.jsonを追加します。

```json:Article.json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "Article",
    "type": "object",
    "properties": {
        "id": {
        "type": "string"
        },
        "author": {
        "type": "string"
        },
        "title": {
        "type": "string"
        },
        "content": {
        "type": "string"
        }
    },
    "required": ["id", "author", "title", "content"]
    }
```

```json:ArticleList.json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "ArticleList",
    "type": "array",
    "items": {"$ref":  "file:Article.json"}
    }
```

JSONスキーマは、APIエンドポイントからのレスポンスを定義するために使用されます。

作業を続ける前に、定義されたスキーマに対してJSONペイロードを検証するために使用されるjsonschemaというPythonライブラリをインストールします。また、Flaskもインストールします。

```terminal
$ poetry add jsonschema Flask
```

それでは、APIのテストを書いてみます。

「test_article」フォルダに test_app.py という新しいファイルを追加します。

```python:test_app.py
import json
import pathlib
import pytest

from blog.app import app
from jsonschema import validate, RefResolver

from blog.models import Article

@pytest.fixture
def client() -> object:
    app.config["TESTING"] = True

    with app.test_client() as client:
        yield client

def validate_payload(payload, schema_name):
    """
    JSON Schemaを使ってペイロードを検証する
    """
    schemas_dir = str(
        f"{pathlib.Path(__file__).parent.absolute()}/schemas"
    )
    schema = json.loads(
        pathlib.Path(
            f"{schemas_dir}/{schema_name}"
        ).read_text()
    )
    validate(
        payload,
        schema,
        resolver=RefResolver(
            "file://" + str(
                pathlib.Path(f"{schemas_dir}/{schema_name}").absolute()
            ),
            schema  # スキーマ内のファイルを正しく解決するために使われる
        )
    )

def test_create_article(client):
    """
    GIVEN 新しい記事のデータ
    WHEN エンドポイント /create-article/ が呼ばれた時
    THEN スキーマにマッチした記事をjson形式で返す
    """
    data = {
        'author': "test@test.com",
        'title': "新しい記事です",
        'content': "すばらしい記事です",
    }
    response = client.post(
        "/create-article/",
        data = json.dumps(
            data
        ),
        content_type="application/json",
    )

    validate_payload(
        response.json,
        "Article.json"
    )

def test_get_article(client):
    """
    GIVEN データベースに格納されている記事のID
    WHEN エンドポイント /article/<id-of-article>/ が呼ばれた時
    THEN スキーマにマッチした記事をjson形式で返す
    """
    article = Article(
        author = "test@test.com",
        title = "新しい記事です",
        content = "すばらしい記事です",
    ).save()
    response = client.get(
        f"/article/{article.id}/",
        content_type="application/json",
    )

    validate_payload(
        response.json,
        "Article.json"
    )

def test_list_articles(client):
    """
    GIVEN データベースに保存されている複数の記事
    WHEN エンドポイント /article-list/ が呼ばれた時
    THEN スキーマにマッチした記事のリストをjson形式で返す
    """
    article = Article(
        author = "test@test.com",
        title = "新しい記事です",
        content = "すばらしい記事です",
    ).save()
    response = client.get(
        "/article-list/",
        content_type="application/json",
    )

    validate_payload(
        response.json,
        "ArticleList.json"
    )
```

上記コードでは以下のことを行っています。
1. テストで使用できるように、Flaskのテストクライアントをfixtureとして定義しました。
1. 次に、ペイロードを検証する機能を追加しました。これには2つのパラメータが必要です。
    1. `payload` - APIからのJSONレスポンス
    1. `schema_name` - 「schemas」ディレクトリ内のスキーマファイルの名前
1. 最後に、エンドポイントごとに 1 つずつ、合計 3 つのテストがあります。各テスト内では API の呼び出しと、返されたペイロードの検証が行われます。


それではテストを実行し、失敗することを確認します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/a622437f-5366-2ea1-da31-257a99e75204.png)

```terminal
Traceback:
/Users/yamakei/anaconda3/lib/python3.10/importlib/__init__.py:126: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
test_article/test_app.py:5: in <module>
    from blog.app import app
E   ImportError: cannot import name 'app' from 'blog.app'
```

テストを通すため、app.py を以下のように更新します。
`python:app.py
from flask import Flask, jsonify, request

from blog.commands import CreateArticleCommand
from blog.queries import GetArticleByIDQuery, ListArticlesQuery

app = Flask(__name__)

@app.route("/create-article/", methods=["POST"])
def create_article():
    """
    新しい記事を作成する
    """
    cmd = CreateArticleCommand(
        **request.json
    )
    return jsonify(cmd.execute().dict())

@app.route("/article/<article_id>/", methods=["GET"])
def get_article(article_id: str):
    """
    記事を取得する
    """
    query = GetArticleByIDQuery(
        id=article_id
    )
    return jsonify(query.execute().dict())

@app.route("/article-list/", methods=["GET"])
def list_articles():
    """
    記事のリストを取得する
    """
    query = ListArticlesQuery()
    records = [record.dcit() for record in query.execute()]
    return jsonify(records)

if __name__ == "__main__":
    app.run()
`

テストを再実行すると、テストが通りました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/7ed0143c-0241-1bed-a1e0-0c7ba4502bff.png)


ここまでテストを色々としてきましたが、実際に運用するとなると、クライアントがAPIを常に意図されたとおりに使用するとは限りません。

たとえば、記事を作成するリクエストがタイトルなしで行われたとき、`CreateArticleCommand`コマンドによって`ValidationError`が発生し、内部サーバーエラーとHTTPステータス500となります。

このような事態は回避したいので、エラーを処理して、ユーザーに適切に通知する必要があります。

このようなケースをカバーするテストを書いてきます。
test_app.pyに以下を追加します。pytestの`pytest.mark.parametrize`を使用することで、複数の入力を単一の渡す実装としています。

```python:test_app.py
@pytest.mark.parametrize(
    "data",
    [
        {
            'author': "testname",
            'title': "新しい記事です",
            'content': "すばらしい記事です",
        },
        {
            'author': "testname",
            'title': "新しい記事です",
        },
        {
            'author': "testname",
            "title": None,
            'content': "すばらしい記事です",
        }
    ]
)
def test_create_article_bad_request(client, data):
    """
    GIVEN 無効な値または欠落した属性のデータ
    WHEN エンドポイント /create-article/ が呼ばれた時
    THEN ステータス 400を返す
    """
    response = client.post(
        "/create-article/",
        data=json.dumps(
            data
        ),
        content_type="application/json",
    )

    assert response.status_code == 400
    assert response.json is not None
```

テストを実行し、失敗することを確認します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/0e2ae86d-23e4-b388-a459-f03a791e924d.png)

それでは、app.pyのFlaskアプリにエラーハンドラーを追加していきます。

```diff_python:app.py
from flask import Flask, jsonify, request
+ from pydantic import ValidationError

from blog.commands import CreateArticleCommand
from blog.queries import GetArticleByIDQuery, ListArticlesQuery

app = Flask(__name__)

+ @app.errorhandler(ValidationError)
+ def handle_validation_exception(error):
+     """
+     バリデーションエラーを処理する
+     """
+     response = jsonify(error.errors())
+     response.status_code = 400
+     return response

@app.route("/create-article/", methods=["POST"])
def create_article():
    """
    新しい記事を作成する
    """
    cmd = CreateArticleCommand(
        **request.json
    )
    return jsonify(cmd.execute().dict())

@app.route("/article/<article_id>/", methods=["GET"])
def get_article(article_id: str):
    """
    記事を取得する
    """
    query = GetArticleByIDQuery(
        id=article_id
    )
    return jsonify(query.execute().dict())

@app.route("/article-list/", methods=["GET"])
def list_articles():
    """
    記事のリストを取得する
    """
    query = ListArticlesQuery()
    records = [record.dict() for record in query.execute()]
    return jsonify(records)

if __name__ == "__main__":
    app.run()
```

テストを再実行すると、テストが通りました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/125a0936-e044-45ca-7a56-8da7d6cc8d18.png)

### コードカバレッジを確認する
アプリケーションのテストが完了したので、コード カバレッジを確認します。
pytest-covをpoetryでインストールします。
`terminal
$ poetry add pytest-cov
`

インストールできたら、ターミナルに以下を入力してコードカバレッジを確認します。
`terminal
$ poetry run python -m pytest tests --cov=blog

~中略~

---------- coverage: platform darwin, python 3.10.9-final-0 ----------
Name               Stmts   Miss  Cover
--------------------------------------
blog/__init__.py       0      0   100%
blog/app.py           25      1    96%
blog/commands.py      16      0   100%
blog/models.py        57      1    98%
blog/queries.py       12      0   100%
--------------------------------------
TOTAL                110      2    98%
`


### エンドツーエンドのテスト
次に、エンドツーエンド (e2e) テストを作成していきます。
すでにシンプルなAPIがあるため、次のシナリオをカバーする単一のe2eテストを作成できます。

1. 新しい記事を作成する
1. 記事を一覧表示する
1. リストから最初の記事を取得する


まずは、poetryでrequestsをインストールします。
`terminal
$ poetry add requests
`

新しいテストをtest_app.pyに追加します。
`diff_python:test_app.py
import json
import pathlib
import pytest
+ import requests

from blog.app import app
from jsonschema import validate, RefResolver

from blog.models import Article

@pytest.fixture
def client() -> object:
    app.config["TESTING"] = True

    with app.test_client() as client:
        yield client

def validate_payload(payload, schema_name):
    """
    JSON Schemaを使ってペイロードを検証する
    """
    schemas_dir = str(
        f"{pathlib.Path(__file__).parent.absolute()}/schemas"
    )
    schema = json.loads(
        pathlib.Path(
            f"{schemas_dir}/{schema_name}"
        ).read_text()
    )
    validate(
        payload,
        schema,
        resolver=RefResolver(
            "file://" + str(
                pathlib.Path(f"{schemas_dir}/{schema_name}").absolute()
            ),
            schema  # スキーマ内のファイルを正しく解決するために使われる
        )
    )

def test_create_article(client):
    """
    GIVEN 新しい記事のデータ
    WHEN エンドポイント /create-article/ が呼ばれた時
    THEN スキーマにマッチした記事をjson形式で返す
    """
    data = {
        'author': "test@test.com",
        'title': "新しい記事です",
        'content': "すばらしい記事です",
    }
    response = client.post(
        "/create-article/",
        data = json.dumps(
            data
        ),
        content_type="application/json",
    )

    validate_payload(
        response.json,
        "Article.json"
    )

def test_get_article(client):
    """
    GIVEN データベースに格納されている記事のID
    WHEN エンドポイント /article/<id-of-article>/ が呼ばれた時
    THEN スキーマにマッチした記事をjson形式で返す
    """
    article = Article(
        author = "test@test.com",
        title = "新しい記事です",
        content = "すばらしい記事です",
    ).save()
    response = client.get(
        f"/article/{article.id}/",
        content_type="application/json",
    )

    validate_payload(
        response.json,
        "Article.json"
    )

def test_list_articles(client):
    """
    GIVEN データベースに保存されている複数の記事
    WHEN エンドポイント /article-list/ が呼ばれた時
    THEN スキーマにマッチした記事のリストをjson形式で返す
    """
    Article(
        author = "test@test.com",
        title = "新しい記事です",
        content = "すばらしい記事です",
    ).save()
    response = client.get(
        "/article-list/",
        content_type="application/json",
    )

    validate_payload(
        response.json,
        "ArticleList.json"
    )

@pytest.mark.parametrize(
    "data",
    [
        {
            'author': "testname",
            'title': "新しい記事です",
            'content': "すばらしい記事です",
        },
        {
            'author': "testname",
            'title': "新しい記事です",
        },
        {
            'author': "testname",
            "title": None,
            'content': "すばらしい記事です",
        }
    ]
)
def test_create_article_bad_request(client, data):
    """
    GIVEN 無効な値または欠落した属性のデータ
    WHEN エンドポイント /create-article/ が呼ばれた時
    THEN ステータス 400を返す
    """
    response = client.post(
        "/create-article/",
        data=json.dumps(
            data
        ),
        content_type="application/json",
    )

    assert response.status_code == 400
    assert response.json is not None


+ @pytest.mark.e2e
+ def test_create_list_get(client):
+     requests.post(
+         "http://localhost:5000/create-article/",
+         json={
+             'author': "test@test.com",
+             'title': "新しい記事です",
+             'content': "すばらしい記事です",
+         }
+     )
+     response = requests.get(
+         "http://localhost:5000/article-list/",
+     )
+
+     articles = response.json()
+
+     response = requests.get(
+         f"http://localhost:5000/article/{articles[0]['id']}/",
+     )
+
+     assert response.status_code == 200
`

このテストを実行する前に以下の2つに準備をする必要があります。
1. pytest.iniに以下のコードを追加して、e2eというマーカーをpytestに登録する
1. データベースを作成しておく

まずは、pytest.iniに以下のコードを追加して、e2eというマーカーをpytestに登録します。
`init:pytest.init
[pytest]
markers =
    e2e: marks tests as e2e (deselect with '-m "not e2e"')
`

pytestのマーカーは、一部のテストを実行から除外したり、場所に関係なく選択したテストを実行するのに使用します。

e2eだけをテストする場合は、コマンドラインから以下で実行します。
`terminal
$ poetry run python -m pytest tests -m 'e2e'
`

e2e以外をテストする場合は、コマンドラインから以下で実行します。
`terminal
$ poetry run python -m pytest tests -m 'not e2e'
`

e2eテストは起動済みサーバーを使用するので、アプリを起動する必要があります。
新しいターミナルウィンドウでプロジェクトに移動し、アプリを実行します。
`terminal
$ poetry shell
$ FLASK_APP=blog/app.py python -m flask run
`

それではe2eのマークをつけたテストを実行してみます。
`terminal
$ poetry run python -m pytest tests -m 'e2e'
`<Response [500]`が返ってきて、テストに失敗します。
データベースを作成していないのでテストに失敗してしまいました。
テストを通すため、データベースを作成していきます。

init_db.pyファイルを「blog」フォルダーに追加します。
`python:init_db.py
if __name__ == "__main__":
    from blog.models import Article
    Article.create_table()
`

作成したinit_db.pyファイルを実行後、サーバーを再起動します。
`terminal
$ python blog/init_db.py
$ FLASK_APP=blog/app.py python -m flask run
`

テストを再実行すると、テストが通りました。
`
$ poetry run python -m pytest tests -m 'e2e'

~中略~

============== 1 passed, 10 deselected, 2 warnings in 0.13s ===============
`
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/584199/238b3565-55f6-56bb-2d1c-abb8fa2ee673.png)

# まとめ
簡単なブログアプリについて、バックエンド側の開発をテスト駆動開発で進めてみました。
慣れないとテスト駆動で開発を進めていくのは大変そうですが、慣れてさえしまえば、素早く高品質な開発が実践できそうです。

# 参考
- [Modern Test-Driven Development in Python](https://testdriven.io/blog/modern-tdd/)
- [VSCodeとpytestでPythonコードをテスト&デバッグする](https://gri.jp/media/entry/358)

:::note info
明日は@Yfuruchinさんによる記事です！お楽しみに！！
:::