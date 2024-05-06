# Django + Svelte + Tailwindcss な開発環境の構築方法

Django で Web アプリを開発する際に Svelte と Tailwind CSS を組み合わせて使用することは、現代の Web 開発の流れに沿っており、フロントエンドの開発体験をより良いものにします。
以下に、このような開発環境を整えるための基本的な手順を示します。

## 1. Django プロジェクトのセットアップ

まずは Django プロジェクトをセットアップします。プロジェクトがまだない場合は、以下のコマンドで新しいプロジェクトを作成します。

```bash
django-admin startproject myproject
cd myproject
```

## 2. Svelte プロジェクトのセットアップ

Django プロジェクト内に、フロントエンド用のディレクトリを作成し、その中で Svelte プロジェクトをセットアップします。以下のコマンドは、`frontend`という名前のディレクトリで Svelte プロジェクトを初期化する方法を示しています。

```bash
mkdir frontend
cd frontend
npx degit sveltejs/template svelte-app
cd svelte-app
npm install
```

## 3. Tailwind CSS の導入

Svelte プロジェクト内で Tailwind CSS を使用するためには、Tailwind とその依存関係をインストールし、設定ファイルを初期化します。

```bash
npm install -D tailwindcss@latest postcss@latest autoprefixer@latest
npx tailwindcss init
```

その後、`tailwind.config.js`ファイルと`postcss.config.js`ファイルを編集して、Tailwind をプロジェクトに統合します。また、Svelte コンポーネント内で Tailwind のクラスを使えるようにするために、`src`ディレクトリ内のグローバル CSS ファイル（通常は`global.css`）に Tailwind のディレクティブを追加します。

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## 4. Django と Svelte の統合

Django テンプレートで Svelte アプリケーションを読み込むためには、Svelte ビルドプロセスをカスタマイズして、ビルドされた静的ファイルが Django の静的ファイルシステムで利用可能になるようにします。これを実現する一つの方法は、Svelte のビルド出力ディレクトリを Django の`static`ディレクトリに設定することです。

`rollup.config.js`（またはあなたが使用しているバンドラーの設定ファイル）を編集して、出力ディレクトリを Django の静的ファイルディレクトリに合わせます。例えば：

```js
output: {
  sourcemap: true,
  format: 'iife',
  name: 'app',
  file: '../myproject/static/js/bundle.js'
}
```

この設定により、Svelte でビルドしたファイルが Django の`static`ディレクトリ内に配置され、Django のテンプレートから参照できるようになります。

## 5. 開発サーバーの起動

Django と Svelte の開発サーバーをそれぞれ起動します。Django の場合は以下のコマンドを使用します。

```bash
python manage.py runserver
```

Svelte の場合は、`frontend/svelte-app`ディレクトリ内で以下のコマンドを使用します。

```bash
npm run dev
```

このセットアップにより、Django をバックエンドとして、Svelte と Tailwind CSS を使ったフロントエンドの開発が可能になります。
フロントエンドの変更は Svelte の開発サーバーによってホットリロードされ、Django サーバーは API やテンプレートの提供を担当します。
