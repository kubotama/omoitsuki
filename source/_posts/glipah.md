---
title: グローバルIPアドレスの変更を確認するwebサイトの作成
tags:
  - Netlify
  - Vue.js
  - glipah
---

プロバイダーから割り当てられたグローバルなIPアドレスを気にする必要は、通常はあまりない。ただし、たとえば一日のアクセス数が少ないブログなどのアクセス数をGoogleアナリティクスで確認するときに、自分のアクセスを除外したい場合には、自分に割り当てられているグローバルなIPアドレスを確認する必要がある。また、プロバイダーからの割り当ては、いつ変更されるのかわからないため、定期的に確認する必要もある。

このwebサイトにアクセスすると、プロバイダーに割り当てられているグローバルなIPアドレスを確認するとともに、以前に確認したときから変更されているのかがわかる。IPアドレスの履歴などはローカルな環境に保存されるため、サーバーに保存される情報はない。

サイトの名前はglipahとする。GLobal IP Address Historyの頭文字である。

## 画面のデザイン

| IPアドレス | 初回のアクセス日時 | 最後のアクセス日時 | 登録済み |
|-----|-----|-----|-----|
| xx.xx.xx.xx | 2020/04/21 12:34:56 | 2020/04/22 01:23:45 | □ |
| yy.yy.yy.yy | 2020/03/31 00:00:00 | 2020/04/20 11:11:11 | ■ |

各フィールドのデータタイプはIPアドレスはstring、初回のアクセス日時、最後のアクセス日時はDate、登録済みはBooleanとする。
各フィールドのうち、IPアドレスと初回のアクセス日時は、新しいIPアドレスを確認したときに設定されて、そのあとは更新されない。
最後のアクセス日時は、該当するIPアドレスを確認するたびに更新される。登録済みの初期値はfalseであり、Googleアナリティクスのフィルターに登録したら、trueにする。

## データ構造

確認したIPアドレスなどは、WebストレージのlocalStorageに保存する。保存する形式は以下の通りとする。

```json
{ glipah:
  [
    { ipAddress: "xx.xx.xx.xx", firstDatetime: "2020/04/21 12:34:56", lastDatetime: "2020/04/22 01:23:45", isRegistered: false},
    { ipAddress: "yy.yy.yy.yy", firstDatetime: "2020/03/31 00:00:00", lastDatetime: "2020/04/20 11:11:11", isRegistered: true},
  ]
}
```

## 構築方法

GitHubのリポジトリの作成、GitHubとNetlifyのサイトの関連付け、Vue.jsのプロジェクトの作成は
[Netlify functionsを利用したサーバーレスアプリケーションの開発](https://omoitsuki.netlify.app/2020/04/17/functions/)を参考にする。

GitHubのリポジトリの作成、GitHubのリポジトリとNetlifyのサイトの関連付け、GitHubのリポジトリのローカルの環境へのクローン、vueプロジェクトの作成、Netlify Functionのパッケージのインストールなどは、上記のリンク先の通りとする。

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
