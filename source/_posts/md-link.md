---
title: 入力したURLからMarkdownのリンク形式に変換
date: 2020-04-26
tags:
  - must
  - Netlify
---

URLを入力して、Markdownのリンク形式に変換する。たとえば<https://www.google.co.jp/>が入力されたら、\[Google\](<https://www.google.co.jp>)に変換する。表示する見出しは、入力されたURLのwebページのtitleタグから取得する。

webブラウザで実行しているJavaScriptから入力されたURLにアクセスするとCORSとなるため、Netlify Functionsを利用する。
Netlify Functionsのコード(この後は、コードとする)で入力されたタイトルを取得して、リンクを作成する。

[Netlify functionsを利用したサーバーレスアプリケーションの開発](https://omoitsuki.netlify.app/2020/04/17/functions/)を参考にして、
Netlify Functionsのコードと、呼び出すメソッドを作成する。

## netlify-lambdaとnetlify-lambdaパッケージのvue-cliプラグインをインストール

Netlify Functionsをローカル環境でテストするために、以下のコマンドでnetlify-lambdaをインストールする。

```sh
% yarn add netlify-lambda
```

続いて以下のコマンドで、netlify-lambdaパッケージのvue-cliプラグインをインストールする。

```sh
% vue add netlify-lambda
```

プラグインをインストールすると、以下のファイルが生成される。

netlify.toml

```yaml
[build]
  command = "yarn build"
  functions = "lambda"
  publish = "dist"
```

src/lambda/hello.js

```javascript
export function handler(event, context, callback) {
  console.log(event);
  callback(null, {
    statusCode: 200,
    body: JSON.stringify({ msg: "Hello, World!" })
  });
}
```

### コードのURL取得のテストを作成

コードのURLを返すメソッドのテストを追加する。テストの要件は以下の通りとする。

- メソッド名をgetFunctionUrlとする。MustUi.vueに定義する。
- 引数としてアクセスしているwebページのURL(location.href)を受け取る。
- ホスト名がlocalhostの場合には、ポート番号を9000とする。
- webページのURLに/.netlify/functions/titleを追加したURLを返す。

テストするURLは以下の通りとする。

| 引数として渡すURL | 返すべきURL |
| --- | --- |
| <http://localhost:8080> | <http://localhost:9000/.netlify/functions/title> |
| <https://must-kubotama.netlify.app/> | <https://must-kubotama.netlify.app/.netlify/functions/title> |

テストをmd.link.spec.jsに作成する。

```javascript
describe("コードのURLを取得する。", () => {
  let wrapper;

  beforeEach(() => {
    wrapper = shallowMount(MustUi);
  });

  it.each`
    beforeUrl                                           | afterUrl
    ${"http://localhost:8080"}                          | ${"http://localhost:9000/.netlify/functions/title"}
    ${"https://kubotama-sample-functions.netlify.com/"} | ${"https://kubotama-sample-functions.netlify.com/.netlify/functions/title"}
  `("$beforeUrl -> $afterUrl", ({ beforeUrl, afterUrl }) => {
    expect(wrapper.vm.getFunctionUrl(beforeUrl)).toBe(afterUrl);
  });
});
```

## コードのURL取得のテストを成功するメソッドを作成

src/components/MustUi.vueにgetFunctionUrlを作成する。

```javascript
    getFunctionUrl(pageUrl) {
      const url = new URL(pageUrl);
      if (url.hostname === "localhost") {
        url.port = 9000;
      }
      url.pathname = ".netlify/functions/title";
      return url.href;
    }
```

## コードを確認するテストを作成

コードの仕様を以下のとおりとする。

- 名称はtitleとする。URLは<http://サーバー名/.netlify/functions/title>となる。
- GETでアクセスされることを前提とする。タイトルを取得するURLはクエリ文字列として渡される。
- 渡されたURLにGETでアクセスしてタイトルを取得する。
- 取得したタイトルはレスポンスとして返される。
- タイトルが取得できた場合には、ステータスコードは200を返す。
- 渡されたURLのwebページが存在しない場合、あるいは渡されたURLのwebページからタイトルが取得できない場合、ステータスコードは204を返す。

上記の仕様を確認するテストパターンは以下の通りとする。

| URL || ステータスコード | タイトル |
|-----||------------------|----------|
| <http://example.com> || 200 | Example Domain |
| <https://must-kubotama.netlify.app/> || 200 | Markup Support Tool |
| <https://omoitsuki.netlify.app> || 200 | 思いつきを書くブログ |
| || 204 | |
| <http://localhost> || 204 | |

上記を確認するテストを作成する。ファイル名をtitle.node.spec.jsとする。

```javascript
/**
 * @jest-environment node
 */

import axios from "axios";

describe("Netlify Functionsから返されるステータスコードとデータを確認する。", () => {
  it.each`
  url | title | statusCode
  ${"http://example.com"} | ${"Example Domain"} | ${200}
  ${"https://must-kubotama.netlify.app"} | ${"MarkUp Support Tool by netlify"} | ${200}
  ${"https://omoitsuki.netlify.app"} | ${"思いつきを書くブログ"} | ${200}
  ${""} | ${""} | ${204}
  ${"http://localhost"} | ${""} | ${204}
  `("$url", async ({url, title, statusCode}) => {
    let response;
    try {
      response = await axios.get(
        "http://localhost:9000/.netlify/functions/title?url=" + url
      );
    } catch (e) {
      console.error(e);
      expect(false).toBeTruthy();
      return;
    }
    expect(response.status).toBe(statusCode);
    expect(response.data).toBe(title);
  });
})
```

## コードを確認するテストを成功する関数を作成

src/lambda/hello.jsをsrc/lambda/title.jsに変更して、テストを成功する関数を作成する。

```javascript
import cheerio from "cheerio"
import axios from "axios"

export async function handler(event) {
  const returnData = { statusCode: 0, headers: { "Content-Type": "text/plain" } };
  const url = event.queryStringParameters.url;
  console.log(event.headers)
  console.log("url: " + url)
  try {
    //  URLが指定されていない場合にはstatusCodeを204を返す。
    if (url.length == 0) {
      returnData.statusCode = 204
    }
    else {
      const response = await axios.get(url)
      const body = response.data
      const $ = cheerio.load(body)
      const title = $('title').text()
      console.log("title: " + title)
      // タイトルが取得できた場合にはstatusCodeを200を返す。
      returnData.statusCode = 200
      returnData.body = title
      returnData.isBase64Encoded = false

      // テスト環境では、ボタンが表示されているページとNetlify Functionsのポート番号が違うためCORS制約に違反する。
      // CORS制約を回避するためにAccess-Control-Allow-Origin属性を設定する。
      if (event.headers.host === "localhost:9000" && !event.headers["user-agent"].match(/axios/)) {
        if (event.headers.origin) {
          returnData.headers["Access-Control-Allow-Origin"] = event.headers.origin
        }
        else if (event.headers.referer) {
          returnData.headers["Access-Control-Allow-Origin"] = event.headers.referer
        }
      }
    }
  } catch (error) {
    // 指定されたURLにアクセスできない場合にはstatusCodeを204を返す。
    returnData.statusCode = 204;
  }
  console.log(returnData)
  return returnData;
}
```

### CORS違反を回避

Netlify Functionsのローカルでのテスト環境では、webページとは別のポートにアクセスする。そのため、webブラウザ環境のJavaScriptからアクセスするとCORS違反になる。Netlify環境では同じポートになるため、CORS違反はg発生しない。

テスト環境へのアクセスは、以下の3つのパターンがある。

1. テストスクリプトから呼び出される場合
2. localhost:8080でアクセスしたページから呼び出される場合
3. IPアドレスでアクセスしたページから呼び出される場合

2と3はAccess-Control-Allow-Origin属性が適切に定義されていないとCORS違反となる。それぞれの場合のevent.headersの値をまとめる。
2と3はUbuntu 18.04上のChromeでアクセスした場合の値である。

<table style="table-layout: fixed">
  <tr>
    <th class=id-number></th>
    <th class=url>referer</th>
    <th class=url>origin</th>
    <th>user-agent</th>
  </tr>
  <tr>
    <td>1</td>
    <td>未定義</td>
    <td>未定義</td>
    <td>axios/0.19.2</td>
  <tr>
    <td>2</td>
    <td><a href=http://localhost:8080/>http://localhost:8080/</a></td>
    <td><a href=http://localhost:8080>http://localhost:8080</a></td>
    <td>Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.122 Safari/537.36</td>
  </tr>
  <tr>
    <td>3</td>
    <td>http://IPアドレス:8080/</a></td>
    <td>未定義</td>
    <td>Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.122 Safari/537.36</td>
  </tr>
</table>

<span>
<style>
.id-number{
  width: 5%;
}
.url{
  width: 20%;
}
</style>
</span>

いずれの場合もhostはlocalhost:9000である。

hostがlocalhost:9000でuser-agentがaxios/0.19.2でないときに、Access-Control-Allow-Origin属性を設定する必要があることがわかる。2の場合、refererとoriginでは最後の/の有無の違いがある。Access-Control-Allow-Origin属性にrefererを設定してもCORS違反となるため、hostを設定する。3の場合には、refererを設定することでCORS違反は発生しない。

## ボタンをクリックすると呼び出されるメソッドのテストを作成

ボタンをクリックすると呼び出されるメソッドのテストを作成する。メソッドの要件は以下の通りとする。

- メソッド名はonMdLinkとする。
- テキスト領域になにも入力されていなければ、なにもしないで終了する。つまりテキスト領域は空白のままである。
- テキスト領域に入力された文字列をクエリ文字列として、<http://サーバー名/.netlify/functions/title>にGETでアクセスする。サーバー名はアクセスしているwebページと同じとする。
- ステータスコードが200で返ってきた場合には、テキスト領域をMarkdownのリンク形式に書き換える。
たとえば、<http://example.com/>の場合には、[Example Domain](http://example.com/)となる。
- ステータスコードが204で返ってきた場合には、なにもしないで終了する。つまりテキスト領域はそのままである。

テストでは、実際にwebページにはアクセスしないでモックを利用する。モックが呼び出された回数と引数を確認する。モックからタイトルを返して、テキスト領域が正しく更新されることを確認する。

上記の仕様を確認するテストパターンは以下の通りとする。

| テキスト領域 | モックが呼び出された回数 | モックの引数 | ボタンをクリックした後のテキスト領域 |
|--------------|--------------------------|--------------|--------------------------------------|
| (空白) | 0 | なし | (空白) |
| <http://example.com/> | 1 | <http://localhost:9000/.netlify/functions/title?url=http://example.com/> | \[Example Domain\](<http://example.com/>) |
| <https://must-kubotama.netlify.app/> | 1 | <http://localhost:9000/.netlify/functions/title?url=https://must-kubotama.netlify.app/> | \[Markup Support Tool\](<https://must-kubotama.netlify.app/>) |
| <http://localhost> | 1 | <http://localhost:9000/.netlify/functions/title?url=http://localhost> | <http://localhost> |

上記を確認するテストを作成する。ファイル名をmd.link.spec.jsとする。

```javascript
import axios from 'axios'
import flushPromises from 'flush-promises'
import { shallowMount } from '@vue/test-utils'
import MustUi from '@/components/MustUi.vue'

// モックから返す値をテスト関数の中でセットするためのグローバル変数
let statusCode, title

jest.mock("axios");
axios.get.mockImplementation(() =>
  Promise.resolve({
    status: statusCode, data: title
  })
)

describe("ボタンをクリックすると呼び出されるメソッドのテスト", () => {
  let wrapper;

  beforeEach(() => {
    wrapper = shallowMount(MustUi);
  });

  it.each`
  url | calledTimes | calledArg | outputText | testStatusCode | testTitle
  ${""} | ${0} | ${""} | ${""} | ${204} | ${""}
  ${"http://example.com"} | ${1} | ${"http://localhost:9000/.netlify/functions/title?url=http://example.com"} | ${"[Example Domain](http://example.com)"} | ${200} | ${"Example Domain"}
  ${"https://must-kubotama.netlify.app"} | ${1} | ${"http://localhost:9000/.netlify/functions/title?url=https://must-kubotama.netlify.app"} | ${"[MarkUp Support Tool by netlify](https://must-kubotama.netlify.app)"} | ${200} | ${"MarkUp Support Tool by netlify"}
  ${"http://localhost"} | ${1} | ${"http://localhost:9000/.netlify/functions/title?url=http://localhost"} | ${"http://localhost"} | ${204} | ${""}
  `("$url", async ({ url, calledTimes, calledArg, outputText, testStatusCode, testTitle }) => {
    // モックから返す値をグローバル変数にセットする。
    statusCode = testStatusCode
    title = testTitle
    wrapper.setData({ mustArea: url })
    wrapper.find("#mdLinkButton").trigger("click")
    // mdLinkButton(Markdownのリンク)ボタンをクリックして呼び出されるメソッド(onMdLink)は、非同期処理(axios)を呼び出す。
    // expectの前に非同期処理を終了している必要があるため、ここでflushPromisesを呼び出す。
    await flushPromises()
    expect(axios.get).toBeCalledTimes(calledTimes)

    // calledTimesが0は、モックが呼び出されないため、引数をテストしない。
    if (calledTimes > 0) {
      expect(axios.get).toBeCalledWith(calledArg)
    }
    expect(wrapper.vm.mustArea).toBe(outputText)
  })
})
```

### ボタンをクリックすると呼び出されるメソッドの内部で非同期で処理している場合

Markdownのリンクを作成するには、指定されたURLのwebページのタイトルを取得する必要がある。webページのアクセスするために利用するaxiosは非同期のため、実際に処理が終了する前に、その後のプログラムに進んでしまう。expectでテキストエリアの値を判定するときには、まだタイトルが取得できていないため、テストが失敗してしまう。

expectの前にflushPromisesを実行することで、非同期の処理が完了されるため、正しくテストできる。

### モックが呼び出される回数の初期化

axiosをモックで置き換えて呼び出される回数を確認しているが、デフォルトの設定では呼び出される回数はテストにまたがって累積される。テストごとに初期化するには、jest.config.jsに以下の設定を追加する。

```javascript
module.exports = {
  ...
  clearMocks: true
}
```

## メソッドを確認するテストを成功するメソッドを作成

src/components/MustUi.vueのonMdLinkメソッドを作成する。

```javascript
    async onMdLink() {
      if (this.mustArea.length == 0) {
        return
      }
      const url = this.getFunctionUrl(window.location.href) + "?url=" + this.mustArea
      const res = await axios.get(url)
      if (res.status == 200) {
        this.mustArea = '[' + res.data + "](" + this.mustArea + ")"
      }
    },
```

## Netlify環境への適用

作成したプログラムをGitのリポジトリにコミットして、GitHubのリポジトリにプッシュする。GitHubのリポジトリとNetlifyのサイトが
正しく連携していれば、自動的にNetlifyのサイトが更新されて、自動的にNetlify Functionsが開始される。

更新されたサイトに表示されるテキストエリアに<http://example.com>を入力して、「Markdownのリンク」ボタンをクリックすると、
テキストエリアが\[Example Domain\](<http://example.com>)に更新される。これをコピーしてMarkdownの文書に張り付ければ、Markdownのリンクとなる。

### Netlify環境とローカル環境の違い

Netlify Functionsのコードのreturn、つまりレスポンスのheaders属性を[]にすると、ローカル環境では問題は発生しないが、Netlify環境ではステータスコードが502となってエラーになる。headers属性を{}にすると、問題は発生しない。
