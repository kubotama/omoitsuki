---
title: 入力したURLからMarkdownのリンク形式に変換
tags:
  - must
  - Netlify
---

URLを入力して、Markdownのリンク形式に変換する。たとえば<https://www.google.co.jp/>が入力されたら、\[Google\](<https://www.google.co.jp>)に変換する。表示する見出しは、入力されたURLのwebページのtitleタグから取得する。

webブラウザで実行しているJavaScriptから入力されたURLにアクセスするとCORSとなるため、Netlify Functionsを利用する。Netlify Functionsのコード(この後は、コードとする)で入力されたタイトルを取得して、リンクを作成する。

[Netlify functionsを利用したサーバーレスアプリケーションの開発](https://omoitsuki.netlify.app/2020/04/17/functions/)を参考にして、Netlify Functionsのコードと、呼び出すメソッドを作成する。

## netlify-lambdaとnetlify-lambdaパッケージのvue-cliプラグインをインストール

Netlify Functionsをローカル環境でテストするために、以下のコマンドでnetlify-lambdaをインストールする。

```sh
% yarn add netlify-lambda
```

続いて以下のコマンドで、netlify-lambdaパッケージのvue-cliプラグインをインストールする。

```sh
% vue add netlify-lambda
```

プラグインをインストールすると、以下のファイルが生成される。

netlify.toml

```yaml
[build]
  command = "yarn build"
  functions = "lambda"
  publish = "dist"
```

src/lambda/hello.js

```javascript
export function handler(event, context, callback) {
  console.log(event);
  callback(null, {
    statusCode: 200,
    body: JSON.stringify({ msg: "Hello, World!" })
  });
}
```

### コードのURL取得のテストを作成

コードのURLを返すメソッドのテストを追加する。テストの要件は以下の通りとする。

- メソッド名をgetFunctionUrlとする。MustUi.vueに定義する。
- 引数としてアクセスしているwebページのURL(location.href)を受け取る。
- ホスト名がlocalhostの場合には、ポート番号を9000とする。
- webページのURLに/.netlify/functions/titleを追加したURLを返す。

テストするURLは以下の通りとする。

| 引数として渡すURL | 返すべきURL |
| --- | --- |
| <http://localhost:8080> | <http://localhost:9000/.netlify/functions/title> |
| <https://must-kubotama.netlify.app/> | <https://must-kubotama.netlify.app/.netlify/functions/title> |

## コードのURL取得のテストを成功するメソッドを作成

src/components/MustUi.vueにgetFunctionUrlを作成する。

## コードを確認するテストを作成

コードの仕様を以下のとおりとする。

- 名称はtitleとする。URLは<http://サーバー名/.netlify/functions/title>となる。
- GETでアクセスされることを前提とする。タイトルを取得するURLはクエリ文字列として渡される。
- 渡されたURLにGETでアクセスしてタイトルを取得する。
- 取得したタイトルはレスポンスとして返される。
- タイトルが取得できた場合には、ステータスコードは200を返す。
- 渡されたURLのwebページが存在しない場合、あるいは渡されたURLのwebページからタイトルが取得できない場合、ステータスコードは204を返す。

上記の仕様を確認するテストパターンは以下の通りとする。

| URL || ステータスコード | タイトル |
|-----||------------------|----------|
| <http://example.com> || 200 | Example Domain |
| <https://must-kubotama.netlify.app/> || 200 | Markup Support Tool |
| <https://omoitsuki.netlify.app> || 200 | 思いつきを書くブログ |
| || 204 | |
| <http://localhost> || 204 | |

上記を確認するテストを作成する。ファイル名をtitle.node.spec.jsとする。

## コードを確認するテストを成功する関数を作成

src/lambda/hello.jsをsrc/lambda/title.jsに変更して、テストを成功する関数を作成する。

## ボタンをクリックすると呼び出されるメソッドのテストを作成

ボタンをクリックすると呼び出されるメソッドのテストを作成する。メソッドの要件は以下の通りとする。

- メソッド名はonMdLinkとする。
- テキスト領域になにも入力されていなければ、なにもしないで終了する。つまりテキスト領域は空白のままである。
- テキスト領域に入力された文字列をクエリ文字列として、<http://サーバー名/.netlify/functions/title>にGETでアクセスする。サーバー名はアクセスしているwebページと同じとする。
- ステータスコードが200で返ってきた場合には、テキスト領域をMarkdownのリンク形式に書き換える。たとえば、<http://example.com/>の場合には、[Example Domain](http://example.com/)となる。
- ステータスコードが204で返ってきた場合には、なにもしないで終了する。つまりテキスト領域はそのままである。

テストでは、実際にwebページにはアクセスしないでモックを利用する。モックが呼び出された回数と引数を確認する。モックからタイトルを返して、テキスト領域が正しく更新されることを確認する。

上記の仕様を確認するテストパターンは以下の通りとする。

| テキスト領域 | モックが呼び出された回数 | モックの引数 | ボタンをクリックした後のテキスト領域 |
|--------------|--------------------------|--------------|--------------------------------------|
| (空白) | 0 | なし | (空白) |
| <http://example.com/> | 1 | <http://example.com/> | \[Example Domain\](<http://example.com/>) |
| <https://must-kubotama.netlify.app/> | 1 | <https://must-kubotama.netlify.app/> | \[Markup Support Tool\](<https://must-kubotama.netlify.app/>) |
| <http://localhost> | 1 | <http://localhost> | <http://localhost> |

上記を確認するテストを作成する。ファイル名をmd.link.spec.jsとする。

## メソッドを確認するテストを成功するメソッドを作成

src/components/MustUi.vueのonMdLinkメソッドを作成する。
