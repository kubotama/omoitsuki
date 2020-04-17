---
title: Netlify functionsを利用したサーバーレスアプリケーションの開発
tags:
  - Netlify
  - Vue.js
  - Jest
---

## 概要

Netlify Functionsの設定、およびNetlify Functionsのコード(以下コードと呼ぶ)の作成、コードを呼び出すwebページの作成について説明する。

[Netlifyのドキュメント](https://docs.netlify.com/functions/configure-and-deploy/)を参考にして、以下の手順で設定する。

## GitHubのリポジトリを作成する

Netlifyと連携するためのリポジトリを作成する。ここでは[サンプルのリポジトリ](https://github.com/kubotama/sample_functions)を作成した。

## GitHubのリポジトリとNetlifyのサイトを関連付ける

[Netlifyのサイト](https://kubotama-sample-functions.netlify.com/)を作成して、GitHubのリポジトリと関連付けて、masterブランチが更新されるとNetlifyのサイトが更新されるように設定する。

## ローカル環境にクローンする

`git clone git@github.com:kubotama/sample_functions.git functions`で作成したリポジトリを、ローカル環境にクローンする。リポジトリ名が長いのでディレクトリ名をfunctionsとした。

## vueプロジェクトを作成する

コードを呼び出すwebページを作成するために、`vue create functions`を実行する。オプションとして、テストツールとしてJestを選択する。

## ボタン1つとテキストを表示するwebページを作成する

[テスト駆動開発でbuttonを追加](https://omoitsuki.netlify.com/2020/03/26/button/)を参考にして、ボタン1つとテキストを表示するwebページを作成する。作成したソースコードをローカル環境のgitのリポジトリにコミットして、GitHubにプッシュする。

## netlify-lambdaをインストールする

以下のコマンドでnetlify-lambdaをインストールする。

```sh
% yarn add netlify-lambda
```

以下のコマンドで、netlify-lambdaパッケージのvue-cliプラグインをインストールする。

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

## axiosをインストールする

Netlify functionsにアクセスするHTTPクライアントとして、axiosを以下のコマンドでインストールする。

```sh
% yarn add axios
```

## コードを確認するテストを追加する

コードの仕様を以下のとおりとする。

- 名称はsampleとする。URLは<http://サーバー名/.netlify/functions/sample>となる。
- ステータスコードは200を返す。
- "sample"という文字列を返す。

これらを確認するテストを作成する。

```javascript
import axios from "axios";

describe("Netlify functions", () => {
  it("ステータスコードとデータを確認する。", async () => {
    let response;
    try {
      response = await axios.get(
        "http://localhost:9000/.netlify/functions/sample"
      );
    } catch (e) {
      console.error(e);
      return;
    }
    expect(response.status).toBe(200);
    expect(response.data).toBe("sample");
  });
});
```

対象とする[URL](http://localhost:9000/.netlify/functions/sample)が存在しないため、ここではテストは失敗する。

### Jestの実行モード

テストを実行するとError: Cross origin <http://localhost> forbiddenでエラーになる。Jestの実行環境がjsdomモードのため、CORSに違反するためである。nodeモードでJestを実行することでCORSを回避できる。テストモードの切替単位は、全体とファイル単位の2通りがある。他のテストはjsdomモードで実行する必要があり、jsdomモードとnodeモードのテストを共存する必要があるため、ファイル単位での切替とする。

[Jestのドキュメント](https://jestjs.io/docs/en/configuration#testenvironment-string)を参照して、ファイルごとに実行環境を設定するために、jsdomモードで実行するfunctions.spec.jsとnodeモードで実行するlambda.spec.jsに分割して、lambda.jsファイルの先頭に以下を追加することで、実行環境をnodeに設定する。

```javascript
/**
 * @jest-environment jsdom
 */
```

## ローカル環境でfunctionsを起動する

以下のコマンドをローカル環境でfunctionsを起動する。

```bash
% netlify-lambda serve src/lambda
```

<http://localhost:9000/.netlify/functions/hello>にwebブラウズでアクセスすると、`{"msg":"Hello, World!"}`が表示されることを確認する。

## コードをテストする

hello.jsをsample.jsにファイル名を変更して、 仕様にあわせて以下のように修正するとテストが成功する。

```javascript
export function handler(event, context, callback) {
  callback(null, {
    statusCode: 200,
    body: "sample"
  });
}
```

ここまでで、Netlify Functionsの設定およびコードが作成されている。

### ビルドのエラーを解消する

`yarn build`を実行すると`Unknown browser query 'dead'`というエラーになる。

package.jsonの"browserslist"ブロックから`"not dead"`を削除するとエラーがなくなる。

## コードの呼び出しのテストを作成する

コードの呼び出しのテストを作成する。

- ボタンをクリックされると呼び出される。
- axios.getが呼び出される。
- 引数は、コードのURL(ローカルの場合は<http://localhost:9000/.netlify/functions/sample>)である。
- ボタンがクリックされた後は、テキスト領域に"sample"が設定されている。
