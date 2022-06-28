---
title: "URLを選択する機能を追加"
date: 2022-06-28
tags:
  - VSCode
  - Markdown
---

## 概要

[マークアップ言語のリンク作成を支援する VSCode の拡張機能](https://omoitsuki.netlify.app/posts/must-vsce/)で開発した拡張機能に、カーソルの下にある URL を選択する機能を追加しました。

<!--more-->

具体的な例で説明します。

たとえば、`https://xtech.nikkei.com/atcl/nxt/column/18/00849/00081/?n_cid=nbpnxt_mled_itmh` という URL の上にカーソルがあると気に、コマンドパレットから`Must: Select URL`コマンドを実行すると、この URL 全体が選択されます。URL からリンクを作成する`Must: Format Linkfrom URL`を実行する準備ができたことになります。

リポジトリ、導入環境、導入手順は上記のリンク先と同じなので省略します。

## 利用方法

1. 選択したい URL の上にカーソルを移動します。

1. コマンドパレットから Must: Select URL を選択して、実行します。

1. URL が選択されます。

## 設定方法

settings.json などに設定が可能です。

- URL の正規表現 (must-vscode.urlRegex)

選択する URL の正規表現表現を指定します。デフォルトでは、以下の正規表現が登録されています。

```json:settings.json
{
  "description": "The regular expression of URL.",
  "type": "string",
  "default": "https?:\\/\\/[\\w\\/:%#\\$&\\?~\\.=\\+\\-]+"
}
```
