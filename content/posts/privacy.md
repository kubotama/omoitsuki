---
title: プライバシーポリシーのページを追加
date: 2020-05-30
---

Googleアナリティクスを利用する場合にはプライバシーポリシーを表示することが義務付けられているため、[GLIPAH](https://glipah.netlify.app/)サイトにプライバシーポリシーのページを追加しました。[GLIPAH](https://glipah.netlify.app/)サイトはページ遷移がないため、ページ遷移のない方法を探したところ、[Vuetify](https://vuetifyjs.com/ja/)のdialogを利用する方法を利用することにしました。

<!--more-->

導入手順は、vuetifyをインストールすると作成されるデフォルトのコンポーネント(HelloWorld.vue)に、これまで作成したコンポーネントを追加して、出来上がったらファイル名をGlipahUi.vueに変更します。

## テストをスキップ

一通りの修正が終わるまではテストが失敗するため、テストを実行しないように、すべてのdescribeをdescribe.skipに書き換えます。

`yarn run test:unit`コマンドを実行して、すべてのテストがスキップされることを確認します。

## Vuetifyをインストール

以下のコマンドでVuetifyをインストールします。

```sh
% vue add vuetify
```

`yarn start`コマンドを実行して、デフォルトのwebページが表示されることを確認します。

## コンポーネントを作成

### 「プライバシーポリシー」ボタンを追加

Vuetifyのサイトの[ダイアログの使い方](https://vuetifyjs.com/ja/components/dialogs/#usage)を参考にして、「プライバシーポリシー」ボタンを作成します。ボタンをクリックするとダイアログでプライバシーポリシーを表示します。ダイアログの「閉じる」ボタンをクリックすると、ダイアログが閉じます。

```html
    <div class="text-right dialog-privacy">
      <v-dialog v-model="dialog" width="500">
        <template v-slot:activator="{ on }">
          <v-btn color="black" dark v-on="on" outlined class="button-privacy">
            プライバシーポリシー
          </v-btn>
        </template>

        <v-card>
          <v-card-title class="headline grey lighten-2" primary-title>
            プライバシーポリシー
          </v-card-title>

          <v-card-text>
            当サイトでは、Googleによるアクセス解析ツール「Googleアナリティクス」を使用しています。このGoogleアナリティクスはデータの収集のためにCookieを使用しています。このデータは匿名で収集されており、個人を特定するものではありません。
            この機能はCookieを無効にすることで収集を拒否することが出来ますので、お使いのブラウザの設定をご確認ください。
            この規約に関しての詳細は<a
              href="https://marketingplatform.google.com/about/analytics/terms/jp/"
              target="”_blank”"
            >
              Googleアナリティクスサービス利用規約のページ </a
            >や
            <a
              href="https://policies.google.com/technologies/ads?hl=ja"
              target="”_blank”"
              >Googleポリシーと規約ページ</a
            >をご覧ください。
          </v-card-text>

          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" text @click="dialog = false">
              閉じる
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>
```

### glipahのボタンとtableを追加

これまでに作成したコンポーネントから確認ボタンとIPアドレスの履歴を表示するtableをコンポーネントに追加します。

#### buttonをv-btnに置き替え(clickイベントをclick.nativeに書き換え)

Vuetifyを追加することでbuttonの見た目が変わってしまうため、buttonをv-btnに置き替えます。
buttonの枠の線がなくなって、立体的に見えなくなります。下の図は、buttonとv-btnを並べていますが、buttonの周囲の枠がありません。

<img src="vuetify-button.png" alt="ボタンの違い" style="border:1px solid #000000;" />

v-btnに置き換えると、以下のテストで失敗します。

```javascript
  it("確認ボタンをクリックして呼び出されるメソッドを確認する", done => {
    const accessFunction = jest.fn();
    wrapper.setMethods({ accessFunction });
    wrapper.find("#buttonClick").trigger("click");
    expect(accessFunction).toHaveBeenCalledTimes(1);
    done();
  });
```

`@click`を`@click.native`に書き換えることで、テストが成功します。

```html
      <v-btn
        @click.native="onButtonClick"
        id="buttonClick"
        class="button-click"
      >
        確認
      </v-btn>
```

### ナビゲーションバーを削除

デフォルトのwebページにあるナビゲーションバーを削除します。

### コンポーネント名を変更

これまでのsrc/components/GlipahUi.vueを削除して、src/components/HelloWorld.vueをsrc/components/GlipahUi.vueにリネームします。src/App.vueでのHelloWorldコンポーネントへの参照をGlipahUiコンポーネントへの参照に書き換えます。

## テストを成功

テストのdescribe.skipをdescribeに戻して、テストを実行します。正しく設定できていれば、テストは成功します。

## Jestで警告が表示されない設定

VuetifyのコンポーネントをJestが認識しないため、テストを実行すると`[Vue warn]: Unknown custom element:`のメッセージが出力されます。以下の設定をすることで、メッセージが表示されなくなります。

jest.config.jsに以下を追加します。

```javascript
module.exports = {
  ...
  setupFiles: ["./tests/unit/setup.js",
  ...
```

tests/unit/setup.jsを作成します。

```javascript
import Vue from "vue";
import Vuetify from "vuetify";

Vue.use(Vuetify);
```

## Vue Devtoolsのダウンロードを勧めるメッセージを表示しないように設定

テストを実行すると以下のメッセージが表示されます。

```Plain Text
Download the Vue Devtools extension for a better development experience:
```

tests/unit/setup.jsに以下を追加することでメッセージが表示されなくなります。

```javascript
Vue.config.devtools = false;
```

## productionモードでデプロイすることを勧めるメッセージを表示しないように設定

テストを実行すると以下のメッセージが表示されます。

```Plain Text
You are running Vue in development mode.
Make sure to turn on production mode when deploying for production.
See more tips at https://vuejs.org/guide/deployment.html
```

tests/unit/setup.jsに以下を追加することでメッセージが表示されなくなります。

```javascript
Vue.config.productionTip = false;
```
