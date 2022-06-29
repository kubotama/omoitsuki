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

## プログラムの説明

### 1. コマンドを登録および設定します

settings.json にコマンドを登録します。command 属性にプログラムから呼び出されるコマンド文字列、title 属性にコマンドパレットに表示される文字列を設定します。

```json:settings.json
{
  "command": "must-vscode.selectUrl",
  "title": "Must: select URL"
}
```

コマンドが初めて実行されるタイミングで拡張機能をアクティブにするように設定します。

```typescript:src/extension.ts
export const activate = (context: vscode.ExtensionContext) => {
  let selectDisposable = vscode.commands.registerCommand(
      "must-vscode.selectUrl",
  ...
```

### 2. コマンドパレットで Must: select URL コマンドを実行すると拡張機能がアクティブになります

### 3. 設定から URL の正規表現を取得します

```typescript:src/extension.ts
const getUrlRegex: () => string | undefined = () => {
  const config = vscode.workspace.getConfiguration("must-vscode");
  const urlRegex: string | undefined = config.get("urlRegex");
  return urlRegex;
};
```

### 4. カーソル位置から取得した URL の正規表現に一致する範囲を確認します

```typescript:extension.ts
const selected = editor.document.getWordRangeAtPosition(
  editor.selection.active,
  new RegExp(urlRegex)
  );
```

### 5. 確認した範囲を選択します

```typescript:src/extension.ts
editor.selection = new vscode.Selection(selected.start, selected.end);
```

## まとめ

URL からリンクを作成するための選択を、ほぼ自動でできるようになりました。

## 注意点

URL の後ろに.(ピリオド)など URL に含まれる文字が続いている場合、その文字が URL に含まれるかどうかの判断が難しいため、人間の判断が必要になる場合があります。
