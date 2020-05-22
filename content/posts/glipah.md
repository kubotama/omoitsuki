---
title: グローバルIPアドレスを確認するwebサイトの作成
date: 2020-05-04
tags:
  - Netlify
  - Vue.js
  - glipah
---

通常は、プロバイダーから割り当てられたグローバルなIPアドレスを気にする必要はない。ただし、たとえば一日のアクセス数が少ないブログなどのアクセス数をGoogleアナリティクスで確認するときに、自分のアクセスを除外するためにIPアドレスでフィルターを作成する場合には、自分に割り当てられているグローバルなIPアドレスを確認する必要がある。また、プロバイダーからの割り当ては、いつ変更されるのかわからないため、定期的に確認する必要もある。

このwebサイト([GLIPAH: Global IP Address History](https://glipah.netlify.app/))にアクセスすると、プロバイダーに割り当てられているグローバルなIPアドレスを確認できる。IPアドレスは、サーバーに保存しない。ただし、Googleアナリティクスでアクセス情報を収集している。

ソースコードは[GitHub \- kubotama/glipah: Global IP Address History](https://github.com/kubotama/glipah)にある。

<!--more-->

## 構築方法

GitHubのリポジトリの作成、GitHubのリポジトリとNetlifyのサイトの関連付け、GitHubのリポジトリのローカルの環境へのクローン、vueプロジェクトの作成、Netlify Functionのパッケージのインストールなどは、[Netlify functionsを利用したサーバーレスアプリケーションの開発](https://omoitsuki.netlify.app/2020/04/17/functions/)の通りとする。

## ファンクションの作成

アクセス元のIPアドレスを返すファンクションを作成する。ファンクションからはリクエストのheadersを返して、どの属性を参照するのかはwebページ側で判断する。

ローカル環境では、webページとファンクションのポートが違うために発生するCORS違反を回避するには、レスポンスのヘッダにAccess-Control-Allow-Origin属性を設定する必要がある。ローカル環境の判定は、event.headers.originあるいはevent.headers.refererに設定されているURLのポート番号が8080の場合、とする。

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

## webページを表示するコンポーネントの作成

webページを表示するコンポーネント(GlipahUi)を作成する。GlipahUiには、IPアドレスを返すファンクションへのアクセスと、ファンクションが返したIPアドレスとアクセスした日時の表示の、2つの機能を作成する。

### IPアドレスを返すファンクションへのアクセス

ファンクションへのアクセスは、mountedのフックから呼び出す。

最初にIPアドレスを返すファンクションへのURLを取得する。Netlify環境の場合にはwebページのURLに".netlify/functions/ipaddress"を追加する。ローカル環境の場合には、ポート番号を9000に設定したURLに".netlify/functions/ipaddress"を追加する。

```javascript
    getFunctionUrl(pageUrl) {
      const url = new URL(pageUrl);
      if (url.port == 8080) {
        url.port = 9000;
      }
      url.pathname = ".netlify/functions/ipaddress";
      return url.href;
    }
```

webページのURLのポートで、Netlify環境とローカル環境の判別する。テストスクリプトの実行時のURLのポート番号をjest.config.jsで設定する。

```javascript
module.exports = {
  ...
  testURL: "http://localhost:8080",
  ...
};
```

取得したURLにアクセスする。

```javascript
  mounted: function() {
    axios.get(this.getFunctionUrl(window.location.href)).then(response => {
      this.addIpHistory(response.data, new Date());
    });
  },
```

### ファンクションが返したIPアドレスとアクセスした日時の表示

IPアドレスとアクセスした日時を表示するtableをHTMLで作成する。
いまは直近の1件のみを表示するが、将来的に履歴を表示するために、v-forのループで表示する。

```html
<template>
  <div>
    <table id="ipHistory">
      <thead>
        <tr>
          <th>IPアドレス</th>
          <th>アクセス日時</th>
        </tr>
      </thead>
      <tbody v-for="item in ipHistory" :key="item.ipAddress">
        <tr>
          <td>{{ item.ipAddress }}</td>
          <td>{{ item.accessDate }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
```

ファンクションが返したIPアドレスとアクセスした日時をipHistory配列の先頭に追加する。
IPアドレスはファンクションから返されたリクエストのヘッダから取得する。
アクセス元のIPアドレスは、Netlify環境ではリクエストのheadersのclient-ip属性を参照する。ローカルの検証環境ではアクセス元のIPアドレスが設定されている属性がないので、ダミーのIPアドレス(xx.xx.xx.xx)とする。

```javascript
    /**
     * ipHistory配列の先頭にIPアドレスとアクセス日時を追加する。
     * @param {json} headers ファンクションのリクエストのヘッダ
     * @param {Date} date アクセスした日時
     */
    addIpHistory(headers, date) {
      this.ipHistory.unshift({
        ipAddress: this.getIpAddress(headers),
        accessDate: this.dateToString(date)
      });
    },

    /**
     * Dateを文字列に変換する
     * @param {Date} date アクセスした日時
     * @returns {string} 文字列に変換した日時
     */
    dateToString(date) {
      /**
       * @type {Object} 日時のフォーマット
       * 年は4桁、月、日、時、分、秒は2桁
       * 1桁の数字の場合は十の位に0を入れる
       */
      const options = {
        year: "numeric",
        month: "2-digit",
        day: "2-digit",
        hour: "2-digit",
        minute: "2-digit",
        second: "2-digit"
      };
      return Intl.DateTimeFormat("ja-JP", options).format(date);
    },

    /**
     * リクエストのheadersからIPアドレスを取り出す。
     * @param {json} headers ファンクションにアクセスしたときのリクエストのheaders
     * @returns {string} アクセス元のIPアドレス
     */
    getIpAddress(headers) {
      /**
       * @type {string} IPアドレス
       * リクエストのheadersのclient-ip属性から取得する
       * client-ip属性が設定されていない場合、xx.xx.xx.xx.とする
       */
      let ipAddress;
      if (headers["client-ip"]) {
        ipAddress = headers["client-ip"];
      } else {
        ipAddress = "xx.xx.xx.xx";
      }
      return ipAddress;
    },
```

ファンクションが非同期で実行されるため、最初にtableの見出しだけが表示される。
ファンクションから値が返されて、ipHistory配列を更新すると、割り当てているtableの表示に反映される。
