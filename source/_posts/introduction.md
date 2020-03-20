---
title: はじめに
date: 2020-03-20 16:46:15
tags:
  - must
  - Hexo
---

このサイトでは、[MarkUp Support Tool(MUST)](https://must-kubotama.netlify.com/)の開発と、このサイトをHexoで構築するにあたって、思いついたことを書く。

MUSTは、以下の機能を作成する。

- Markdown, LaTeXの特殊文字をエスケープ
- 入力したURLからMarkdown, LaTeXのリンク形式に変換

このサイトは、以下の手順で構築する。

1. Markdown形式で作成したファイルをGitで管理する。
2. コミットしたファイルをGitHubにアップロードする。
3. masterブランチが更新されるとnetlifyに登録しているコマンド(yarn run build)でサイトに反映する。
