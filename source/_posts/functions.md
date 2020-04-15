---
title: Netlify functionsの設定
tags:
  - Netlify
---

## 目的

vue-cliで作成したプロジェクトをベースにした空白のwebページに、Netlify Functionsにアクセスするメソッドをクリックすろと呼び出すボタンを追加する。

## 作業手順

[Netlifyのドキュメント](https://docs.netlify.com/functions/configure-and-deploy/)を参考にして、以下の手順で設定する。

### GitHubのリポジトリを作成する

Netlifyと連携するためのリポジトリを作成する。ここでは[サンプルのリポジトリ](https://github.com/kubotama/sample_functions)を作成する。

### ローカル環境にクローンする

`git clone git@github.com:kubotama/sample_functions.git functions`で作成したリポジトリを、ローカル環境にクローンする。リポジトリ名が長いのでディレクトリ名をfunctionsとする。

### vueプロジェクトを作成する

`vue create functions`でvueプロジェクトを作成する。

### ボタン1つとテキストを表示するwebページを作成する

[テスト駆動開発でbuttonを追加](https://omoitsuki.netlify.com/2020/03/26/button/)を参考にして、ボタン1つとテキストを表示するwebページを作成する。作成したソースコードをローカル環境のgitのリポジトリにコミットして、GitHubにプッシュする。

### GitHubのリポジトリとNetlifyのサイトを関連付ける

[Netlifyのサイト](https://kubotama-sample-functions.netlify.com/)を作成して、GitHubのリポジトリと関連付けて、masterブランチが更新されるとNetlifyのサイトが更新されるように設定する。

### netlify-lambdaをインストールする

ここからは試行錯誤をするので、間違ったことも含めて詳しく記録する。

以下のコマンドでnetlify-lambdaをインストールする。

```sh
% yarn add netlify-lambda
```

以下のコマンドで、netlify-lambdaパッケージのvue-cliプラグインをインストールする。

```sh
% vue add netlify-lambda
```

プラグインをインストールすると以下のファイルが生成される。

netlify.toml

```toml
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

### axiosをインストールする

Netlify functionsにアクセスするためのHTTPクライアントとして利用するaxiosを以下のコマンドでインストールする。

```sh
% yarn add axios
```

### functionsを確認するテストを追加する

Netlify functionsで実装するAPIの仕様を以下のとおりとする。

- 名称はsampleとする。
- ステータスコードは200を返す。
- "サンプル"という文字列を返す。

これらを確認するテストを作成する。

最初に以下のテストを作成した。

```javascript
it("ステータスコードとデータを確認する。", () => {
  axios
    .get("/.netlify/functions/sample")
    .then(response => {
      expect(response.status).toBe(200);
      expect(response.data).toBe("サンプル");
    });
  });
```

該当するURLが存在しないため失敗するはずが、成功してしまった。axios.getが失敗した場合のcatchがないためと考えて、以下に修正した。

```javascript
it("ステータスコードとデータを確認する。", () => {
  axios
    .get("/.netlify/functions/sample")
    .then(response => {
      expect(response.status).toBe(200);
      expect(response.data).toBe("サンプル");
    })
    .catch(() => {
      expect(false).toBeTruthy();
    });
  });
```

これでもテストに成功した。非同期処理が終了した後にthen, catchが実行されていないためと考えて、以下に修正した。

```javascript
it("ステータスコードとデータを確認する。", async => {
  const res = await axios.get("/.netlify/functions/sample");
  expect(response.status).toBe(200);
  expect(response.data).toBe("サンプル");
});
```

Network Errorが発生した。package.jsonのjestブロックに`"testEnvironment": "node"`を追加したところ、これまで成功していた他のテストもすべて失敗した。

あらためて以下のテストを作成した。catchを実行したときに失敗する適切なjestのmatchが見当たらないため、強引に失敗させている。

```javascript
it("ステータスコードとデータを確認する。", done => {
  axios
    .get("/.netlify/functions/sample")
    .then(response => {
      expect(response.status).toBe(200);
      expect(response.data).toBe("サンプル");
      done();
    })
    .catch(() => {
      expect(false).toBeTruthy();
      done();
    });
  });
```

意図した通り、catchで失敗するようになった。

### ローカル環境でサーバーを立ち上げる

以下のコマンドをローカル環境でサーバーを立ち上げる。

```bash
% netlify-lambda serve src/lambda
```

<http://localhost:9000/.netlify/functions/hello>にwebブラウズでアクセスすると、`{"msg":"Hello, World!"}`が表示されることを確認する。

### テストを実行する

hello.jsをsample.jsに変更して、 仕様にあわせて以下のように修正する。

```javascript
export function handler(event, context, callback) {
  console.log(event);
  callback(null, {
    statusCode: 200,
    body: "サンプル"
  });
}
```

テストを実行するとError: Cross origin http://localhost forbiddenで失敗する。jestの実行環境がjsdomのため、CORSの制限がかかるのが原因らしい。jestの実行環境を"testEnvironment": "node"を設定するとCORSは回避できるが、ブラウザ環境を前提としている他のテストが動かなくなる。

[JESTのドキュメント](https://jestjs.io/docs/en/configuration#testenvironment-string)を参照して、ファイルごとに実行環境を設定できるということなので、button.spec.jsとfunctons.spec.jsに分割して、ファイルの先頭に以下を追加することで、実行環境をnodeに設定する。

```javascript
/**
 * @jest-environment jsdom
 */
```

あらためてテストを実行したが、catchを実行して失敗する。

いろいろと調べた結果、expectの失敗が原因でcatchが実行されていることがわかった。Promiseのthenでexpectを実行して失敗するとcatchが実行されてしまうので、テストを以下のように書き換えた。

```javascript
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

buildを実行すると`Unknown browser query `dead``というエラーになる。

package.jsonの"browserslist"ブロックから`"not dead"`を削除するとエラーがなくなる。

さきにyarn serveを実行するとfunctionsのポートも専有されるが<http://localhost:9000/.netlify/functions/sample>にアクセスするとエラーになる。lsof -i:9000で確認したIDのプロセスをkillした後で、`npx netlify-lambda serve src/lambda`を実行するとfunctionsがローカル環境で起動する。

functionsのメソッドの中で、event.headers['client-ip']で呼び出し元のIPアドレスが取得できるはずであるが、ローカル環境では取得できないバグがあった。2019年7月に修正されているはずだが、反映されていない？
