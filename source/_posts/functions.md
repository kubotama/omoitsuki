---
title: Netlify functionsの設定
date: 2020-03-21 22:22:06
tags:
  - must
  - Netlify
---

## 目的

[MUST](https://must-kubotama.netlify.com/)にNetlify functionsの機能を追加する。
これは入力されたURLのサイトのタイトルを取得するために、ブラウザで実行しているJavaScriptからサイトにアクセスすると
発生するCORS違反を回避するためである。titleを取得するAPIなのでエンドポイント名をtitleとする。

## 作業手順

[Netlifyのドキュメント](https://docs.netlify.com/functions/configure-and-deploy/)を参考にして、以下の手順で設定する。

### netlify.tomlを作成して、command, functionsを定義する

[netlify-lambdaのGitHub](https://github.com/netlify/netlify-lambda)によると、netlify-lambda buildを実行すると、コマンドで指定したディレクトリのファイルから、netlify.tomlのfunctionsで指定したディレクトリに出力するとある。netlify.tomlは以下の通りに作成する。

```json:netlify.toml
[build]
  build = yarn run build
  functions = /dist/api
```

### netlify-lambdaをインストールする

以下のコマンドでnetlify-lambdaをインストールする。

```bash
% yarn add netlify-lambda
```

### package.jsonにbuild, devを追加する

既にpackage.jsonには以下のように定義されている。

```json:package.json
{
  "scripts": {
    "build": "vue-cli-service build"
  }
}
```

これを以下の通りに変更する。

```json:package.json
{
  "scripts": {
    "build": "vue-cli-service build; netlify-lambda build /functions"
  }
}
```

### 1で指定したディレクトリにtitle.jsを作成する

ダミーとして以下のファイルを/functionsディレクトリに作成する。

```title.js
exports.handler = function(event, context, callback) {
  callback(null, {
    statusCode: 200,
    body: 'Hello World'
  });
}
```

### ローカル環境でビルドする

以下のコマンドを実行する。

```bash
% yarn run build
```

### [ローカル環境のエンドポイント](http://localhost:9000/.netlify/functions/title)にアクセスすることで動作を確認する

ブラウザからアクセスして'Hello World'が返されることを確認する。

## 作業結果

### 【結果】netlify.tomlを作成して、command, functionsを定義する

### 【結果】netlify-lambdaをインストールする

### 【結果】package.jsonにbuild, devを追加する
