---
title: テスト駆動開発でbuttonを追加
date: 2020-03-27
tags:
  - Vue.js
  - Jest
---

Vue.jsで作成した空白のwebページに、Jestを利用したテスト駆動開発でbuttonを追加をする。

- button(id=sampleButton)の存在をテスト
- buttonのラベル(サンプルのボタン)をテスト
  - buttonのラベルが変数にバインドされていない場合
  - buttonのラベルが変数にバインドされている場合
    - buttonのラベルがバインドされた変数のテスト
- buttonがクリックされて呼び出されるメソッド(onClick)のテスト

完成したソースコードは[GitHubのリポジトリ](https://github.com/kubotama/sample_button)にある。

<!--more-->

## buttonを追加する前のディレクトリ構成とファイル

buttonを追加する前のディレクトリ構成

```sh
├ public
│  └ index.html
└ src
    ├ App.vue
    ├ components
    │ └ SampleButton.vue
    └ main.js
```

public/index.html

```html
<!DOCTYPE html>
<html lang="en">
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
    <SampleButton />
  </div>
</template>

<script>
import SampleButton from "./components/SampleButton.vue";

export default {
  name: "App",
  components: {
    SampleButton
  }
};
</script>

<style></style>
```

src/components/SampleButton.vue

```html
<template>
  <div></div>
</template>

<script>
export default {
  name: "SampleButton"
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
```

## buttonの存在を確認するテストを追加

tests/unit/button.spec.js

```javascript
import { shallowMount } from '@vue/test-utils'
import SampleButton from '@/components/SampleButton.vue'

describe('追加するbuttonのidをsampleButtonとする。', () => {
  let wrapper
  beforeEach(() => {
    wrapper = shallowMount(SampleButton)
  })

  it('存在する。', () => {
    expect(wrapper.find('#sampleButton').exists()).toBeTruthy()
  })
})
```

以下のようにエラーになる。

```sh
FAIL  tests/unit/button.spec.js
  追加するbuttonのidをsampleButtonとする。
    ✕ 存在する。 (39ms)

  ● 追加するbuttonのidをsampleButtonとする。 › 存在する。

    expect(received).toBeTruthy()

    Received: false

       9 |
      10 |   it('存在する。', () => {
    > 11 |     expect(wrapper.find('#sampleButton').exists()).toBeTruthy()
         |                                                    ^
      12 |   })
      13 | })
      14 |
```

src/components/SampleButton.vueを修正(template以外は変更なし)することでテストは成功する。

```html
<template>
  <div>
   <button id='sampleButton'></button>
  </div>
</template>
```

## buttonのラベルのテストを追加

tests/unit/button.spec.jsに以下を追加

```javascript
  it('ラベル(サンプルのラベル)が正しい。', () => {
    expect(wrapper.find('#sampleButton').text()).toBe('サンプルのラベル')
  })
```

以下のようにエラーになる。

```sh
FAIL  tests/unit/button.spec.js
  追加するbuttonのidをsampleButtonとする。
    ✓ 存在する。 (23ms)
    ✕ ラベル(サンプルのラベル)が正しい。 (6ms)

  ● 追加するbuttonのidをsampleButtonとする。 › ラベル(サンプルのラベル)が正しい。

    expect(received).toBe(expected) // Object.is equality

    Expected: "サンプルのラベル"
    Received: ""

      13 |
      14 |   it('ラベル(サンプルのラベル)が正しい。', () => {
    > 15 |     expect(wrapper.find('#sampleButton').text()).toBe('サンプルのラベル')
         |                                                  ^
      16 |   })
      17 | })
      18 |
```

### buttonのラベルを変数にバインドしない場合

src/components/SampleButton.vueを修正(template以外は変更なし)することでテストは成功する。

```html
<template>
  <div>
   <button id='sampleButton'>サンプルのラベル</button>
  </div>
</template>
```

### buttonのラベルを変数にバインドする場合

src/components/SampleButton.vueを修正することでテストは成功する。

```html
<template>
  <div>
    <button id="sampleButton">{{ sampleLabel }}</button>
  </div>
</template>

<script>
export default {
  name: "SampleButton",
  data() {
    return {
      sampleLabel: "サンプルのラベル"
    };
  },
  methods: {
    onClick() {
      return;
    }
  }
};
</script>
```

#### buttonのラベルがバインドされた変数のテストを追加

buttonのラベルがバインドされた変数(sampleLabel)のテストを追加する。

```javascript
it('buttonにラベルにバインドされた変数が正しい。', () => {
    expect(wrapper.vm.sampleLabel).toBe('サンプルのラベル')
  })
```

既に正しく変数にバインドされているため、テストは成功する。

## buttonがクリックされて呼び出されるメソッドのテスト

tests/unit/button.spec.jsに以下を追加

```javascript
  it('クリックするとonClickが呼び出される。', () => {
    const onClick = jest.fn()
    wrapper.setMethods({ onClick })
    wrapper.find('#sampleButton').trigger('click')
    expect(onclick).toHaveBeenCalledTimes(1)
  })
```

以下のようにエラーになる。

```sh
FAIL  tests/unit/button.spec.js
  追加するbuttonのidをsampleButtonとする。
    ✓ 存在する。 (22ms)
    ✓ ラベル(サンプルのラベル)が正しい。 (2ms)
    ✕ クリックするとonClickが呼び出される。 (7ms)

  ● 追加するbuttonのidをsampleButtonとする。 › クリックするとonClickが呼び出される。

    TypeError: Cannot set property 'onClick' of undefined

      18 |   it('クリックするとonClickが呼び出される。', () => {
      19 |     const onClick = jest.fn()
    > 20 |     wrapper.setMethods({ onClick })
         |             ^
      21 |     wrapper.find('#sampleButton').trigger('click')
      22 |     expect(onclick).toHaveBeenCalledTimes(1)
      23 |   })
```

src/components/SampleButton.vueを以下のように修正することでテストに成功する。

```html
<template>
  <div>
   <button id='sampleButton' @click="onClick">サンプルのラベル</button>
  </div>
</template>

<script>
export default {
  name: "SampleButton",
  methods: {
    onClick() {
      return
    }
  }
};
</script>
```

以上で、Vue.jsで作成するwebページにテスト駆動でbuttonを追加した。
