---
title: IPアドレスの履歴を表示する機能を追加しました
# date: 2020-05-14 15:25:15
tags:
  - glipah
  - Netlify
  - Vue.js
  - Jest
---

[GLIPAH: Global IP Address History](https://glipah.netlify.app/)に、これまでに取得したIPアドレスの履歴を表示する機能を追加しました。

<img src="{% post_path glipah110 %}Screenshot_2020-05-14GLIPAH.png" alt="GLIPAHの画面" style="border:1px solid #000000;" />

履歴を確認できることで、プロバイダから割り当てられているIPアドレスが変更されたことに気がつきやすくなります。

取得したIPアドレスの履歴は、IndexedDB APIを利用してwebブラウザが動作しているクライアント環境に保存します。サーバー環境にはIPアドレスを保存しません。

これまでは、webページにアクセスしたタイミングでIPアドレスを取得していたが、確認ボタンをクリックするタイミングで取得することに変更しました。

## IPアドレスの履歴の保存

IPアドレスの履歴をIndexedDBに保存することにしたため、これまでと処理の流れを変更しました。これまでは、以下の通りです。

<img src="{% post_path glipah110 %}システム構成V1.0.0.png" alt="GLIPAH V1.0.0の処理の流れ" style="border:1px solid #000000;" />
