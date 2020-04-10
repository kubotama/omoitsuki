---
title: Google Search Consoleに登録する
date: 2020-04-10 15:19:48
tags:
---

このブログサイトををGoogle Search Consoleに登録する。このブログサイトは<https://omoitsuki.netlify.com/>と<https://omoitsuki.netlify.app/>の2つのURLがあるため、それぞれを登録する。あらかじめGoogle Analyticsに登録しておいたため、Search Consoleでプロパティを追加して、URLプレフィックスでそれぞれのURLを入力すると自動的に確認された。

次にサイトマップを登録する。サイトマップを登録するには、ブログサイトをHexoで生成するときに、自動的にサイトマップを生成して、生成したサイトマップを参照するようにGoogle Search Consoleを設定する。

サイトマップを自動的に生成するために、`yarn add hexo-generator-sitemap`でhexo-generator-sitemapをプロジェクトに追加する。_config.ymlに以下を追加して、ブログサイトを生成するときにサイトマップも生成するように設定する。

```yml
sitemap:
    path: sitemap.xml
```

`yarn run build'を実行して、<http://localhost:4000/sitemap.xml>が生成されていることを確認する。変更した_cofig.ymlを含むコミットを作成して、GitHubのmasterブランチを更新するとNetlifyのビルドが実行されてNetlify環境にもsitemap.xmlが生成される。ただし、Netlify環境で生成されるsitemapは_config.ymlのURLを参照しているようで、<https://omoitsuki.netlify.app>のプロパティのみ、サイトマップを登録できた。
