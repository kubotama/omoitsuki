---
title: はじめに
date: 2020-03-20 16:46:15
tags:
  - must
  - glipah
---

このサイトでは、個人的なソフトウェア開発や、このサイトの構築について、JavaScriptやVue.js, Netlifyなど思いついたことを書いています。

<!--more-->

いまのところ、以下のソフトウェアを開発しています。

- [MUST: Markup Language Support Tool](https://github.com/kubotama/must-netlify)
- [GLIPAH: Global IP Address History](https://glipah.netlify.app/)

## MUST

MUSTは、以下の機能があります。

- Markdownの特殊文字をエスケープ
- 入力したURLからMarkdownのリンク形式に変換

今後、以下の機能を追加する予定です。

- LaTeXの特殊文字をエスケープする。
- 入力したURLからLaTeXのリンク形式に変換

## GLIPAH

GLIPAHはプロバイダから割り当てられるグローバルなIPアドレスを確認します。

## このサイトの構築

このサイトは、Hugoを利用して、以下の手順で構築します。

1. Markdown形式で作成したファイルをGitで管理します。
2. コミットしたファイルをGitHubにアップロードします。
3. masterブランチが更新されるとnetlifyに登録しているコマンドでサイトに反映します。
