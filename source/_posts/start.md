---
title: Netlify Functionsのローカル環境をyarn startで起動する設定
# date: 2020-05-11 15:51:32
tags:
  - Netlify
---

Netlify Functionsの開発、検証用としてローカルでの実行環境が用意されている。netlify-lambdaパッケージをローカルにインストールしている場合、以下のコマンドを実行することで`http://localhost:9000/.netlify/functions/ファンクション名`にアクセスできるようになる。

```sh
% npx netlify-lambda serve src/lambda
```

通常は、ファンクションだけではなく、webサイトとあわせて起動する必要がある。たとえばwebサイトをVue.jsで開発している場合、webサイトは以下のコマンドで起動する。

```sh
% vue-cli-service serve
```

これらをpackage.jsonに登録することで、npmあるいはyarnコマンドで実行できるようになる。さきにファンクションの環境を起動して、しばらくしてからwebページを起動しないと、ファンクションが起動できない。startにwebページの起動、prestartにファンクションの起動と3秒の待機(sleep 3)を設定することで、yarn startでwebページとファンクションが起動できるようになる。

```json
...
  "scripts": {
    "start": "vue-cli-service serve",
    "prestart": "npx netlify-lambda serve src/lambda & sleep 3",
...
```
