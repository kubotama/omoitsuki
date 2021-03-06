---
title: 環境設定の見直し
date: 2020-04-03 23:06:49
tags:
  - Hexo
---

_config.ymlが初期設定のままなので、ここで見直して、以下の項目を修正する。

- タイトル、コピーライトを修正する。
- shareボタンを修正、あるいは削除する。
- faviconを設定する。
- Google analyticsを設定する。

<!--more-->

## タイトル、コピーライトの修正

ブログの設定ファイル(/_config.yml)の以下の項目を変更した。

- title項目を「思いつきを書くブログ」に変更した。これによって、サイトのヘッダのタイトルが変更された。
- authorを「kubotama」に変更した。これによって、サイトのフッタに書かれているコピーライトが変更された。
- languageを「ja」に変更した。これによって、アーカイブのMarch 2020が3月 2020に変更された。shareリンクが共有リンクに変更された。
- timezoneに「'Asia/Tokyo'」を設定した。どこに反映されたのか、わからない。

## 共有ボタンの修正

ブログの設定ファイル(/_config.yml)のurl項目を「<https://omoitsuki.netlify.app/>」に変更した。これによって、共有リンクをクリックしたときにポップアップされて表示されるリンクが変更された。

## faviconの設定

1. [ICOON MONO](https://icooon-mono.com/)から[頭のアイコン](https://icooon-mono.com/10226-%e9%a0%ad%e3%81%ae%e3%82%a2%e3%82%a4%e3%82%b3%e3%83%b3/)を、色はrgb(16, 64, 239)、サイズは256pxを指定して、pngファイルとしてダウンロードした。
1. ダウンロードしたpngファイルを[favicon generator](https://favicon.il.ly/)でicoファイルに変換した。
1. 変換したファイルを/themes/landscape/sourceにfavicon.icoとして置いた。
1. テーマの設定ファイル(/themes/landscape/_config.yml)のfavicon項目をfavicon.icoに変更した。

## Google analyticsの設定

_config.ymlの設定以外に、Googleアナリティクスのコンソールの設定も必要そうなので、別エントリーとする。
