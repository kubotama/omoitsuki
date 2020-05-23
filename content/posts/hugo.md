---
title: Hugoへの移行
date: 2020-05-23
tags:
  - Hugo
---

このサイトは、Hexoを利用して構築していましたが、他の静的サイトジェネレーターへの変更を検討した結果、[Hugo](https://gohugo.io/)に移行することに決めました。このエントリーでは、移行の手順について説明します。

<!--more-->

変更を考えた理由は、以下の通りです。

- GitHubからSecurity Alertが送られてきた脆弱性を、私個人の技術力の問題ではあるが、簡単には解決できそうもなかった。
- 作成した文章の見た目とかを確認するために、`yarn run server`で起動したローカル環境では、文章を数回修正すると、正常に表示されなくなる。

## 前提

オペレーティングシステムはUbuntu 18.04です。プロジェクトのディレクトリは$ROOT_DIRとします。

## Hexo環境の削除

.git, .gitignore, .markdownlist.json, .vscode/setting.jsonなどの設定ファイル以外を、すべて削除しました。
もともとGitHubで管理しているため、コンテンツはGitHubから復元することにしました。

## Hugoのインストール

以下のコマンドでHugoをインストールします。

```sh
sudo snap install hugo --channel=extended
```

以下のコマンドでhugoコマンドが実行できることを確認します。

```sh
$ which hugo
/snap/bin/hugo
$ hugo version
Hugo Static Site Generator v0.70.0/extended linux/amd64 BuildDate: 2020-05-13T17:30:34Z
$
```

## サイトの作成

$ROOT_DIRの親ディレクトリで、以下のコマンドを実行して、サイトを作成します。

```sh
hugo new site blog --force
```

## テーマの設定

テーマは[Mainroad](https://github.com/vimux/mainroad/)を利用します。$ROOT_DIR/themesで、以下のコマンドを実行して、サブモジュールとしてクローンします。

```sh
git submodule add https://github.com/vimux/mainroad
```

作成された.gitmoduleとthemes/mainroadをgitにコミットします。

## Hugoの環境設定ファイル(config.toml)を作成

config.tomlを以下のように修正しました。

```yaml
baseURL = "https://omoitsuki.netlify.app/"
languageCode = "ja"
DefaultContentLanguage = "ja"
title = "思いつきを書くブログ"
theme = "mainroad"
googleAnalytics = "UA-107278500-2"

[sitemap]
  changefreq = "monthly"
  priority = 0.5
  filename = "sitemap.xml"

[params]
  highlightColor = "#1133cc"
  toc = true
  post_meta = ["date"]

# メインメニュー
[[Menus.main]]
  Name = "ホーム"
  URL = "/"
[[Menus.main]]
  Name = "投稿一覧"
  URL = "/posts/"

[[Menus.footer]]
  Name = "プライバシーポリシー"
  URL = "/privacy/"

[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true
```

主な修正点は以下の通りです。

| セクション | 属性 | 影響 |
|-----|-----|-----|
| | DefaultContentLanguage | Hugoが生成するメッセージが日本語になります |
| params | toc | 各ページの先頭に目次が作成されます |
| params | post_meta | サマリーおよび各ページに日付が表示されます |
| [Menus.main] | | メインメニューを設定します |
| markup.goldmark.renderer | unsafe | HTMLを処理します |

## Netlifyのビルド設定ファイル(netlify.toml)を更新

netlify.tomlを以下のように修正しました。

```yaml
[build]
publish = "public"
command = "hugo --theme=mainroad --gc --minify"

[context.production.environment]
HUGO_ENV = "production"
```

## コンテンツの復元

$/ROOT_DIR/source/_postsにあったMarkdownファイルと画像ファイルを$ROOT_DIR/content/postsに復元します。

### サマリーの表示

Hugoには、サマリーに表示するテキストを指定できる機能があります。先頭から<!\-\-more\-\->まで表示されます。
各ページの適切な場所に、<!\-\-more\-\->を挿入します。

### 画像の組み込み

画像ファイルは、エントリーのファイル名から拡張子(.md)を取り除いたディレクトリに保存します。
たとえば、glipah110.mdが参照している画像ファイルはglipah110ディレクトリに置きます。

### faviconの復元

favicon.icoを$ROOT_DIR/staticに置きます。

<hr>

以上でHugoへの移行が完了です。
