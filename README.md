# Node.jsでCloud Functionsを書いてみる。

## ランタイムに関して

>Cloud Functions の Node.js 実行環境は、2016 年 10 月 18 日に公開された v6 LTS 以降の Node「LTS」リリースに従います。Cloud Functions で現在実行されている Node.js バージョンは Node v6.11.5 です。

と書いてあるが、現在は[v8.15.0が利用でき、v6はdeprecated](https://cloud.google.com/functions/docs/concepts/exec#runtimes)。

- [The Node.js 8 Runtime](https://cloud.google.com/functions/docs/concepts/nodejs-8-runtime)

## Cloud Functionsのタイプ

以下の2つのタイプが存在する。

- 「フォアグラウンド」（HTTP）
- 「バックグラウンド」

### HTTP関数

HTTPリクエストから実行する関数。

以下が単純なHTTP関数の例。

```js
/**
 * HTTP Cloud Function.
 *
 * @param {Object} req Cloud Function request context.
 * @param {Object} res Cloud Function response context.
 */
exports.helloHttp = (req, res) => {
  res.send(`Hello ${req.body.name || 'World'}!`);
};
```

### バックグラウンド関数

>Cloud インフラストラクチャのイベント（Google Cloud Pub/Sub トピックのメッセージや Google Cloud Storage バケットの変更など）を処理するには、バックグラウンド関数を使用できます。

```js
/**
 * Background Cloud Function.
 *
 * @param {object} event The Cloud Functions event.
 * @param {function} callback The callback function.
 */
exports.helloBackground = (event, callback) => {
  callback(null, `Hello ${event.data.name || 'World'}!`);
};
```

## デプロイ

`gcloud`コマンドを利用したデプロイをしてみる。

今回デプロイするのは以下の`index.js`

```js
exports.helloGET = (req, res) => {
  res.send('Hello World!');
};
```

`index.js`が存在する階層で以下のコマンドを実行すればデプロイできる（設定しているデフォルトプロジェクトにデプロイされる）。

```bash
gcloud functions deploy helloGET --runtime nodejs8 --trigger-http
```

- `deploy`: 実行するCloud Functionsのコマンド。今回は`deploy`コマンドを指定している。
- `helloGET`: デプロイするCloud Functionsの名前。英数字とハイフンのみ使用できる。エクスポートする関数名はこの名前と同じにする必要がある（`--entry-point`オプションを利用すれば、このオプションとは別の名前を利用できる）。
- `--runtime nodejs8 `: 実行するランタイム。
- `--trigger-http`: デプロイする関数のトリガーの種類。今回場合HTTPリクエストがトリガーとなる。

### デプロイした関数を実行する

HTTPリクエストで確認できるので、`curl`コマンドで確認できる。

```shell
$ curl "https://REGION-PROJECT_ID.cloudfunctions.net/helloGET" 
```

`REGION`はプロジェクトのリージョンで`PROJECT_ID`のIDなので、実際は以下のようなリクエストになる。

```shell
$ curl "https://us-central1-awesome-project.cloudfunctions.net/helloGET" 
Hello World!
```