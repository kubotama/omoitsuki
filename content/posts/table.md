---
title: IPアドレスの一覧表の形式を修正
date: 2020-06-06
---

[GLIPAH](https://glipah.netlify.app/)サイトのIPアドレスの一覧表の形式を変更します。

これまでの一覧表は以下の形式です。

<table>
  <thead>
    <tr>
      <th>id</th>
      <th>IPアドレス</th>
      <th>アクセス日時</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2</td>
      <td>xx.xx.xx.xx</td>
      <td>2020/05/31 21:46:18</td>
    </tr>
    <tr>
      <td>1</td>
      <td>xx.xx.xx.xx</td>
      <td>2020/05/31 21:46:17</td>
    </tr>
  </tbody>
</table>

<style>
table {
  margin-left: auto;
  margin-right: auto;
  margin-top: 1em;
  border-collapse: collapse;
}
td,
th {
  border: 1px solid;
  margin: 0;
  padding: 0.2em 3em;
}
</style>

これを以下の形式に変更します。

<table>
  <thead>
    <tr>
      <th>IPアドレス</th>
      <th>アクセス回数</th>
      <th>初回のアクセス日時</th>
      <th>最新のアクセス日時</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>xx.xx.xx.xx</td>
      <td>2</td>
      <td>2020/05/31 21:46:18</td>
      <td>2020/05/31 21:46:18</td>
    </tr>
    <tr>
      <td>yy.yy.yy.yy</td>
      <td>10</td>
      <td>2020/05/31 21:46:17</td>
      <td>2020/05/31 21:46:17</td>
    </tr>
  </tbody>
</table>

IPアドレスごとにまとめて、各IPアドレスごとにアクセス回数、初回と最新のアクセス日時を表示します。
各行は初回のアクセス日時が新しいIPアドレスが上にソートします。
これは、次にIPアドレスごとに、なんらかの処理、たとえばGoogleアナリティクスのフィルターとして登録したことを記録するための準備です。

<!--more-->

## 影響するテストをスキップ

一覧表をHTMLレベルで確認するテストは修正が一区切りするまでエラーになるため、スキップさせます。

## テストケースの見直し

一覧表の修正にあわせて、テストケースを見直します。
以下は、現状のHTMLレベルで確認するテストケースです。

### ケース1

IPアドレスとアクセス日時をipHistory配列に設定します。

| IPアドレス | アクセス日時 |
|-----|-----|
|zz.zz.zz.zz|2020-04-30 12:34:56|

以下のように表示されます。

<table>
  <thead>
    <tr>
      <th>IPアドレス</th>
      <th>アクセス回数</th>
      <th>初回のアクセス日時</th>
      <th>最新のアクセス日時</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>zz.zz.zz.zz</td>
      <td>1</td>
      <td>2020-04-30 12:34:56</td>
      <td>2020-04-30 12:34:56</td>
    </tr>
  </tbody>
</table>

### ケース2

IPアドレスとアクセス日時をipHistory配列に設定します。

| IPアドレス | アクセス日時 |
|-----|-----|
|zz.zz.zz.zz|2020-04-30 12:34:56|
|yy.yy.yy.yy|2020-05-01 11:11:11|

<table>
  <thead>
    <tr>
      <th>IPアドレス</th>
      <th>アクセス回数</th>
      <th>初回のアクセス日時</th>
      <th>最新のアクセス日時</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>yy.yy.yy.yy</td>
      <td>1</td>
      <td>2020-05-01 11:11:11</td>
      <td>2020-05-01 11:11:11</td>
    </tr>
    <tr>
      <td>zz.zz.zz.zz</td>
      <td>1</td>
      <td>2020-04-30 12:34:56</td>
      <td>2020-04-30 12:34:56</td>
    </tr>
  </tbody>
</table>

### ケース3

axios.getとDateをモックにして、テストデータを返します。

| IPアドレス | アクセス日時 |
|-----|-----|
|ab.cd.ef.gh|2020-05-06 01:02:03|

以下のように表示されます。

<table>
  <thead>
    <tr>
      <th>IPアドレス</th>
      <th>アクセス回数</th>
      <th>初回のアクセス日時</th>
      <th>最新のアクセス日時</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ab.cd.ef.gh</td>
      <td>1</td>
      <td>2020-05-06 01:02:03</td>
      <td>2020-05-06 01:02:03</td>
    </tr>
  </tbody>
</table>

### ケース4

axios.getとDateをモックにして、テストデータを返します。

| IPアドレス | アクセス日時 |
|-----|-----|
|ab.cd.ef.gh|2020-05-06 01:02:03|
|ab.cd.ef.gh|2020-05-06 01:02:03|

<table>
  <thead>
    <tr>
      <th>IPアドレス</th>
      <th>アクセス回数</th>
      <th>初回のアクセス日時</th>
      <th>最新のアクセス日時</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ab.cd.ef.gh</td>
      <td>2</td>
      <td>2020-05-06 01:02:03</td>
      <td>2020-05-06 01:02:03</td>
    </tr>
  </tbody>
</table>

### ケース5

axios.getとDateをモックにして、テストデータを返します。

| IPアドレス | アクセス日時 |
|-----|-----|
|ab.cd.ef.gh|2020-05-06 01:02:03|
|11.22.33.44|2020-05-10 00:11:22|

以下のように表示されます。

<table>
  <thead>
    <tr>
      <th>IPアドレス</th>
      <th>アクセス回数</th>
      <th>初回のアクセス日時</th>
      <th>最新のアクセス日時</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>11.22.33.44</td>
      <td>1</td>
      <td>2020-05-10 00:11:22</td>
      <td>2020-05-10 00:11:22</td>
    </tr>
    <tr>
      <td>ab.cd.ef.gh</td>
      <td>1</td>
      <td>2020-05-06 01:02:03</td>
      <td>2020-05-06 01:02:03</td>
    </tr>
  </tbody>
</table>

上記のテストケースのうち、以下のように変更します。

- テストケース1と2はテストケース3と5と重複しているため、削除します。

- ケース4の2回目のアクセス日時を2020-05-07 03:49:38に変更して、初回のアクセス日時と最新アクセス日時の違うテストをします。

あわせて以下のテストケースを追加します。追加するテストケースは、axios.getとDateをモックにしてテストデータを返します。

### ケース6

3回とも同じIPアドレスからのアクセス、日時の順にアクセスされています。

| IPアドレス | アクセス日時 |
|-----|-----|
|ab.cd.ef.gh|2020-05-06 01:02:03|
|ab.cd.ef.gh|2020-05-21 07:23:39|
|ab.cd.ef.gh|2020-06-01 22:11:00|

<table>
  <thead>
    <tr>
      <th>IPアドレス</th>
      <th>アクセス回数</th>
      <th>初回のアクセス日時</th>
      <th>最新のアクセス日時</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ab.cd.ef.gh</td>
      <td>3</td>
      <td>2020-05-06 01:02:03</td>
      <td>2020-06-01 22:11:00</td>
    </tr>
  </tbody>
</table>

### ケース7

3回とも同じIPアドレスからのアクセス、アクセスの順序が入れ替わっています。

| IPアドレス | アクセス日時 |
|-----|-----|
|ab.cd.ef.gh|2020-05-06 01:02:03|
|ab.cd.ef.gh|2020-06-01 22:11:00|
|ab.cd.ef.gh|2020-05-21 07:23:39|

<table>
  <thead>
    <tr>
      <th>IPアドレス</th>
      <th>アクセス回数</th>
      <th>初回のアクセス日時</th>
      <th>最新のアクセス日時</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ab.cd.ef.gh</td>
      <td>3</td>
      <td>2020-05-06 01:02:03</td>
      <td>2020-06-01 22:11:00</td>
    </tr>
  </tbody>
</table>

### ケース8

3回目に別のIPアドレスからアクセスされています。

| IPアドレス | アクセス日時 |
|-----|-----|
|ab.cd.ef.gh|2020-05-06 01:02:03|
|ab.cd.ef.gh|2020-06-01 22:11:00|
|11.22.33.44|2020-06-10 00:11:22|

<table>
  <thead>
    <tr>
      <th>IPアドレス</th>
      <th>アクセス回数</th>
      <th>初回のアクセス日時</th>
      <th>最新のアクセス日時</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>11.22.33.44</td>
      <td>1</td>
      <td>2020-06-10 00:11:22</td>
      <td>2020-06-10 00:11:22</td>
    </tr>
    <tr>
      <td>ab.cd.ef.gh</td>
      <td>2</td>
      <td>2020-05-06 01:02:03</td>
      <td>2020-06-01 22:11:00</td>
    </tr>
  </tbody>
</table>

### ケース9

2回めに別のIPアドレスからアクセスされています。

| IPアドレス | アクセス日時 |
|-----|-----|
|ab.cd.ef.gh|2020-05-06 01:02:03|
|11.22.33.44|2020-05-10 00:11:22|
|ab.cd.ef.gh|2020-06-01 22:11:00|

<table>
  <thead>
    <tr>
      <th>IPアドレス</th>
      <th>アクセス回数</th>
      <th>初回のアクセス日時</th>
      <th>最新のアクセス日時</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>11.22.33.44</td>
      <td>1</td>
      <td>2020-05-10 00:11:22</td>
      <td>2020-05-10 00:11:22</td>
    </tr>
    <tr>
      <td>ab.cd.ef.gh</td>
      <td>2</td>
      <td>2020-05-06 01:02:03</td>
      <td>2020-06-01 22:11:00</td>
    </tr>
  </tbody>
</table>

### ケース10

初回のアクセス日時の更新を確認します。

| IPアドレス | アクセス日時 |
|-----|-----|
|ab.cd.ef.gh|2020-06-01 22:11:00|
|ab.cd.ef.gh|2020-05-06 01:02:03|
|11.22.33.44|2020-05-10 00:11:22|

<table>
  <thead>
    <tr>
      <th>IPアドレス</th>
      <th>アクセス回数</th>
      <th>初回のアクセス日時</th>
      <th>最新のアクセス日時</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>11.22.33.44</td>
      <td>1</td>
      <td>2020-05-10 00:11:22</td>
      <td>2020-05-10 00:11:22</td>
    </tr>
    <tr>
      <td>ab.cd.ef.gh</td>
      <td>2</td>
      <td>2020-05-06 01:02:03</td>
      <td>2020-06-01 22:11:00</td>
    </tr>
  </tbody>
</table>

## プログラムの修正

テンプレートを以下のように修正します。

```html
        <thead>
          <tr>
            <th>IPアドレス</th>
            <th>アクセス回数</th>
            <th>初回のアクセス日時</th>
            <th>最新のアクセス日時</th>
          </tr>
        </thead>
        <tbody v-for="item in ipHistory" :key="item.ipAddress">
          <tr>
            <td>{{ item.ipAddress }}</td>
            <td>{{ item.accessCount }}</td>
            <td>{{ item.firstAccessDate }}</td>
            <td>{{ item.lastAccessDate }}</td>
          </tr>
        </tbody>
```

ipHistory配列への代入を以下のように修正します。

```javascript
    addIpHistory(id, ipAddress, accessDate) {
      const index = this.ipHistory.findIndex(
        address => address.ipAddress === ipAddress
      );
      if (index == -1) {
        this.ipHistory.unshift({
          id: id,
          ipAddress: ipAddress,
          accessCount: 1,
          firstAccessDate: accessDate,
          lastAccessDate: accessDate
        });
      } else {
        if (this.ipHistory[index].firstAccessDate > accessDate) {
          this.ipHistory[index].firstAccessDate = accessDate;
        }
        if (this.ipHistory[index].lastAccessDate < accessDate) {
          this.ipHistory[index].lastAccessDate = accessDate;
        }
        this.ipHistory[index].accessCount++;
      }
    },
```
