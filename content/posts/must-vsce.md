---
title: "マークアップ言語のリンク作成を支援するVSCodeの拡張機能"
date: 2022-06-25
tags:
  - VSCode
  - Markdown
---

## 概要

Markdown などのマークアップ言語で Web ページへのリンクを作る場合、以下の手順で作ります。

1. リンクを作る URL を決める。
1. URL の Web ページを開く。
1. 開いた Web ページから F12 でソースコードを表示する。
1. title タグを検索して、タイトル文字列を選択、コピーする。
1. 最初に決めた URL とコピーしたタイトル文字列でリンクを作る。

これらの手順+α に対応した VSCode の拡張機能を開発しました。+α とは以下の機能です。

1. タイトルの前後のサイト名などを削除する。
1. URL に追加されているアクセス経路の集計などのためのフラグなどを削除する。

<!--more-->

具体的な例で説明します。

たとえば、`https://xtech.nikkei.com/atcl/nxt/column/18/00849/00081/?n_cid=nbpnxt_mled_itmh` という URL を選択して、コマンドパレットから`Must: Format Link from URL`コマンドを実行すると、選択されている URL は以下の文字列に置き替えられます。

```markdown
[「IE 終了でも IE モードで大丈夫」は本当か、ブラウザー移行の先送りで訪れる悲劇](https://xtech.nikkei.com/atcl/nxt/column/18/00849/00081/)
```

### 結果

> [「IE 終了でも IE モードで大丈夫」は本当か、ブラウザー移行の先送りで訪れる悲劇](https://xtech.nikkei.com/atcl/nxt/column/18/00849/00081/)

### +α 機能その 1: URL の整形

この URL の?以降はなくても同じ web ページにアクセスできるようなので、リンクを作成する URL 文字列を`https://xtech.nikkei.com/atcl/nxt/column/18/00849/00081/`とします。

### +α 機能その 2: タイトルの整形

この URL のソースコードを見て、`<title></title>`タグから、タイトルが` 「IE 終了でも IE モードで大丈夫」は本当か、ブラウザー移行の先送りで訪れる悲劇 | 日経クロステック（xTECH）`であることがわかります。

タイトルの最後の` | 日経クロステック（xTECH）`はサイト名であり、リンクの見出しに含めないことにすると、リンクを作成するタイトル文字列は`「IE終了でもIEモードで大丈夫」は本当か、ブラウザー移行の先送りで訪れる悲劇`となります。

URL 文字列とタイトル文字列を組み合わせて、上記のリンクが作成できます。

## リポジトリ

[kubotama/must-vscode: markup language support tool of Visual Studio Code extension](https://github.com/kubotama/must-vscode)

## 導入環境

以下の環境であれば導入可能だと思います。Linux(Ubuntu 20.04.4 LTS)で開発および検証しています。

- node.js: V16.15.1
- VScode: 1.68.1

## 導入手順

1. 上記 GitHub のリポジトリを適当なローカル環境にクローンします。

```sh
$ git clone git@github.com:kubotama/must-vscode.git
```

2. 必要なパッケージをダウンロードします。

```sh
$ yarn
```

3. 拡張機能のパッケージを作成します。

```sh
$ npx vsce package
```

4. 作成されたパッケージをインストールします。拡張機能のサイドバーの右上のメニューから「VSIX からのインストール...」を選択して、作成されたパッケージファイル(must-vscode-X.Y.Z.vsix)をインストールします。

## 利用方法

1. Markdown ファイル(README.md など)でリンクを作成したい URL を選択します。

1. コマンドパレットから Must: Format Link from URL コマンドを選択して、実行します。

1. 選択されている URL がリンクに置き替えられます。

## 設定方法

settings.json などに設定が可能です。

- 言語ごとのリンク形式の指定
- タイトル文字列のフォーマットの指定
- URL のフォーマットの指定

### 言語ごとのリンク形式の指定 (Must-vscode: Link Formats: must-vscode.linkFormats)

言語ごとにリンク形式を指定します。デフォルトでは Markdown のリンク形式が指定されています。

```json:settings.json
{
  "languageId": "markdown",
  "format": "[${title}](${url})"
}
```

languageId 属性は [Visual Studio Code language identifiers](https://code.visualstudio.com/docs/languages/identifiers#_known-language-identifiers) で定義されている identifiers を指定します。format 属性で定義されている${title}がタイトル文字列、${url}が URL に置き替えられます。

たとえば以下の指定を追加すると、LaTeX ファイルで適切なリンクが作成できるようになります。

```json:settings.json
{
  "languageId": "latex",
  "format": "\\href{${url}}{${title}}"
}
```

### タイトル文字列のフォーマットの指定 (Must-vscode: Url Patterns: must-vscode.titlePatterns)

タイトル文字列のフォーマットを指定します。デフォルトでは以下のサイトのタイトル文字列のフォーマットが指定されています。

- Qiita
- 日経クロステック
- GitHub

```json:settings.json
{
  "url": "https://qiita.com/",
  "pattern": "(.*) - Qiita",
  "format": "$1"
},
{
  "url": "https://xtech.nikkei.com/.*",
  "pattern": "(.*) \\| 日経クロステック（xTECH）",
  "format": "$1"
},
{
  "url": "https://github.com/.*",
  "pattern": "GitHub - (.*)",
  "format": "$1"
}
```

url 属性と一致する web サイトのタイトル文字列のフォーマットを定義します。replace メソッドを第一引数を pattern 属性、第二引数を format 属性で呼び出して、タイトル文字列を置き替えます。

### URL のフォーマットの指定

URL のフォーマットを指定します。デフォルトでは以下のサイトの URL のフォーマットが指定されています。

-日経クロステック

```json:settings.json
{
  "url": "(https://xtech.nikkei.com/.*)\\?.*",
  "format": "$1"
}
```

url 属性と一致する web サイトの URL 文字列のフォーマットを定義します。replace メソッドを第一引数を url 属性、第二引数を format 属性でで呼び出して、URL 文字列を置き替えます。

## プログラムの説明

### 1. コマンドを登録および設定します

settings.json にコマンドを登録します。command 属性にプログラムから呼び出されるコマンド文字列、title 属性にコマンドパレットに表示される文字列を設定します。

```json:settings.json
{
  "command": "must-vscode.urlToLink",
  "title": "Must: Format Link from URL"
}
```

コマンドが初めて実行されるタイミングで拡張機能をアクティブにするように設定します。

```typescript:src/extension.ts
export const activate = (context: vscode.ExtensionContext) => {
  let disposable = vscode.commands.registerCommand(
      "must-vscode.urlToLink",
  ...
```

### 2. コマンドパレットで Must: Format Link from URL コマンドを実行すると、拡張機能がアクティブになります

### 3. アクティブなエディタの言語を取得します

```typescript:src/extension.ts
import * as vscode from "vscode";

const editor = vscode.window.activeTextEditor;
const languageId = editor.document.languageId;
```

### 4. 設定からリンクのフォーマットを取得します

```typescript:src/extension.ts
const config = vscode.workspace.getConfiguration("must-vscode");
const linkFormats: { languageId: string; format: string }[] | undefined =
  config.get("linkFormats");
const linkFormat = linkFormats.find(
  (linkFormat) => linkFormat.languageId === languageId
);

if (linkFormat) {
  return linkFormat.format;
}
```

### 5. 選択されているテキスト(=URL)を取得します

```typescript:src/extension.ts
const selection = editor.selection;
const selectedText = editor.document.getText(selection);
```

### 6. URL からタイトルを取得します

axios.get で取得した HTML コードを JSDOM に読み込ませて title 属性を取得します

```typescript:src/link.ts
const response = await axios.get(url);
const dom = new JSDOM(response.data);
const title = dom.window.document.title;
```

### 7. タイトルを整形します

titleInfo は、設定から取得したタイトルのフォーマットの配列です。URL に該当するタイトルのフォーマットが見つかれば、そのフォーマットに整形します。なければ、そのままとします。

```typescript:src/link.ts
const pattern = titleInfo.titlePatterns.find((pattern) => {
  const reUrl = new RegExp(pattern.url);
  return reUrl.test(titleInfo.url);
});
if (pattern) {
  const rePattern = new RegExp(pattern.pattern);
  return titleInfo.title.replace(rePattern, pattern.format);
}
```

### 8. URL を整形します

urlPatterns は、設定から取得した URL のフォーマットの配列です。URL に該当するフォーマットが見つかれば、そのフォーマットに整形します。なければ、そのままとします。

```typescript:src/link.ts
const urlPattern = urlPatterns.find((urlPattern) => {
  const reUrl = new RegExp(urlPattern.url);
  return reUrl.test(url);
});
if (urlPattern) {
  const formatUrl = urlPattern.format;
  const reUrl = new RegExp(urlPattern.url);
  const displayUrl = url.replace(reUrl, formatUrl);
  return displayUrl;
}
```

### 9. 言語ごとに定義されているフォーマットでリンクを作成します

```typescript:src/link.ts
const linkText = format
  .replace("${title}", displayTitle)
  .replace("${url}", url);
return linkText;
```

### 10. 作成したリンクで選択されているテキストを置き替えます

```typescript:src/extension.ts
const editor = vscode.window.activeTextEditor;
  if (editor) {
    const selection = editor.selection;
    const selectedText = editor.document.getText(selection);
    if (selectedText) {
      if (text) {
        editor.edit((editBuilder) => {
          editBuilder.replace(selection, text);
        });
      }
    }
  }
```

## まとめ

URL からリンクを作成するという単純な作業を VSCode の拡張機能に任せることができるようになりました。この記事を書くときにも、何度かリンクを作成するときに利用しています。
