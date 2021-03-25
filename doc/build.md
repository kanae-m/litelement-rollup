# Build for production

LitElementコンポーネントを含むアプリをビルドする場合、Rollupやwebpackなどの一般的なJavaScriptビルドツールを使用できます。

標準のESモジュール形式で動作するように設計されているため、Rollupをお勧めします。

別のツールを使用してビルドする場合、またはLitElementを既存のビルドシステムに統合する場合は、ビルド要件を参照してください。

## Building with Rollup

プロジェクトをバンドルするRollupをセットアップする方法はたくさんありまが、ここでは2つ紹介します。

- 常にバックグラウンドで更新されるブラウザ(Chrome, firefoxなど)で実行されるモダンなビルド
- IE11のブラウザで実行される全般的なビルド

もし、常にバックグラウンドで更新されるようなブラウザしかサポートしないならば、モダンなビルドを用います。もし、様々なブラウザをサポートする必要がある場合は、互換性のあるビルドを用います。

このセクションの最後の部分では、2つのビルドを一緒に使用して、古いブラウザーもサポートしながら、高速で小型の最新ビルドを提供する方法について説明します。

ここでの構成例では、Shopデモアプリを例として使用しています。ここで説明されているすべての構成は、ショップリポジトリのブランチにあります。

https://github.com/Polymer/shop/tree/rollup-examples-v2

### Modern browser build

最新のブラウザビルドは、次のnpmパッケージを使用します。

|package|description|
|:--|:--|
|[`rollup`](https://www.npmjs.com/package/rollup)|rollup バンドラー|
|[`@rollup/plugin-node-resolve`](https://www.npmjs.com/package/@rollup/plugin-node-resolve)|ベアモジュール指定子を解決するため|
|[`rollup-plugin-terser`](https://www.npmjs.com/package/rollup-plugin-terser)|JavaScriptを縮小します。これは厳密には必須ではありませんが、本番環境にバンドルする場合は、JavaScriptを縮小することをお勧めします。|
|[`rollup-plugin-copy`](https://www.npmjs.com/package/rollup-plugin-copy)|静的アセットをビルドフォルダーにコピーします。|
|[`rollup-plugin-minify-html-literals`](https://www.npmjs.com/package/rollup-plugin-minify-html-literals)|オプションの最適化|

このビルドのロールアップ構成ファイルは次のようになります。

```js
import resolve from '@rollup/plugin-node-resolve';
import { terser } from 'rollup-plugin-terser';
import minifyHTML from 'rollup-plugin-minify-html-literals';
import copy from 'rollup-plugin-copy';

// Static assets will vary depending on the application
const copyConfig = {
  targets: [
    { src: 'node_modules/@webcomponents', dest: 'build-modern/node_modules' },
    { src: 'images', dest: 'build-modern' },
    { src: 'data', dest: 'build-modern' },
    { src: 'index.html', dest: 'build-modern' },
  ],
};

// The main JavaScript bundle for modern browsers that support
// JavaScript modules and other ES2015+ features.
const config = {
  input: 'src/components/shop-app.js',
  output: {
    dir: 'build-modern/src/components',
    format: 'es',
  },
  plugins: [
    minifyHTML(),
    copy(copyConfig),
    resolve(),
  ],
  preserveEntrySignatures: false,
};

if (process.env.NODE_ENV !== 'development') {
  config.plugins.push(terser());
}

export default config;
```

#### Even simpler starter

[rollup-starter-app](https://github.com/rollup/rollup-starter-app)リポジトリは、Rollupを使用してアプリを構築するための最低限のスタータープロジェクトです。リポジトリにはLitElementは含まれていませんが、LitElementアプリをバンドルするために必要なすべてのものが含まれています。このプロジェクトを独自のプロジェクトのモデルとして使用する場合、確認する最も重要なファイルは[`package.json`](https://github.com/rollup/rollup-starter-app/blob/master/package.json)と[`rollup.config.js`](https://github.com/rollup/rollup-starter-app/blob/master/rollup.config.js)です。

必要なプラグインに加えて、[rollup-starter-app](https://github.com/rollup/rollup-starter-app)にはCommonJSプラグイン、[`@rollup/plugin-commonjs`](https://www.npmjs.com/package/@rollup/plugin-commonjs)が含まれていることに注意してください。このプラグインはLitElementには必要ありませんが、CommonJSモジュールとしてのみ配布されるパッケージをインポートする場合に役立ちます。

### Universal build

IE11用だから今回は必要ありません。

## Build requirements

このセクションでは、LitElementを使用してアプリケーションを構築するための要件について説明します。独自のビルドセットアップを作成する場合は、このセクションを使用してください。

LitElementは、最新のJavaScript(ES 2017)で記述されたESモジュールのセットとしてパッケージ化されています。これらは、最新のブラウザー(Chrome、Safari、Firefox、Edgeなど)でネイティブにサポートされています。

LitElementを使用してアプリをビルドする場合、ビルドシステムは以下を処理する必要があります。

- ベア（またはノードスタイル）モジュール識別子の解決。 LitElementはベアモジュール指定子を使用します。

### Bare module specifiers

LitElementは、次のように、ベアモジュール指定子を使用してlit-htmlライブラリからモジュールをインポートします。

```js
import {html} from 'lit-html';
```

ブラウザは現在、URLまたは相対パスからのモジュールの読み込みのみをサポートしています。

ブラウザは現在、URLまたは相対パスからのモジュールの読み込みのみをサポートしており、たとえば、 npmパッケージであるため、ビルドシステムはそれらを処理する必要があります。指定子をブラウザーのESモジュールで機能するものに変換するか、出力として別のタイプのモジュールを生成します。

Webpackはベアモジュール指定子を自動的に処理します。 Rollupの場合、プラグイン([`@rollup/plugin-node-resolve`](https://github.com/rollup/plugins/tree/master/packages/node-resolve))が必要です。

なぜベアモジュール指定子なのですか？ベアモジュール指定子を使用すると、パッケージマネージャーがモジュールをインストールした場所を正確に知らなくてもモジュールをインポートできます。[Import maps](https://github.com/WICG/import-maps)と呼ばれる標準の提案により、ブラウザはベアモジュール指定子をサポートできるようになります。それまでの間、ベアインポート指定子はビルドステップとして簡単に変換できます。インポートマップをサポートするポリフィルとモジュールローダーもいくつかあります。

### Supporting older browsers

IE11用だから今回は必要ありません。

## Optimizations

LitElementプロジェクトは、他のWebプロジェクトと同じ最適化の恩恵を受けます。

- バンドル(Rollup or webpack)
- コードの縮小/最適化(Terserは最新のJavaScriptをサポートしているため、LitElementでうまく機能します)
- 圧縮(gzipなど)

さらに、LitElement固有の最適化があります。

- テンプレートリテラルを縮小します。Lit-htmlテンプレートは、標準のHTMLミニファイアでは処理されないテンプレートリテラルを使用して定義されます。テンプレートリテラルを最小化するプラグインを追加すると、コードサイズがわずかに減少する可能性があります。(テンプレートが非常に大きくない限り、これは小さな最適化です)
- shady-renderをコンパイルします。最新のブラウザのみをサポートしている場合は、古いブラウザをサポートするために使用されていたshady-renderモジュールをコンパイルできます。

### Code minification

コードの縮小には多くのオプションがあります。[terser](https://www.npmjs.com/package/terser)は、LitElementで使用される最新のJavaScriptでうまく機能します。

### Minify template literals

Lit-htmlテンプレートは、標準のHTMLミニファイアでは処理されないテンプレートリテラルを使用して定義されます。テンプレートリテラルを最小化するプラグインを追加すると、コードサイズがわずかに減少する可能性があります。(テンプレートが非常に大きくない限り、これは小さな最適化です)

この最適化を実行するために、いくつかのパッケージを利用できます。

- Rollup: [rollup-plugin-minify-html-literals](https://www.npmjs.com/package/rollup-plugin-minify-html-literals?activeTab=readme)
- Webpack: [minify-template-literal-loader](https://www.npmjs.com/package/minify-template-literal-loader)

### Compile out the shady-render module

最新のブラウザのみを対象に構築している場合は、LitElementのShady DOMの組み込みサポートであるShadowDOMポリフィルを削除して、バンドルサイズを約12KB節約できます。

そのためには、`shady-render`モジュールを、`render`の汎用バージョンを提供するベースの`lit-html`モジュールに置き換えるように、ビルドシステムを設定します。これにより、バンドルから約12KB節約できます。

ロールアップビルドの場合

1. `alias`プラグインをインストールします。

    ```bash
    npm i -D @rollup/plugin-alias
    ```

2. `shady-render`モジュールへの参照をメインの`lit-html`モジュールへの参照に置き換えるようにエイリアスプラグインを構成します。

    ```js
    alias({
      entries: [{
        find: 'lit-html/lib/shady-render.js',
        replacement: 'node_modules/lit-html/lit-html.js'
      }]
    }),
    ```

webpackビルドの場合

- 次のresolve.alias設定をwebpack構成に追加します。

    ```js
    resolve: {
      alias: {
        'lit-html/lib/shady-render.js': path.resolve(__dirname, './node_modules/lit-html/lit-html.js')
      }
    },
    ```

## TypeScript

TypeScript言語は、型と型チェックを追加することでJavaScriptを拡張します。TypeScriptコンパイラ`tsc`は、TypeScriptを標準のJavaScriptにコンパイルします。

バンドルプロセスの一部としてTypeScriptコンパイラを実行することは可能ですが、プロジェクトの中間JavaScriptバージョンを生成するために、個別に実行することをお勧めします。古いブラウザ用のTypeScriptコンパイラの出力に問題があるためです。最新のJavaScript（ES2017ターゲットおよびESモジュール）を出力するようにTypeScriptを構成することをお勧めします。

たとえば、`tsconfig.json`ファイルがある場合、最新のJavaScriptを出力するために次のオプションを含めます。

```json
{
  "compilerOptions": {
    "target": "es2017",
    "module": "es2015",
    ...
```

この中間バージョンをビルドツールへの入力として使用します。