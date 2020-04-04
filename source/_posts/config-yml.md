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

## タイトル、コピーライトの修正

## shareボタンの修正

## faviconの設定

[ICOON MONO](https://icooon-mono.com/)から[頭のアイコン](https://icooon-mono.com/10226-%e9%a0%ad%e3%81%ae%e3%82%a2%e3%82%a4%e3%82%b3%e3%83%b3/)を、色はrgb(16, 64, 239)、サイズは256pxを指定して、pngファイルとしてダウンロードした。

ダウンロードしたpngファイルを[favicon generator](https://favicon.il.ly/)でicoファイルに変換した。

変換したファイルを/themes/landscape/sourceにfavicon.icoとして置いた。

/themes/landscape/_config.ymlのfavicon項目をfavicon.icoに変更した。

## Google analyticsの設定
