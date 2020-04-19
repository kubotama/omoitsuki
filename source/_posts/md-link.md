---
title: 入力したURLからMarkdownのリンク形式に変換
tags:
  - must
  - Netlify
---

URLを入力して、Markdonwのリンク形式に変換する。たとえば<https://www.google.co.jp/>が入力されたら、\[Google\](<https://www.google.co.jp>)に変換する。表示する見出しは、入力されたURLのwebページのtitleタグから取得する。

webブラウザで実行しているJavaScriptから入力されたURLにアクセスするとCORSとなるため、Netlify Functionsを利用する。Netlify Functionsのコードで入力されたタイトルを取得して、リンクを作成する。
