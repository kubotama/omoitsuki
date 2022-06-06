---
title: "Cloud Functions for Firebaseのチュートリアルがビルド時にエラー"
date: 2022-06-06+09:00
---

## まとめ

### 問題点

[Cloud Functions for Firebaseのチュートリアル](https://firebase.google.com/docs/functions/get-started)で[プロジェクトの初期化](https://firebase.google.com/docs/functions/get-started#initialize-your-project)を実行した後にbuildあるいはserveなどを実行するとエラーになる。

### 解決策

@types/node, @types/react, @types/react-domをインストールする。

### 詳細

Next.jsを利用してFirebase Hostingのサイトを開発している途中で、firebase init functionsを実行してCloud Functions for Firebase(CFfF)を追加した。作成されたfunctionsディレクトリでyarn buildあるいはyarn serveを実行すると以下のエラーになる。

```shell
$ yarn build
yarn run v1.22.10
$ tsc
node_modules/@types/react/index.d.ts:3130:14 - error TS2300: Duplicate identifier 'LibraryManagedAttributes'.

3130         type LibraryManagedAttributes<C, P> = C extends React.MemoExoticComponent<infer T> | React.LazyExoticComponent<infer T>
                  ~~~~~~~~~~~~~~~~~~~~~~~~

  ../node_modules/@types/react-dom/node_modules/@types/react/index.d.ts:3127:14
    3127         type LibraryManagedAttributes<C, P> = C extends React.MemoExoticComponent<infer T> | React.LazyExoticComponent<infer T>
                      ~~~~~~~~~~~~~~~~~~~~~~~~
    'LibraryManagedAttributes' was also declared here.

../node_modules/@types/react-dom/node_modules/@types/react/index.d.ts:3127:14 - error TS2300: Duplicate identifier 'LibraryManagedAttributes'.

3127         type LibraryManagedAttributes<C, P> = C extends React.MemoExoticComponent<infer T> | React.LazyExoticComponent<infer T>
                  ~~~~~~~~~~~~~~~~~~~~~~~~

  node_modules/@types/react/index.d.ts:3130:14
    3130         type LibraryManagedAttributes<C, P> = C extends React.MemoExoticComponent<infer T> | React.LazyExoticComponent<infer T>
                      ~~~~~~~~~~~~~~~~~~~~~~~~
    'LibraryManagedAttributes' was also declared here.

../node_modules/@types/react-dom/node_modules/@types/react/index.d.ts:3237:13 - error TS2717: Subsequent property declarations must have the same type.  Property 'table' must be of type 'DetailedHTMLProps<TableHTMLAttributes<HTMLTableElement>, HTMLTableElement>', but here has type 'DetailedHTMLProps<TableHTMLAttributes<HTMLTableElement>, HTMLTableElement>'.

3237             table: React.DetailedHTMLProps<React.TableHTMLAttributes<HTMLTableElement>, HTMLTableElement>;
                 ~~~~~

  node_modules/@types/react/index.d.ts:3240:13
    3240             table: React.DetailedHTMLProps<React.TableHTMLAttributes<HTMLTableElement>, HTMLTableElement>;
                     ~~~~~
    'table' was also declared here.


Found 3 errors in 2 files.

Errors  Files
     1  node_modules/@types/react/index.d.ts:3130
     2  ../node_modules/@types/react-dom/node_modules/@types/react/index.d.ts:3127
error Command failed with exit code 2.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

以下のコマンドを実行してモジュールをインストールすることでエラーが発生しなくなった。

```shell
$ yarn add --dev @types/node @types/react @types/react-dom
```
