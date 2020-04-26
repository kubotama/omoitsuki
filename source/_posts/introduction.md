---
title: はじめに
date: 2020-03-20 16:46:15
tags:
  - must
  - Hexo
---

このサイトでは、[MarkUp Support Tool(MUST)](https://github.com/kubotama/must-netlify)の開発と、このサイトをHexoで構築するときに、JavaScriptやvue.js, Netlifyなど思いついたことを書く。

MUSTは、以下の機能がある。

- Markdownの特殊文字をエスケープ

今後、以下の機能を追加する。

- 入力したURLからMarkdownのリンク形式に変換
- LaTeXの特殊文字をエスケープする。
- 入力したURLからLaTeXのリンク形式に変換

このサイトは、Hexoを利用して、以下の手順で構築する。

1. Markdown形式で作成したファイルをGitで管理する。
2. コミットしたファイルをGitHubにアップロードする。
3. masterブランチが更新されるとnetlifyに登録しているコマンド(yarn run build)でサイトに反映する。
