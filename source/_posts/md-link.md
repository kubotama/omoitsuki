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
| <http://localhost/> || 204 | |

上記を確認するテストを作成する。ファイル名をtitle.node.spec.jsとする。

## テストを成功する関数を作成

src/lambda/hello.jsをsrc/lambda/title.jsに変更して、テストを成功する関数を作成する。

```javascript
```
