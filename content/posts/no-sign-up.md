---
title: "Firebase Authenticationのメールアドレス認証で自動サインアップを禁止する"
date: 2022-07-09
tags:
  - React
  - Firebase
---

## 概要

[firebase/firebaseui-web-react: React Wrapper for firebaseUI Web](https://github.com/firebase/firebaseui-web-react) パッケージを
利用すると Firebase Authentication による認証機能を web サイトに簡単に組み込むことができます。
デフォルトではサインアップが可能ですが、サインアップを禁止するオプションもあります。

<!--more-->

## リポジトリ

[kubotama/sample-firebase](https://github.com/kubotama/sample-firebase)

## 前提条件

Next.js でサイトを構築しています。

- node.js: 16.15.1
- Next.js: 12.2.0

詳しくは上記リポジトリで確認して下さい。

## 事前の準備

1. [Firebase](https://console.firebase.google.com/) でプロジェクトを作成します。
1. 作成したプロジェクトで Authentication を開始します。
1. 開始した Authentication でメール/パスワードプロバイダを有効にします。
1. Authentication にユーザーを追加します。
1. 上記リポジトリをローカル環境にクローンします。
1. 作成したプロジェクトにウェブアプリを追加して、「SDK の設定と構成」で「構成」を選択すると表示される「アプリのキーと ID が含まれている Firebase 構成オブジェクト:」をコピーして、ローカル環境にクローンしたリポジトリの./lib/firebase.js に貼り付けます。
1. firebaseConfig の変数宣言の先頭に export を追加します。

## プログラムの説明

ログインしていなければログイン画面(login.tsx)、ログインしていればメイン画面(index.tsx)が表示されます。

1. 上記リポジトリのディレクトリで`yarn dev`を実行すると、ローカル環境で検証用の web サイトが立ち上がりますので、<http://localhost:3000> にアクセスすると、ログインしていませんのでログイン画面が表示されます。

1. 事前の準備で Authentication に追加したユーザーのメールアドレスとパスワードを入力してログインすると、メイン画面(index.tsx)にログインしたユーザーのメールアドレスが表示されます。

1. 追加したユーザーとは違うメールアドレスを入力すると、**Not Authorized** **入力したメールアドレス** is not authorized to view the requested page. と表示されます。

サインアップを禁止するためには、StyledFirebaseAuth にパラメータとして引き渡す uiConfig で signInOptions の disableSignUp の status 属性に true を設定します。

詳しくは、[FirebaseUI for Web — Auth](https://github.com/firebase/firebaseui-web#configure-email-provider)の Configure Email Provider のあたりを参照して下さい。

```tsx:pages/login.tsx
  const uiConfig: firebaseui.auth.Config = {
    signInFlow: "redirect",
    signInOptions: [
      {
        provider: EmailAuthProvider.PROVIDER_ID,
        disableSignUp: { status: true },
      },
    ],
    signInSuccessUrl: "/",
  };
```

## 注意点

上記の設定で UI からのサイプアップは禁止できるが、REST API 経由でのサイプアップは禁止できないとのこと。

> Note that this flag will only disable sign up from the UI and will not prevent sign up via REST API. It is highly recommended that Identity Platform projects enforce this policy via one of these 2 mechanisms:
>
> - Blocking functions: Set a beforeCreate trigger to disable sign up for email providers.
> - In the Cloud Console / Settings / USERS tab, uncheck Enable create (sign-up) checkbox. Though for this setting, sign up for all providers will be disabled.

勧められている Identity Platform は有料プランが前提のため未確認です。

## 追記

有料プランに切り替えて検証したところ、以下の手順で UI からのサインアップが禁止できることを確認しました。

1. Identity Platform を有効にします。この時点で有料プランへの切り替えが必要です。

1. Identigy Platform の設定画面を開きます。

1. ユーザータブにあるユーザーアクションの「作成を可能にする（登録する）」チェックボックスのチェックを外すと、ユーザーの登録ができなくなります。具体的には、メールアドレス、氏名、パスワードを入力した後に、

> Firebase: This operation is restricted to administrators only. (auth/admin-restricted-operation)

というメッセージが表示されてユーザーを登録できません。
