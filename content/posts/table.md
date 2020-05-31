---
title: IPアドレスの一覧表の形式を修正
date: 2020-05-30
draft: true
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
これは、次にIPアドレスごとに、なんらかの処理、たとえばGoogleアナリティクスのフィルターとして登録したことを記録するための準備です。

<!--more-->

## 影響するテストをスキップ

## テストケースの見直し
