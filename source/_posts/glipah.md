---
title: グローバルIPアドレスの変更を確認するwebサイトの作成
tags:
  - Netlify
  - Vue.js
  - glipah
---

通常は、プロバイダーから割り当てられたグローバルなIPアドレスを気にする必要はない。ただし、たとえば一日のアクセス数が少ないブログなどのアクセス数をGoogleアナリティクスで確認するときに、自分のアクセスを除外するためにIPアドレスでフィルターを作成する場合には、自分に割り当てられているグローバルなIPアドレスを確認する必要がある。また、プロバイダーからの割り当ては、いつ変更されるのかわからないため、定期的に確認する必要もある。

このwebサイト([GLIPAH: Global IP Address History](https://glipah.netlify.app/))にアクセスすると、プロバイダーに割り当てられているグローバルなIPアドレスを確認できる。IPアドレスは、サーバーに保存しない。ただし、Googleアナリティクスでアクセス情報を収集している。

## 構築方法

GitHubのリポジトリの作成、GitHubのリポジトリとNetlifyのサイトの関連付け、GitHubのリポジトリのローカルの環境へのクローン、vueプロジェクトの作成、Netlify Functionのパッケージのインストールなどは、[Netlify functionsを利用したサーバーレスアプリケーションの開発](https://omoitsuki.netlify.app/2020/04/17/functions/)の通りとする。

## ファンクションの作成

アクセス元のIPアドレスを返すファンクションを作成する。アクセス元のIPアドレスは、Netlify環境ではリクエストのヘッダのclient-ip属性を参照する。ローカル環境ではアクセス元のIPアドレスが設定されている属性がないので、ダミーのIPアドレス(xx.xx.xx.xx)を返す。

ローカル環境では、webページとファンクションのポートが違うためCORS違反を回避するため、レスポンスのヘッダにAccess-Control-Allow-Origin属性を設定する必要がある。ローカル環境の判定は、event.headers.originあるいはevent.headers.refererに設定されているURLのポート番号が8080の場合、とする。

```javascript
export function handler(event, context, callback) {
  const returnData = {
    statusCode: 200,
    headers: { "Content-Type": "text/plain" }
  };
  // テスト環境では、ボタンが表示されているページとNetlify Functionsのポート番号が違うためCORS制約に違反する。
  // CORS制約を回避するためにAccess-Control-Allow-Origin属性を設定する。
  // ローカル環境の判定は、event.headers.originあるいはevent.headers.refererに設定されているURLのポート番号が8080の場合、
  // とする。
  let url;
  if (event.headers.origin) {
    url = new URL(event.headers.origin);
  } else if (event.headers.referer) {
    url = new URL(event.headers.referer);
  }
  if (url && url.port == 8080) {
    returnData.body = "xx.xx.xx.xx";
    if (!event.headers["user-agent"].match(/axios/)) {
      if (event.headers.origin) {
        returnData.headers["Access-Control-Allow-Origin"] =
          event.headers.origin;
      } else if (event.headers.referer) {
        returnData.headers["Access-Control-Allow-Origin"] =
          event.headers.referer;
      }
    }
  } else {
    returnData.body = event.headers["client-ip"];
  }
  callback(null, returnData);
}
```
