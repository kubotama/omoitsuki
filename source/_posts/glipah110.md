---
title: IPアドレスの履歴を表示する機能を追加しました
# date: 2020-05-14 15:25:15
tags:
  - glipah
  - Netlify
  - Vue.js
  - Jest
---

[GLIPAH: Global IP Address History](https://glipah.netlify.app/)に、これまでに取得したIPアドレスの履歴を表示する機能を追加しました。

<img src="{% post_path glipah110 %}Screenshot_2020-05-14GLIPAH.png" alt="GLIPAHの画面" style="border:1px solid #000000;" />

履歴を確認できることで、プロバイダから割り当てられているIPアドレスが変更されたことに気がつきやすくなります。

取得したIPアドレスの履歴は、IndexedDB APIを利用してwebブラウザが動作しているクライアント環境に保存します。サーバー環境にはIPアドレスを保存しません。

これまでは、webページにアクセスしたタイミングでIPアドレスを取得していたが、確認ボタンをクリックするタイミングで取得することに変更しました。

## 処理の流れ

IPアドレスの履歴をIndexedDBに保存することにしたため、これまでと処理の流れを変更しました。これまでは、以下の通りです。

<img src="{% post_path glipah110 %}システム構成V1.0.0.png" alt="GLIPAH V1.0.0の処理の流れ" />

以下のような処理の流れに変更しました。

<img src="{% post_path glipah110 %}システム構成V1.1.0.png" alt="GLIPAH V1.1.0の処理の流れ" />

ファンクションから取得したIPアドレスを表示する前にIndexedDBに保存して、IndexedDBから取得した履歴を表示します。

## IPアドレスの履歴の保存

IPアドレスの履歴をIndexedDBに保存します。IndexedDBへのアクセスにはDexie.jsを利用します。
IndexedDBは、いわゆるnon-SQLデータベースです。以下のようにスキーマを定義します。

```javascript
db.version(1).stores({ access: "++id, ipAddress" });
```

idは自動インクリメントな主キーとして、ipAddressはその他のキーとして定義しています。
idは、tableをv-forで表示するために必要となるユニークなキーとするための項目です。
アクセス日時としてaccessDateを保存しますが、キーとしないため、スキーマでの定義は不要です。

以下のようにIPアドレスとアクセス日時を保存します。

```javascript
          return db.access
            .add({
              ipAddress: ipAddress,
              accessDate: accessDate
            })
            .then(() => {
              return this.loadHistory();
            });
```

`db.access.add`はPromiseを返すので、thenブロックでIPアドレスの履歴をIndexedDBから読み込みます。

## IPアドレスの履歴の表示

IndexedDBから読み込んだデータをipHistory配列に展開します。
読み込みでもスキーマを指定します。

```javascript
db.version(1).stores({ access: "++id, ipAddress" });
```

toArrayメソッドで、保存されている履歴を配列として読み込みます。
toArrayメソッドはPromiseを返すので、thenブロックで読み込んだ配列の要素ごとにaddHistoryメソッドを呼び出します。

```javascript
      return db.access.toArray().then(addresses => {
        addresses.forEach(address => {
          this.addIpHistory(address.id, address.ipAddress, address.accessDate);
        });
      });
```

addHistoryメソッドでは、idとipAccess, accessDateをipHistory配列の先頭に追加します。

```javascript
    /**
     * ipHistory配列の先頭にIPアドレスとアクセス日時を追加する。
     * @param {integer} id データベース項目のid
     * @param {string} ipAddress アクセス元のIPアドレス
     * @param {string} accessDate アクセスした日時
     */
    addIpHistory(id, ipAddress, accessDate) {
      this.ipHistory.unshift({
        id: id,
        ipAddress: ipAddress,
        accessDate: accessDate
      });
    },
```

ipHistory配列はtableに割り当てているため、展開されたデータがtableとして表示されます。

```html
      <table id="ipHistory">
        <thead>
          <tr>
            <th>id</th>
            <th>IPアドレス</th>
            <th>アクセス日時</th>
          </tr>
        </thead>
        <tbody v-for="item in ipHistory" :key="item.id">
          <tr>
            <td>{{ item.id }}</td>
            <td>{{ item.ipAddress }}</td>
            <td>{{ item.accessDate }}</td>
          </tr>
        </tbody>
      </table>
```
