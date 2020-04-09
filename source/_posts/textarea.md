---
title: テスト駆動開発でtextareaを追加
date: 2020-04-09
tags:
  - Vue.js
  - Jest
---

vue-cliで作成したプロジェクトをベースにした空白のwebページに、Jestを利用したテスト駆動開発でtextareaを追加をする。

- textarea(id=sampleArea)の存在をテスト
- textareaの初期値(='')をテスト
- textareaに入力された文字列の取得をテスト

完成したソースコードは[GitHubのリポジトリ](https://github.com/kubotama/sample_textarea)にある。

## textareaを追加する前のディレクトリ構成とファイル

textareaを追加する前のディレクトリ構成とファイル

```sh
├ public
│  └ index.html
└ src
    ├ App.vue
    ├ components
    │ └ SampleTextArea.vue
    └ main.js
```

public/index.html

```html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>
```

src/main.js

```javascript
import Vue from "vue";
import App from "./App.vue";

Vue.config.productionTip = false;

new Vue({
  render: h => h(App)
}).$mount("#app");
```

src/App.vue

```html
<template>
  <div id="app">
    <SampleTextArea />
  </div>
</template>

<script>
import SampleTextArea from "./components/SampleTextArea.vue";

export default {
  name: "App",
  components: {
    SampleTextArea
  }
};
</script>

<style></style>
```

src/components/SampleTextArea.vue

```html
<template>
  <div></div>
</template>

<script>
export default {
  name: "SampleTextArea"
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
```

## TextAreaの存在を確認するテストを追加

tests/unit/textarea.spec.jsに、textareaの存在を確認するテストを追加する。

```javascript
import { shallowMount } from "@vue/test-utils";
import SampleTextArea from "@/components/SampleTextArea.vue";

describe("SampleTextAreaコンポーネント", () => {
  let wrapper;
  beforeEach(() => {
    wrapper = shallowMount(SampleTextArea);
  });

it("textareaの存在をテストする。", () => {
    expect(wrapper.find("#sampleTextArea").exists()).toBeTruthy();
  });
});
```

textareaにidが割り当てられていないため、テストは失敗する。

```sh
FAIL  tests/unit/textarea.spec.js
  SampleTextAreaコンポーネント
    ✕ textareaの存在をテストする。 (19ms)

  ● SampleTextAreaコンポーネント › textareaの存在をテストする。

    expect(received).toBeTruthy()

    Received: false

      16 |
      17 |   it("textareaの存在をテストする。", () => {
    > 18 |     expect(wrapper.find("#sampleTextArea").exists()).toBeTruthy();
         |                                                      ^
      19 |   });
      20 | });
      21 |
```

src/components/SampleTextArea.vueで、textareaにidを追加することでテストは成功する。

```html
<template>
  <div>
   <button id='sampleTextArea'></button>
  </div>
</template>
```

## textareaの初期値のテストを追加

textareaの初期値のテストは以下の2つを確認する。

- テキストは""
- placeholderは"変換したい文字列を入力してください"
- 行数は10
- 桁数は50

tests/unit/textarea.spec.jsにテキストとplaceholder、行数、桁数を確認するテストを追加する。

```javascript
  const idTa = "#sampleTextArea";
  let wrapper;
  let elTa;

  beforeEach(() => {
    wrapper = shallowMount(SampleTextArea);
    elTa = wrapper.find(idTa).element;
  });

  describe("textareaの初期設定を確認する。", () => {
    it('初期値は""とする。', () => {
      expect(elTa.value).toBe("");
    });

    it("textareaのplaceholderは「変換したい文字列を入力してください」とする。", () => {
      expect(elTa.placeholder).toBe("変換したい文字列を入力してください");
    });

    it("textareaの行を10、桁を50とする。", () => {
      expect(elTa.rows).toBe(10);
      expect(elTa.cols).toBe(50);
    });
  });
```

これらの初期値を設定していないため、テストは失敗する。これらの初期値の設定を追加することでテストが成功する。

```html
<template>
  <div>
    <textarea
      id="sampleTextArea"
      placeholder="変換したい文字列を入力してください"
      rows="10"
      cols="50"
    ></textarea>
  </div>
</template>
```

## textareaに入力された文字列の取得のテストを追加

textareaに入力されたテキストを取得するテストを追加する。テキストの入力は、[vue-test-utils](https://vue-test-utils.vuejs.org/ja/)の[setData](https://vue-test-utils.vuejs.org/ja/api/wrapper/#setdata-data)を利用する。

```javascript
  describe("textareaに入力されたテキストの取得をテストする。", () => {
    it.each`
      testText
      ${"abcdefghijklmnopqrstuvwxyz"}
      ${"これはテストのテキストです。"}
    `("$testText をセット", ({ testText }) => {
      wrapper.setData({ sampleText: testText });
      expect(wrapper.vm.sampleText).toBe(testText);
    });
  });
```

textareaのテキストが変数にバインドされていないため、テストは失敗する。変数を定義して、定義した変数をtextareaにバインドすることでテストに成功する。ここでは両方向のバインドのv-modelを利用する。

```html
<template>
  <div>
    <textarea
      id="sampleTextArea"
      v-model="sampleText"        // 定義した変数をtextareaのテキストに両方向でバインド
      placeholder="変換したい文字列を入力してください"
      rows="10"
      cols="50"
    ></textarea>
  </div>
</template>

<script>
export default {
  name: "SampleTextArea",
  data() {
    return {
      sampleText: ""              // textareaのテキストにバインドする変数を定義
    };
  }
};
</script>
```

以上で、Vue.jsで作成するwebページにテスト駆動でtextareaを追加した。
