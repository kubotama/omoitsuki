---
title: Netlify functionsを利用したサーバーレスアプリケーションの開発
date: 2020-04-18
tags:
  - Netlify
  - Vue.js
  - Jest
---

## 概要

Netlify Functionsの設定、およびNetlify Functionsのコード(以下コードと呼ぶ)の作成、コードを呼び出すwebページの作成について説明する。サンプルとして、ボタンをクリックするとコードにアクセスして、コードから返された文字列を表示するwebページを作成した。

作成したwebページのソースコードは[こちらのリポジトリ](https://github.com/kubotama/sample_functions)、webサイトは[こちらのサイト](https://kubotama-sample-functions.netlify.com/)である。

[Netlifyのドキュメント](https://docs.netlify.com/functions/configure-and-deploy/)を参考にして、以下の手順で設定する。

## GitHubのリポジトリを作成する

Netlifyと連携するためのリポジトリを作成する。

## GitHubのリポジトリとNetlifyのサイトを関連付ける

Netlifyのサイトを作成して、GitHubのリポジトリと関連付けて、masterブランチが更新されるとNetlifyのサイトが更新されるように設定する。

## ローカル環境にクローンする

作成したリポジトリを、ローカル環境にクローンする。ここではクローンしたディレクトリをfunctionsとする。

## vueプロジェクトを作成する

コードを呼び出すwebページを作成するために、`vue create functions`を実行する。オプションのテストツールとしてJestを選択する。

## ボタン1つとテキストを表示するwebページを作成する

[テスト駆動開発でbuttonを追加](https://omoitsuki.netlify.com/2020/03/26/button/)を参考にして、ボタン1つとテキストを表示するwebページを作成する。作成したソースコードをローカル環境のgitのリポジトリにコミットして、GitHubにプッシュする。

masterブランチを更新して、関連付けたNetlifyのサイトに、作成したwebページが表示されることを確認する。

## netlify-lambdaをインストールする

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
    data: "sample"
  });
}
```

ここまでで、Netlify Functionsの設定およびコードが作成されている。

### ビルドのエラーを解消する

`yarn build`を実行すると`Unknown browser query 'dead'`というエラーになる。

package.jsonの"browserslist"ブロックから`"not dead"`を削除するとエラーがなくなる。

## コードの呼び出しのテストを作成する

コードの呼び出しのテストを作成する。 テストの要件は以下の通りとする。

- ボタンをクリックされると呼び出される。
- axios.getが呼び出される。
- 引数は、コードのURL(ローカルの場合は<http://localhost:9000/.netlify/functions/sample>)である。
- ボタンがクリックされた後は、テキスト領域に'sample'が設定されている。

describeの前でaxiosをモックにする必要があるため、わかりやすくするため、別ファイル(click.spec.js)を作成する。

```javascript
import axios from "axios";
import { shallowMount } from "@vue/test-utils";
import SampleFunctions from "@/components/SampleFunctions.vue";

jest.mock("axios");
axios.get.mockImplementation(() =>
  Promise.resolve({ statusCode: 200, data: "sample" })
);

describe("コードの呼び出し", () => {
  let wrapper;

  beforeEach(() => {
    wrapper = shallowMount(SampleFunctions);
  });

  it("axios.getを呼び出す。", () => {
    wrapper.find("#sampleButton").trigger("click");

    expect(axios.get.mock.calls.length).toBe(1);
  });

  it("引数を確認する。", () => {
    wrapper.find("#sampleButton").trigger("click");

    expect(axios.get.mock.calls[0].length).toBe(1);
    expect(axios.get.mock.calls[0][0]).toBe(
      "http://localhost:9000/.netlify/functions/sample"
    );
  });

  it("ボタンがクリックされた後は、テキスト領域に'sample'が設定されている。", async () => {
    await wrapper.find("#sampleButton").trigger("click");
    expect(wrapper.vm.sampleText).toBe("sample");
  });
});
```

テストを成功するために、SampleFunctions.vueのonClickメソッドを以下のように修正する。

```javascript
    onClick() {
      axios
        .get("http://localhost:9000/.netlify/functions/sample")
        .then(response => {
          this.sampleText = response.data;
        })
        .catch(error => {
          console.error(error);
        });
    }
```

### コードのURLを取得する

ここまではコードのURLを決め打ちでハードコードしているが、Netlify環境にあわせて、取得する機能を追加する。

コードのURLを返すメソッドをgetFunctionUrlとして、このメソッドのテストを追加する。テストの要件は以下の通りとする。

- 引数としてアクセスしているwebページのURL(location.href)を受け取る。
- ホスト名がlocalhostの場合には、ポート番号を9000とする。
- webページのURLに/.netlify/functions/sampleを追加したURLを返す。

テストするURLは以下の通りとする。

| 引数として渡すURL | 返すべきURL |
| --- | --- |
| <http://localhost/> | <http://localhost:9000/.netlify/functions/sample> |
| <http://localhost:8080> | <http://localhost:9000/.netlify/functions/sample> |
| <https://kubotama-sample-functions.netlify.app/> | <https://kubotama-sample-functions.netlify.app/.netlify/functions/sample> |

click.spec.jsに以下を追加する。

```javascript
describe("コードのURLを取得する。", () => {
  let wrapper;

  beforeEach(() => {
    wrapper = shallowMount(SampleFunctions);
  });

  it.each`
    beforeUrl                                           | afterUrl
    ${"http://localhost/"}                              | ${"http://localhost:9000/.netlify/functions/sample"}
    ${"http://localhost:8080"}                          | ${"http://localhost:9000/.netlify/functions/sample"}
    ${"https://kubotama-sample-functions.netlify.com/"} | ${"https://kubotama-sample-functions.netlify.com/.netlify/functions/sample"}
  `("$before -> $after", ({ beforeUrl, afterUrl }) => {
    expect(wrapper.vm.getFunctionUrl(beforeUrl)).toBe(afterUrl);
  });
});
```

SampleFunctions.vueのonClickメソッドをgetFunctionUrlを呼び出すように書き換える。

```javascript
    onClick() {
      axios
        .get(this.getFunctionUrl(window.location.href))
        .then(response => {
          this.sampleText = response.data;
        })
        .catch(error => {
          console.error(error);
        });
    },
````

## Netlify環境への適用

作成したプログラムをGitのリポジトリにコミットして、GitHubのリポジトリにプッシュする。GitHubのリポジトリとNetlifyのサイトが正しく連携していれば、自動的にNetlifyのサイトが更新されて、自動的にNetlify Functionsが開始される。

Netlifyのサイトを表示して、ボタンをクリックすると、ボタンの下に表示されている「メッセージを表示する場所」が「sample」に変更する。

## (オプション)ローカルでの検証環境でのCORSを回避する

ローカルの検証環境のボタンを表示するwebページのURLは<http://localhost:8080>で、コードのURLは<http://localhost:9090/.netlify/functions/sample>のため、CORS制約に違反する。

sample.jsを以下の様に修正して、検証環境のみAccess-Control-Allow-Originを追加することでCORSを回避する。

```javascript
export function handler(event, context, callback) {
  const data = {
    statusCode: 200,
    body: "sample"
  };
  if (event.headers.host === "localhost:9000") {
    data["headers"] = {
      "Access-Control-Allow-Origin": event.headers.origin
    };
  }
  callback(null, data);
}
```
