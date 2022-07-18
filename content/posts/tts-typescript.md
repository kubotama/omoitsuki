---
title: "Google Cloud Text to Speechでテキストを音声に変換するTypeScriptプログラムを開発した"
date: 2022-07-18
tags:
  - TypeScript
---

## 概要

Google Cloud Text to Speech でテキストを音声に変換する TypeScript プログラムを開発しました。

[Text-to-Speech client libraries  |  Cloud Text-to-Speech Documentation  |  Google Cloud](https://cloud.google.com/text-to-speech/docs/libraries) にあるサンプルは JavaScript なので、一部修正が必要でした。

<!--more-->

## 必要な準備

<!-- - Google Cloud プロジェクトを作成する
- API を有効にする
- API にアクセスするサービスアカウントを作成する -->

1. [Google Cloud Platform](https://console.cloud.google.com/)からプロジェクトを作成、あるいは既存のプロジェクトを選択します。

1. [Cloud Text-to-Speech API](https://console.cloud.google.com/marketplace/product/google/texttospeech.googleapis.com)で
   Text-to-Speech API を有効にします。

1. [サービスアカウント](https://console.cloud.google.com/iam-admin/serviceaccounts)でサービスアカウントを作成します。作成したサービスアカウントのキーをダウンロードしてファイルを作成します。環境変数 GOOGLE_APPLICATION_CREDENTIALS で作成したファイルを指定します。

## プログラムの作成

1. node.js のプロジェクトを作成します。

```sh
$ yarn --init
```

2. 必要なモジュールを追加します。

```sh
$ yarn add @google-cloud/text-to-speech
$ yarn add -D typescript @types/node
```

3. TypeScipr 環境を初期化します。

```sh
$ npx tsc --init
```

4. package.json の script セクションにコンパイルと実行コマンドを追加します。

```sh
"scripts": {
    "start": "node index",
    "prestart": "tsc"
}
```

5, [Text-to-Speech クライアント ライブラリ  |  Cloud Text-to-Speech のドキュメント  |  Google Cloud](https://cloud.google.com/text-to-speech/docs/libraries?hl=ja#using_the_client_library) にあるサンプルプログラムを TypeScript に修正します。

```typescript:index.ts
import fs from "fs";

import { TextToSpeechClient } from "@google-cloud/text-to-speech";
import { google } from "@google-cloud/text-to-speech/build/protos/protos";

const client = new TextToSpeechClient();

const text = "This is sample speech.";

const request: google.cloud.texttospeech.v1.ISynthesizeSpeechRequest = {
  input: { text: text },
  voice: { languageCode: "en-US", ssmlGender: "NEUTRAL" },
  audioConfig: {
    audioEncoding: "MP3",
  },
};

client
  .synthesizeSpeech(request)
  .then(([response]) => {
    if (response.audioContent) {
      fs.writeFileSync("output.mp3", response.audioContent);
    }
  })
  .catch((error) => {
    console.error(error);
  });
```

修正したのは、synthesizeSpeech のパラメータである変数 request の型の指定を追加しました。その型を import も追加しています。

6. 以下のコマンドを実行すると音声が保存された output.mp3 が作成されます。

```sh
$ yarn start
```

## オプション

テキストを音声に変換する API である synthesizeSpeech でオプションを指定することが可能です。オプションの一部は[ISynthesizeSpeechRequest - Documentation](https://googleapis.dev/nodejs/text-to-speech/latest/google.cloud.texttospeech.v1.ISynthesizeSpeechRequest.html)で説明されていますが、説明されていないオプションもいくつかあります。

### voice

#### name

一つは voice 属性の中の name 属性です。[IVoiceSelectionParams - Documentation](https://googleapis.dev/nodejs/text-to-speech/latest/google.cloud.texttospeech.v1.IVoiceSelectionParams.html)には string とだけ書かれていて、具体的にどのような値を設定すればいいのか書かれていません。ここで参考になるのが[Text-to-Speech: Lifelike Speech Synthesis  |  Google Cloud](https://cloud.google.com/text-to-speech)のデモです。Voice name で指定した値が name 属性に設定されます。Voice name のメニュー項目は、Language/locale と Voice type で決まります。たとえば、Language/locale が English (United States)で Voice type が WaveNet の場合、メニュー項目は en-US-Wavenet-A から J の 10 項目です。Voice type が Basic の場合、en-US-standard-A から J の 10 項目です。つまり name 属性で WaveNet 形式か standard 形式かを指定します。

### AudioConfig

[IAudioConfig - Documentation](https://googleapis.dev/nodejs/text-to-speech/latest/google.cloud.texttospeech.v1.IAudioConfig.html)には、AudioConfig 属性の中の speakingRate, pitch, volumeGainDb, sampleRateHertz 属性は number, effectsProfileId 属性は string の配列とだけ書かれています。

属性名から、volumeGainDb は音量、sampleRateHertz はサンプリング周波数に関係があるのではないかと考えますが、詳しい情報は見当たりませんでした。

#### effectsProfileId

effectsProfileId は以下のような値を設定するようです。

| Audio device profile のメニュー項目     | オプションとして設定する値            |
| --------------------------------------- | ------------------------------------- |
| Default                                 | 設定なし                              |
| Smart watch or wearable                 | wearable-class-device                 |
| Smartphone                              | handset-class-device                  |
| Headphones or earbuds                   | headphone-class-device                |
| Small home speaker                      | small-bluetooth-speaker-class-device  |
| Smart home speaker                      | medium-bluetooth-speaker-class-device |
| Home entertainment system or smart TV   | large-home-entertainment-class-device |
| Car speaker                             | large-automotive-class-device         |
| Interactive Voice Response (IVR) system | telephony-class-application           |

#### speakingRate

デモでは speakingRate は 0.25 から 4 の間の値を設定します。変換される音声の速度を設定しています。

#### pitch

pitch は-20 から 20 の値を設定します。音声の高さを設定しています。
