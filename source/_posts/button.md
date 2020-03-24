---
title: buttonの追加
tags:
  - Vue.js
---

Vue.jsで作成するwebページにbuttonを追加をするときに、以下のテストを作成する。

- button(id=sampleButton)の存在をテスト
- buttonのラベル(サンプルのボタン)をテスト
- buttonがクリックされて呼び出されるメソッド(onClick)のテスト

## buttonを追加する前のディレクトリ構成とファイル

buttonを追加する前のディレクトリ構成

```sh
└ src
    ├ App.vue
    ├ components
    │ └ SampleButton.vue
    └ main.js
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

```vue
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

```vue
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

```vue
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

src/components/SampleButton.vueを修正(template以外は変更なし)することでテストは成功する。

```vue
<template>
  <div>
   <button id='sampleButton'>サンプルのラベル</button>
  </div>
</template>
```
