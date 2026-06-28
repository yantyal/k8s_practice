### Dockerイメージをビルドする

`Dockerfile`を使ってDockerイメージを作成する。

#### 実行するコマンド

```bash
docker build .\chapter-01\hello-server\ --tag hello-server:1.0
```

#### コマンド全体の意味

`.\chapter-01\hello-server\` ディレクトリをビルドコンテキストとして指定し、その中にある `Dockerfile` をもとに Dockerイメージを作成する。
作成したイメージには `hello-server:1.0` という名前とタグを付ける。

#### オプションの説明

* `--tag hello-server:1.0`

  作成するDockerイメージに名前とタグをつける。
  この例では、イメージ名が `hello-server` 、タグが `1.0`になる。

#### ビルド後の確認

```bash
docker images hello-server
```

### Docker コンテナを起動する

#### 実行するコマンド

```bash
docker run --rm --detach --publish 9080:9080 --name hello-server hello-server:1.0
```

#### 起動確認

```bash
curl http://localhost:9080
```
下記のレスポンスがあればOK
```
Hello World!
```

#### コンテナの停止

```bash
docker stop hello-server
```