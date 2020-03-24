---
title: buttonの追加
tags:
  - Vue.js
---

Vue.jsで作成するwebページに、テスト駆動でbuttonを追加をする。

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

```javascript:src/main.js
import Vue from "vue";
import App from "./App.vue";

Vue.config.productionTip = false;

new Vue({
  render: h => h(App)
}).$mount("#app");
```

src/App.vue

```vue:src/App.vue
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

```vue:src/components/SampleButton.vue
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
