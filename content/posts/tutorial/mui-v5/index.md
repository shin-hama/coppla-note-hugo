---
title: 'Material UI v5 を使ってみる'
date: '2021-10-07T09:28:34'
description: ''
tag: 'Web アプリ'
---

こぷらです。
普段から Web アプリを作成するときは、Material UI を使用してます。
このブログでも使おうと思いドキュメントを開いてみたら、先日の 9 月 16 日に version 5.0 への大規模なアップデートが行われたそうです。
早速使ってみようと思ったので、記事でまとめて紹介します。

また、既存プロジェクトでのアップデートも行ったので、その際に行ったことを備忘録代わりにまとめています。
もしこれからバージョンアップを行いたい方がいれば参考にしてみてください。

## 目次

```toc
from-heading: 2
to-heading: 3
```

## Material UI とは

様々なところで紹介され尽くしていますが、かんたんに説明を。

Material UI は `React` ベースの UI ライブラリー。
UI ライブラリーを使用することで、細かいスタイルを定義せずに整った見た目の UI が作成できる。
UI ライブラリーは様々あるが、特に Material UI は Google が提案する Material Design に則ったシンプルで洗練されたデザインが特徴である。

## インストールと使い方の基本

Node.js などのインストールはすでに完了しているものとします。
以下コマンドで関連ライブラリをインストール可能。

```shell
yarn add @mui/material @emotion/react @emotion/styled
```

Material UI をつかときは、ソースコード内でスタイリングされたコンポーネントを呼び出します。
例えばボタンを使いたい場合は、`import Button from @mui/material/Button` を呼び出します。

<iframe src="https://codesandbox.io/embed/materialuisample-74mud?fontsize=14&hidenavigation=1&module=%2Fsrc%2FButton.js&moduleview=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="MaterialUISample"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>
```shell
export default function App() {
  return <Button variant="contained">Hello Button</Button>;
}
```

ボタン以外にも多数のコンポーネントがありますが、数が多すぎるので紹介はしません。
各種コンポーネントの使い方からサンプルプロジェクトの紹介まで幅広く網羅されている[ドキュメント](https://mui.com/getting-started/usage/)がありますので、そちらを参照してください。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://mui.com/" data-iframely-url="//cdn.iframe.ly/mIkymCk?card=small"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

## v4 からの移行

新規のプロジェクトであればインストールするだけで良いですが、既存のプロジェクトをアップデートする場合はそうは行きません。
特に今回のアップデートでは Style の適用方法が大きく変わっており、自分の場合は Material UI を使うすべてのファイルを修正しました。
個人的には以前よりもわかりやすい形になり満足しています。

以降では、アップデートに伴って行った修正をまとめます。

### パッケージのインストール

v5 環境で v4 の要素を利用するためのパッケージがあるので、そちらをインストールします。

```shell
yarn add @mui/styles
```

その他アイコンや lab 機能を使っていた場合は以下もインストールします。

```shell
yarn add @mui/lab @mui/icons-material
```

### 移行スクリプトの実行

バージョンアップのためにファイルを一つ一つ確認し変更するのは非常に手間です。
そこで、公式から変換スクリプトが提供されているのでそれを活用しましょう。

[ドキュメント](https://mui.com/guides/migration-v4/#run-codemods) では 3 つのスクリプトを実行するよう提案されてます。
一つづつ確認しましょう。

#### preset-safe

v4 から v5 へのマイグレーションには基本的にこれを使うことになります。
実行方法はこちらです、ドキュメントには各ファイルに対して 1 回だけ実行するように書かれているため、注意してください。

```shell
npx @mui/codemod v5.0.0/preset-safe <path|folder>
```

実行すると、既存の v4 用コードの大部分が v5 用に変更されます。
自分の環境では以下の変更が行われました。

- `@material-ui/core` -> `@mui/material`
- `@material-ui/core/styles` -> `@mui/styles`
- `adaptV4Theme` モジュールでテーマ用オブジェクトをラップ
- Light Mode / Dark Mode の切り替えを `theme.palette.type` から `theme.palette.mode` に変更
- デフォルトサイズの `IconButton` を `size="large"` に変更

かなり長いですが、[こちらのページ](https://github.com/mui-org/material-ui/blob/master/packages/mui-codemod/README.md#-preset-safe)に変換の詳細が記載されています。
これを見ると、自分の環境はごく一部の変換だけですんでいるようでした。

#### variant-prop

`<TextField/>, <FormControl/>, <Select/>` 用のもので、`variant` props が定義されていない場合に `variant="standard"` を付与します。
ただし注意点として、テーマ上で TextField の `variant` のデフォルト値を `outlined` にしている場合に限り、**スクリプトを実行してはいけません。**

この変換が必要な理由は TextField の `variant` のデフォルト値が `standard` から `outlined` に変更されたためです。
そのため、Theme でデフォルト値を `outlined` に変えていた場合は、変化がないので実行する必要はなくなります。

```shell
npx @mui/codemod v5.0.0/variant-prop <path>
```

#### link-underline-hover

`<Link/>` コンポーネントの `underline` props が定義されていない場合に `underline="hover"` を付与します。
ただし注意点として、テーマ上で `variant` のデフォルト値を `always` にしている場合に限り、**スクリプトを実行してはいけません。**

[variant-prop](#variant-prop) のときと同じで、`underline` のデフォルト値が `hover` から `always` に変わったために必要な変換です。
自身の環境で本当に必要なのかよく確認してから実行しましょう。

```shell
npx @mui/codemod v5.0.0/link-underline-hover <path>
```

ここまでで変換作業は一通り終わりです。
エラーなくできていれば、すぐにアプリケーションを起動することができるでしょう。

ただし、現状だと v4 のレガシーな作法が混ざってしまっているので、それらを解消します。

### ThemeProvider のアップデート

スクリプト実行後のテーマは、以下のような形になっているはずです。

```javascript
import { createTheme, adaptV4Theme } from '@mui/material/styles';
const theme = createTheme(adaptV4Theme({
    // v4 theme
}));
```

これは、v4 相当のテーマオブジェクトを v5 用に変換している形になっています。
そこで、`adaptV4Theme` を削除して v5 用のテーマを作成しましょう。

- ダークモード使用のためのキーが `theme.palette.type` から `theme.palette.mode` に変更されたので、テーマ上も変更します。

```javascript
 import { createTheme } from '@mui/material/styles';
-const theme = createTheme({palette: { type: 'dark' }}),
+const theme = createTheme({palette: { mode: 'dark' }}),
```

- `theme.props` および `theme.overrides` を廃止し `theme.components` に統一

```shell
 import { createTheme } from '@mui/material/styles';
 const theme = createTheme({
-  props: {
-    MuiButton: {
-      disableRipple: true,
-    },
-  },
-  overrides: {
-    MuiButton: {
-      root: { padding: 0 },
-    },
-  },
+  components: {
+    MuiButton: {
+      defaultProps: {
+        disableRipple: true,
+      },
+      styleOverrides: {
+        root: { padding: 0 },
+      },
+    },
+  },
 });
```

テーマの変更は以上です。

### makeStyles・withStyles

コンポーネントへのスタイルの適用方法も大きく変わっているため、修正が必要です。
スクリプト実行後は、`makeStyles` を使っていた箇所で以下のようになっているはずです。
(`withStyles` も同様)

```shell
-import { makeStyles } from '@material-ui/styles';
+import { makeStyles } from '@mui/styles';
```

これはコア機能から外されたレガシーな手法を流用するための応急措置です。
v5 の作法でスタイルを適用するには主に2つの方法があります。

#### sx property

1 つ目は各コンポーネントに追加された `sx` プロパティを使うものです。
`sx` プロパティに css を模したオブジェクトを渡すことで、そのコンポーネントのスタイルを上書きできます。
以下は使用例です。
ボタンの色を変更したコンポーネントを並べて表示しています。

<iframe src="https://codesandbox.io/embed/materialuisample-74mud?fontsize=14&hidenavigation=1&module=%2Fsrc%2FButtonWithsx.js&moduleview=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="MaterialUISample"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

```javascript
<Button
  variant="contained"
  sx={{
    color: "white",
    backgroundColor: "red",
    "&:hover": {
      backgroundColor: "crimson"
    }
  }}
>
  Hello Styled Button
</Button>
```

#### Styled Component

2 つ目の方法は、あるコンポーネントをベースにして、独自のスタイルを適用した新しいコンポーネントを作る方法です。
ドキュメントでは `Styled Component` と呼ばれています。
使い方は Utility 関数の `styled` をインポートして、引数にベースのコンポーネントを指定します。
そして、初期化子として適用したいスタイルを与えることで、返り値に `Styled Component` を受け取れます。
使い心地は `withStyles` に似ています。
以下は使用例です。
`sx` プロパティと同じものを `Styled Component` で実装しました。

<iframe src="https://codesandbox.io/embed/materialuisample-74mud?fontsize=14&hidenavigation=1&module=%2Fsrc%2FStyledButton.js&moduleview=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="MaterialUISample"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

```javascript
import { styled } from "@mui/material/styles";
const StyledButton = styled(Button)(({ theme }) => ({
  color: "white",
  backgroundColor: "red",
  "&:hover": {
    backgroundColor: "crimson"
  }
}));
```

`sx` プロパティを使えば、その場限りの変更を気軽に実装できます。
逆に同じ見た目を使いまわしたいときや、変更量が多いときなどは `Styled Component` を使うのが良いでしょう。
これらの方法に従って、`makeStyles, withStyles` の使用箇所を修正すればアップデート前の挙動を v5 環境で再現できるようになるはずです。
一通り修正が終わったら、不要なパッケージを削除しておきましょう。

```shell
yarn remove @mui/styles @material-ui/core
```

## 気になる追加機能

アップデートも完了したところで、v5 で追加された新機能の中から気になったものをいくつかピックアップしてみます。
コードの実装方法については[ドキュメント](https://mui.com/)で詳しく紹介されているので、ここでは省略します。

### [Stack](https://mui.com/components/stack/)

<iframe src="https://codesandbox.io/embed/materialuisample-74mud?fontsize=14&hidenavigation=1&module=%2Fsrc%2FStack.js&moduleview=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="MaterialUISample"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

```javascript
<Stack>
  <Typography>item 1</Typography>
  <Typography>item 2</Typography>
  <Typography>item 3</Typography>
</Stack>
```

要素を縦 or 横に並べて表示する機能です。
シンプルな機能ですが、きれいに並べるのは地味に面倒な作業だったため、今後は頻繁に使うことになりそうです。

### [Pagination](https://mui.com/components/pagination/)

<iframe src="https://codesandbox.io/embed/materialuisample-74mud?fontsize=14&hidenavigation=1&module=%2Fsrc%2FPagination.js&moduleview=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="MaterialUISample"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

```javascript
<Pagination count={10} />
```

ページング機能をサポートするための機能です。
よくある次/前のページボタンと現在のページがわかるようになります。
表示範囲を自動で調整したり、見た目もいじりやすいので、自分で作るよりもはるかにいいものができそうです。

### [Masonry](https://mui.com/components/masonry/)

<iframe src="https://codesandbox.io/embed/materialuisample-74mud?fontsize=14&hidenavigation=1&module=%2Fsrc%2FMasonry.js&moduleview=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="MaterialUISample"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

```javascript
<Masonry columns={4} spacing={1}>
  {heights.map((height, index) => (
    <MasonryItem key={index}>
      <Item sx={{ height }}>{index + 1}</Item>
    </MasonryItem>
  ))}
</Masonry>
```

高さの異なるアイテムを隙間なく並べるコンポーネントです。
これを使うだけでかなりおしゃれに見えそうですね。
こちらは Lab 機能のため、追加のパッケージインストールが必要です。

```shell
yarn add @mui/lab
```

## まとめ

今回は Material UI を v4 から v5 へ更新する方法について紹介しました。
変更点が非常に多いのでそれなりに面倒な作業が必要ですが、その分使いやすく便利な機能が増えていました。
アップデートを検討している方がいれば参考にしてみてください。
それでは。
