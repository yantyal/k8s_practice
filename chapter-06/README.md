# Podのライフサイクル

## Replicaset

* `ReplicaSet` の確認

  `ReplicaSet` の一覧を確認するコマンド

  ```bash
  kubectl get replicaset --namespace default
  ```

  * `replicaset`

    取得対象のリソースです。  
    `ReplicaSet` は、指定した数の `Pod` が常に動いているように管理する仕組みです。

  結果

    ```text
    NAME         DESIRED   CURRENT   READY   AGE
    httpserver   3         3         3       74s
    ```

* `ReplicaSet` の削除

  `httpserver` という名前の `ReplicaSet` を削除するコマンド

  ```bash
  kubectl delete replicaset httpserver --namespace default
  ```

  このコマンドを実行すると、`httpserver` という `ReplicaSet` が削除される。  
  `ReplicaSet` によって管理されていた `Pod` もあわせて削除される。

## Deployment

* `Deployment` の確認

  `Deployment` の一覧を確認するコマンド

  ```bash
  kubectl get deployment --namespace default
  ```

  結果

    ```text
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           19s
    ```

* `Deployment` の再起動

  `Deployment` を再起動するコマンド

  ```bash
  kubectl rollout restart deployment/hello-server
  ```

  * `rollout`

    `Deployment` などの更新や反映状況を操作・確認するためのコマンドです。

  * `restart`

    対象のリソースを再起動するためのコマンドです。

  * `deployment/hello-server`

    再起動対象の `Deployment` を指定しています。

    * `deployment`

      対象のリソースの種類です。

    * `hello-server`

      再起動する `Deployment` の名前です。

  このコマンドを実行すると、`Deployment` が管理している `Pod` が順番に作り直される。

  `Pod` の中で動いているアプリケーションを再起動したい場合や、`ConfigMap` などの変更を反映したい場合に使用する。

  再起動の状況は、次のコマンドで確認できる。

  ```bash
  kubectl rollout status deployment/hello-server
  ```

## Pod

* `Pod` の状態をリアルタイムで確認

  `Pod` の一覧を表示し、その後も状態の変化をリアルタイムで監視するコマンド

  ```bash
  kubectl get pod --watch
  ```

  * `--watch`

    リソースの状態変化を継続して監視するためのオプション。  
    `Pod` の作成中や削除中など、状態が変わる様子を確認できる。
    このコマンドを実行すると、`Pod` の状態が変わるたびに表示が更新される。

## Service

* `Service` の確認

  `hello-server-service` という名前の `Service` の状態を確認するコマンド

  ```bash
  kubectl get service hello-server-service --namespace default
  ```

  結果

  ```text
  NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
  hello-server-service   ClusterIP   10.96.133.33   <none>        8080/TCP   61s
  ```

* `Service` へのポートフォワード

  `hello-server-service` という `Service` に対して、ローカルPCからアクセスできるようにするコマンド

  ```bash
  kubectl port-forward svc/hello-server-service 8080:8080 --namespace default
  ```

  * `svc/hello-server-service`

    接続先の `Service` を指定しています。  
    `svc` は `Service` の省略形です。  
    ここでは `hello-server-service` という名前の `Service` に接続します。

  * `8080:8080`

    左側の `8080` は、ローカルPC側のポート番号です。  
    右側の `8080` は、`Service` 側のポート番号です。

    つまり、ローカルPCで `localhost:8080` にアクセスすると、`Service` の `8080` 番ポートに転送されます。

* `Service` の `Type`

  `Service` は、`Pod` へのアクセス方法を定義するリソースです。

  `Type` を変更することで、どこから `Service` にアクセスできるかが変わります。

  Kubernetesでよく使用される `Type` は次の4種類です。

  * `ClusterIP`

    `ClusterIP` はデフォルトの `Type` です。

    `Kubernetes` クラスタの内部からのみアクセスできます。
    外部のPCやブラウザからは直接アクセスできません。

    主に、フロントエンドからバックエンド、バックエンドからDBなど、
    クラスタ内部の通信で使用されます。

    ```mermaid
    flowchart LR
        User["外部ユーザー"] -.アクセス不可.-> Service

        subgraph Kubernetes Cluster
            Service["Service<br/>ClusterIP"]
            Pod1["Pod"]
            Pod2["Pod"]
        end

        Service --> Pod1
        Service --> Pod2
    ```

  * `NodePort`

    `NodePort` は、各 `Node` にポートを公開する `Type` です。

    `Node` のIPアドレスとポート番号を指定することで、
    クラスタの外部からアクセスできます。

    例えば `Node` のIPが `192.168.1.10`、
    `NodePort` が `30080` の場合は、

    `http://192.168.1.10:30080`

    でアクセスできます。

    ```mermaid
    flowchart LR
        User["外部ユーザー"]
        Node["Node<br/>30080"]
        Service["Service"]
        Pod1["Pod"]
        Pod2["Pod"]

        User --> Node
        Node --> Service
        Service --> Pod1
        Service --> Pod2
    ```

  * `LoadBalancer`

    `LoadBalancer` は、クラウドサービスが提供するロードバランサーを利用する `Type` です。

    `AWS` や `Azure`、`GCP` などでは、
    `Service` を作成すると自動でロードバランサーが作成されます。

    利用者はロードバランサーのIPアドレスやDNS名へアクセスします。

    ```mermaid
    flowchart LR
        User["外部ユーザー"]
        LB["Load Balancer"]
        Service["Service"]
        Pod1["Pod"]
        Pod2["Pod"]

        User --> LB
        LB --> Service
        Service --> Pod1
        Service --> Pod2
    ```

  * `ExternalName`

    `ExternalName` は、クラスタ外部のサービス名へ名前解決する `Type` です。

    `Pod` を経由せず、
    外部のDNS名へ接続します。

    例えば、

    * `database.example.com`
    * `api.example.com`

    のような外部サービスへ接続する場合に利用します。

    ```mermaid
    flowchart LR
        Pod["Pod"]
        Service["Service<br/>ExternalName"]
        DNS["database.example.com"]

        Pod --> Service
        Service --> DNS
    ```

  ### `Service Type` の使い分け

  | `Type` | 外部アクセス | 主な用途 |
  | ------- | ------------ | -------- |
  | `ClusterIP` | × | クラスタ内部通信 |
  | `NodePort` | ○ | 開発・検証環境 |
  | `LoadBalancer` | ○ | 本番環境（クラウド） |
  | `ExternalName` | 外部サービスへ接続 | 外部API・DBとの接続 |

* `Pod` のデバッグ用コンテナを起動（Pod内からアプリケーションの接続確認）

  デバッグ用のコンテナを一時的に追加して、シェルを起動するコマンド
  `curlimages/curl` という `curl` が使えるイメージを利用するため、`Pod` 内から通信確認をしたい場合などに便利

  ```bash
  kubectl debug --stdin --tty hello-server-6cc6b44795-4zgbp --image curlimages/curl --target=hello-server -- sh
  ```

  * `debug`

    `Pod` にデバッグ用の一時的なコンテナを追加するためのコマンドです。

  * `--target=hello-server`

    デバッグ対象のコンテナ名を指定しています。  
    ここでは `Pod` 内の `hello-server` コンテナを対象にしています。

  * `--`

    `kubectl debug` のオプションと、コンテナ内で実行するコマンドを区切るために使用します。

  * `sh`

    デバッグ用コンテナ内で起動するシェルです。

* `Pod` の名前とIPアドレスを確認

  `Pod` の一覧を表示し、`Pod` 名と `Pod` のIPアドレスだけを確認するコマンド

  ```bash
  kubectl get pods -o custom-columns=NAME:.metadata.name,IP:.status.podIP
  ```

* `curl` 用の一時的な `Pod` から通信確認（クラスタ内かつ別Podから接続確認）

  `curlimages/curl` イメージを使って一時的な `Pod` を作成し、`10.244.0.10:8080` に通信できるか確認するコマンド

  ```bash
  kubectl run curl --image curlimages/curl --rm --stdin --tty --restart=Never --command -- curl 10.244.0.10:8080
  ```

  * `run curl`

    `curl` という名前の `Pod` を作成して実行します。

* `Service` の名前と `ClusterIP` を確認

  `Service` の一覧を表示し、`Service` 名と `ClusterIP` だけを確認するコマンド

  ```bash
  kubectl get svc -o custom-columns=NAME:.metadata.name,IP:.spec.clusterIP
  ```

  * `svc`

    取得対象のリソースです。  
    `svc` は `Service` の省略形です。

* `curl` 用の一時的な `Pod` から `Service` に通信確認（クラスタ内かつ別PodからService経由で接続確認）

  `curlimages/curl` イメージを使って一時的な `Pod` を作成し、`10.96.239.45:8080` に通信できるか確認するコマンド

  ```bash
  kubectl run curl --image curlimages/curl --rm --stdin --tty --restart=Never --command -- curl 10.96.239.45:8080
  ```

  * `run curl`

    `curl` という名前の `Pod` を作成して実行します。

## Podの外部から情報を読み込むConfigMap

* `Pod` の外部から情報を読み込む方法

  `Pod` の中で動くアプリケーションは、コンテナイメージの中にすべての設定を入れるのではなく、外部から設定情報を読み込むことがあります。

  代表的な方法は次の2つです。

  * `ConfigMap`

    `ConfigMap` は、アプリケーションの設定値を管理するためのリソースです。

    例えば、次のような値を `Pod` の外部で管理できます。

    * ポート番号
    * 環境名
    * ログレベル
    * アプリケーション設定

    パスワードなどの機密情報ではない設定値を管理するときに使用します。

  * `Secret`

    `Secret` は、パスワードやトークンなどの機密情報を管理するためのリソースです。

    例えば、次のような値を管理できます。

    * パスワード
    * APIキー
    * アクセストークン
    * DB接続情報

    `ConfigMap` と似ていますが、機密情報を扱う場合は `Secret` を使用します。

* `ConfigMap` を環境変数として読み込む

  `ConfigMap` の値を `Pod` の環境変数として読み込むことができます。

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: hello-server-configmap
  data:
    PORT: "8080"
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: hello-server
  spec:
    containers:
      - name: hello-server
        image: blux2/hello-server:1.0
        env:
          - name: PORT
            valueFrom:
              configMapKeyRef:
                name: hello-server-configmap
                key: PORT
  ```

  この例では、`hello-server-configmap` という `ConfigMap` に定義した `PORT` の値を、`Pod` の環境変数 `PORT` として読み込んでいます。

  * `ConfigMap`

    設定情報を管理するためのリソースです。

  * `metadata.name`

    `ConfigMap` の名前です。  
    ここでは `hello-server-configmap` という名前を付けています。

  * `data`

    `ConfigMap` に保存する設定値です。  
    ここでは `PORT: "8080"` を定義しています。

  * `env`

    コンテナに環境変数を設定するための項目です。

  * `name: PORT`

    コンテナ内で使用する環境変数名です。

  * `valueFrom`

    値を直接書くのではなく、別のリソースから取得することを表します。

  * `configMapKeyRef`

    `ConfigMap` から値を取得するための指定です。

  * `name: hello-server-configmap`

    参照する `ConfigMap` の名前です。

  * `key: PORT`

    `ConfigMap` の中から取得するキーです。

* `ConfigMap` をファイルとして読み込む

  `ConfigMap` の値は、環境変数だけでなくファイルとして `Pod` に読み込むこともできます。

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-config
  data:
    application.properties: |
      server.port=8080
      log.level=info
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: hello-server
  spec:
    containers:
      - name: hello-server
        image: blux2/hello-server:1.0
        volumeMounts:
          - name: config-volume
            mountPath: /etc/config
    volumes:
      - name: config-volume
        configMap:
          name: app-config
  ```

  この例では、`app-config` という `ConfigMap` の内容を、`Pod` 内の `/etc/config` にファイルとして配置しています。

  コンテナ内では、次のようなファイルとして参照できます。

  ```text
  /etc/config/application.properties
  ```

  * `volumeMounts`

    コンテナ内のどこにファイルを配置するかを指定します。

  * `mountPath: /etc/config`

    `ConfigMap` の内容を配置するコンテナ内のパスです。

  * `volumes`

    `Pod` で使用するボリュームを定義します。

  * `configMap`

    ボリュームの元データとして `ConfigMap` を使用することを表します。

  * `name: app-config`

    読み込む `ConfigMap` の名前です。

* `Secret` を環境変数として読み込む

  `Secret` の値も、`Pod` の環境変数として読み込むことができます。

  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: db-secret
  type: Opaque
  stringData:
    DB_USER: appuser
    DB_PASSWORD: password123
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: hello-server
  spec:
    containers:
      - name: hello-server
        image: blux2/hello-server:1.0
        env:
          - name: DB_USER
            valueFrom:
              secretKeyRef:
                name: db-secret
                key: DB_USER
          - name: DB_PASSWORD
            valueFrom:
              secretKeyRef:
                name: db-secret
                key: DB_PASSWORD
  ```

  この例では、`db-secret` という `Secret` に定義した `DB_USER` と `DB_PASSWORD` を、`Pod` の環境変数として読み込んでいます。

  * `Secret`

    パスワードなどの機密情報を管理するためのリソースです。

  * `type: Opaque`

    一般的なキーと値の形式で `Secret` を作成する指定です。

  * `stringData`

    文字列として `Secret` の値を定義するための項目です。

  * `secretKeyRef`

    `Secret` から値を取得するための指定です。

  * `name: db-secret`

    参照する `Secret` の名前です。

  * `key: DB_PASSWORD`

    `Secret` の中から取得するキーです。

* `Secret` をファイルとして読み込む

  `Secret` もファイルとして `Pod` に読み込むことができます。

  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: db-secret
  type: Opaque
  stringData:
    username: appuser
    password: password123
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: hello-server
  spec:
    containers:
      - name: hello-server
        image: blux2/hello-server:1.0
        volumeMounts:
          - name: secret-volume
            mountPath: /etc/secret
            readOnly: true
    volumes:
      - name: secret-volume
        secret:
          secretName: db-secret
  ```

  この例では、`db-secret` という `Secret` の内容を、`Pod` 内の `/etc/secret` にファイルとして配置しています。

  コンテナ内では、次のようなファイルとして参照できます。

  ```text
  /etc/secret/username
  /etc/secret/password
  ```

  * `volumeMounts`

    `Secret` をコンテナ内のどこに配置するかを指定します。

  * `mountPath: /etc/secret`

    `Secret` の内容を配置するコンテナ内のパスです。

  * `readOnly: true`

    読み取り専用としてマウントします。  
    `Secret` は基本的にアプリケーションから変更しないため、読み取り専用にします。

  * `volumes`

    `Pod` で使用するボリュームを定義します。

  * `secret`

    ボリュームの元データとして `Secret` を使用することを表します。

  * `secretName: db-secret`

    読み込む `Secret` の名前です。

* 使い分け

  | 方法 | 主な用途 | 例 |
  | ---- | -------- | -- |
  | `ConfigMap` を環境変数で読み込む | 少量の設定値 | `PORT`, `LOG_LEVEL` |
  | `ConfigMap` をファイルで読み込む | 設定ファイル | `application.properties` |
  | `Secret` を環境変数で読み込む | 少量の機密情報 | `DB_USER`, `DB_PASSWORD` |
  | `Secret` をファイルで読み込む | 証明書や複数の機密情報 | `token`, `certificate` |

* 初心者向けの覚え方

  * `ConfigMap`

    パスワードではない設定情報を管理する。

  * `Secret`

    パスワードやAPIキーなど、見せたくない情報を管理する。

  * 環境変数として読み込む方法

    アプリケーションが環境変数から設定値を読む場合に使う。

  * ファイルとして読み込む方法

    アプリケーションが設定ファイルを読む場合に使う。

## JobとCronJob

* `Job` と `CronJob`

  `Job` と `CronJob` は、`Kubernetes` 上で一時的な処理や定期実行する処理を動かすためのリソースです。

  通常の `Deployment` は、アプリケーションを継続的に動かし続けるために使います。

  一方で、`Job` や `CronJob` は、処理が完了したら終了するようなバッチ処理に向いています。

  * `Job`

    `Job` は、一度だけ実行したい処理を管理するリソースです。

    例えば、次のような処理に使います。

    * データ移行
    * ファイル変換
    * 一括更新処理
    * 手動で実行するバッチ処理

    `Job` を作成すると、`Pod` が起動して処理を実行します。

    処理が正常終了すると、`Job` は完了状態になります。

    ```mermaid
    flowchart LR
        User["ユーザー"]
        Job["Job"]
        Pod["Pod"]
        Task["一度だけ実行する処理"]

        User --> Job
        Job --> Pod
        Pod --> Task
        Task --> Done["完了"]
    ```

    `Job` のイメージは、次のような流れです。

    ```text
    Jobを作成
      ↓
    Podが起動
      ↓
    処理を実行
      ↓
    処理が終わる
      ↓
    Jobが完了
    ```

  * `CronJob`

    `CronJob` は、決まった時間や間隔で `Job` を自動作成するリソースです。

    例えば、次のような処理に使います。

    * 毎日深夜に集計する
    * 1時間ごとにデータを取得する
    * 毎週バックアップする
    * 定期的に不要データを削除する

    `CronJob` は、指定したスケジュールに従って `Job` を作成します。

    作成された `Job` が、実際に `Pod` を起動して処理を実行します。

    ```mermaid
    flowchart LR
        CronJob["CronJob<br/>定期実行の設定"]
        Job1["Job"]
        Pod1["Pod"]
        Task1["処理"]

        CronJob --> Job1
        Job1 --> Pod1
        Pod1 --> Task1
        Task1 --> Done1["完了"]
    ```

    `CronJob` のイメージは、次のような流れです。

    ```text
    CronJobを作成
      ↓
    指定した時刻になる
      ↓
    Jobが自動作成される
      ↓
    Podが起動する
      ↓
    処理を実行する
      ↓
    Jobが完了する
    ```

* `Job` と `CronJob` の違い

  | リソース | 実行タイミング | 主な用途 |
  | -------- | -------------- | -------- |
  | `Job` | 一度だけ実行 | 手動バッチ、データ移行、一括処理 |
  | `CronJob` | 定期的に実行 | 日次処理、定期バックアップ、定期集計 |

* `Job` の例

  ```yaml
  apiVersion: batch/v1
  kind: Job
  metadata:
    name: sample-job
  spec:
    template:
      spec:
        restartPolicy: Never
        containers:
          - name: sample-job
            image: busybox
            command: ["sh", "-c", "echo Jobを実行しました"]
  ```

  この `YAML` は、`sample-job` という `Job` を作成します。

  `Pod` が起動し、`echo Jobを実行しました` を実行して終了します。

  * `apiVersion: batch/v1`

    `Job` を作成するための `API` バージョンです。

  * `kind: Job`

    作成するリソースの種類です。

  * `metadata.name`

    `Job` の名前です。

  * `spec.template`

    `Job` が作成する `Pod` の定義です。

  * `restartPolicy: Never`

    `Pod` の処理が終了しても、同じ `Pod` を再起動しない設定です。

  * `containers`

    `Pod` 内で実行するコンテナを定義します。

  * `image: busybox`

    使用するコンテナイメージです。

  * `command`

    コンテナ内で実行するコマンドです。

* `CronJob` の例

  ```yaml
  apiVersion: batch/v1
  kind: CronJob
  metadata:
    name: sample-cronjob
  spec:
    schedule: "*/5 * * * *"
    jobTemplate:
      spec:
        template:
          spec:
            restartPolicy: Never
            containers:
              - name: sample-cronjob
                image: busybox
                command: ["sh", "-c", "echo CronJobを実行しました"]
  ```

  この `YAML` は、`sample-cronjob` という `CronJob` を作成します。

  `*/5 * * * *` は、5分ごとに実行するという意味です。

  `CronJob` が5分ごとに `Job` を作成し、その `Job` が `Pod` を起動して処理を実行します。

  * `apiVersion: batch/v1`

    `CronJob` を作成するための `API` バージョンです。

  * `kind: CronJob`

    作成するリソースの種類です。

  * `metadata.name`

    `CronJob` の名前です。

  * `spec.schedule`

    実行スケジュールです。  
    `cron` 形式で指定します。

  * `jobTemplate`

    定期実行時に作成する `Job` の定義です。

  * `restartPolicy: Never`

    `Pod` の処理が終了しても、同じ `Pod` を再起動しない設定です。

  * `containers`

    `Pod` 内で実行するコンテナを定義します。

* `CronJob` のスケジュール例

  | 書き方 | 意味 |
  | ------ | ---- |
  | `*/5 * * * *` | 5分ごと |
  | `0 * * * *` | 毎時0分 |
  | `0 0 * * *` | 毎日0時 |
  | `0 9 * * 1` | 毎週月曜日の9時 |
  | `0 0 1 * *` | 毎月1日の0時 |

* 初心者向けの覚え方

  * `Job`

    一度だけ実行するバッチ処理。

  * `CronJob`

    決まった時間に繰り返し実行するバッチ処理。

  * `CronJob` は `Job` を作る

    `CronJob` が直接処理を実行するのではなく、スケジュールに従って `Job` を作成します。

  * `Job` が `Pod` を作る

    実際にコンテナを起動して処理を実行するのは `Pod` です。
