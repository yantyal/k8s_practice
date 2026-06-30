# アプリケーションのヘルスチェック

* ヘルスチェックの3つの仕組み

  `Kubernetes` には、`Pod` やコンテナの状態を確認するためのヘルスチェックがあります。

  主に次の3つがあります。

  * `Readiness probe`
  * `Liveness probe`
  * `Startup probe`

  それぞれ確認する目的が違います。

* `Readiness probe`

  `Readiness probe` は、`Pod` がリクエストを受け付けられる状態かを確認する仕組みです。

  アプリケーションが起動していても、まだ準備が完了していない場合があります。

  例えば、次のような状態です。

  * DB接続の準備中
  * 初期データの読み込み中
  * 外部APIとの接続確認中
  * キャッシュ作成中

  `Readiness probe` に失敗している間、その `Pod` には `Service` 経由の通信が流れません。

  ただし、コンテナ自体は再起動されません。

  ```mermaid
  flowchart LR
      Service["Service"]
      Pod1["Pod<br/>Ready"]
      Pod2["Pod<br/>Not Ready"]

      Service --> Pod1
      Service -.通信しない.-> Pod2
  ```

  * 主な目的

    `Pod` がリクエストを受け付けてよい状態か確認する。

  * 失敗した場合

    `Service` の通信対象から外される。

  * 再起動されるか

    再起動されない。

* `Liveness probe`

  `Liveness probe` は、コンテナが正常に動き続けているかを確認する仕組みです。

  アプリケーションが異常状態になっていて、自力では復旧できない場合に使います。

  例えば、次のような状態です。

  * アプリケーションが固まっている
  * デッドロックしている
  * 処理が停止している
  * エラー状態から復旧できない

  `Liveness probe` に失敗すると、`Kubernetes` はコンテナを再起動します。

  ```mermaid
  flowchart LR
      Kubelet["kubelet"]
      Pod["Pod"]
      Container["Container"]
      Restart["Containerを再起動"]

      Kubelet --> Pod
      Pod --> Container
      Container -->|Liveness probe 失敗| Restart
  ```

  * 主な目的

    コンテナが生きているか確認する。

  * 失敗した場合

    コンテナが再起動される。

  * 再起動されるか

    再起動される。

* `Startup probe`

  `Startup probe` は、アプリケーションの起動が完了したかを確認する仕組みです。

  起動に時間がかかるアプリケーションで使用します。

  例えば、次のような場合です。

  * 起動時に大量のデータを読み込む
  * 初回起動に時間がかかる
  * キャッシュ作成に時間がかかる
  * 古いアプリケーションで起動が遅い

  `Startup probe` が成功するまでは、`Liveness probe` と `Readiness probe` の判定を待たせることができます。

  これにより、起動途中のアプリケーションが `Liveness probe` に失敗して、何度も再起動されることを防げます。

  ```mermaid
  flowchart TD
      Start["Container 起動"]
      Startup["Startup probe"]
      Success["Startup probe 成功"]
      Liveness["Liveness probe 開始"]
      Readiness["Readiness probe 開始"]

      Start --> Startup
      Startup --> Success
      Success --> Liveness
      Success --> Readiness
  ```

  * 主な目的

    アプリケーションの起動完了を確認する。

  * 失敗した場合

    設定した回数を超えて失敗すると、コンテナが再起動される。

  * 再起動されるか

    失敗し続けると再起動される。

* 3つの違い

  | 種類 | 確認すること | 失敗した場合 | 主な用途 |
  | ---- | ------------ | ------------ | -------- |
  | `Readiness probe` | リクエストを受け付けられるか | `Service` の通信対象から外れる | 準備完了前に通信を流さない |
  | `Liveness probe` | 正常に動き続けているか | コンテナを再起動する | 固まったアプリを復旧する |
  | `Startup probe` | 起動が完了したか | 失敗し続けると再起動する | 起動が遅いアプリを守る |

* 初心者向けの覚え方

  * `Readiness probe`

    「準備できた？」を確認する。

  * `Liveness probe`

    「生きている？」を確認する。

  * `Startup probe`

    「起動し終わった？」を確認する。

* 使用イメージ

  ```text
  Container 起動
    ↓
  Startup probe で起動完了を確認
    ↓
  Readiness probe でリクエスト受付可能か確認
    ↓
  Service から通信が流れる
    ↓
  Liveness probe で動き続けているか確認
    ↓
  異常ならコンテナを再起動
  ```

* `YAML` の例

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: hello-server
  spec:
    containers:
      - name: hello-server
        image: blux2/hello-server:1.0
        ports:
          - containerPort: 8080

        startupProbe:
          httpGet:
            path: /health
            port: 8080
          failureThreshold: 30
          periodSeconds: 10

        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10

        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
  ```

  * `startupProbe`

    アプリケーションの起動完了を確認します。

  * `readinessProbe`

    アプリケーションがリクエストを受け付けられる状態か確認します。

  * `livenessProbe`

    アプリケーションが正常に動き続けているか確認します。

  * `httpGet`

    指定したHTTPパスにアクセスして状態を確認します。

  * `path`

    ヘルスチェックでアクセスするパスです。

  * `port`

    ヘルスチェックでアクセスするポート番号です。

  * `initialDelaySeconds`

    コンテナ起動後、最初のチェックまで待つ秒数です。

  * `periodSeconds`

    何秒ごとにチェックするかを指定します。

  * `failureThreshold`

    何回連続で失敗したら失敗扱いにするかを指定します。

## `Pod` のスケジュール

* `Pod` のスケジュールにおける `Affinity` と `Anti-affinity`

  `Affinity` と `Anti-affinity` は、`Pod` をどの `Node` に配置するかを制御するための仕組みです。

  `Kubernetes` では、通常 `Scheduler` が空いている `Node` を選んで `Pod` を配置します。

  しかし、次のような要件がある場合は、`Affinity` や `Anti-affinity` を使います。

  * 特定の条件を持つ `Node` に `Pod` を配置したい
  * 特定の `Pod` と同じ `Node` に配置したい
  * 特定の `Pod` とは別の `Node` に配置したい
  * 障害に備えて `Pod` を分散配置したい

* `Affinity`

  `Affinity` は、条件に合う場所へ `Pod` を配置しやすくする、または必ず配置するための設定です。

  日本語では「親和性」と呼ばれます。

  簡単にいうと、次のような意味です。

  ```text
  この条件に合うNodeやPodの近くに配置したい
  ```

  例えば、次のようなケースで使います。

  * `SSD` を持つ `Node` に配置したい
  * `GPU` を持つ `Node` に配置したい
  * 同じアプリケーションの `Pod` を近くに配置したい
  * キャッシュ用 `Pod` とアプリケーション用 `Pod` を同じ `Node` に配置したい

* `Anti-affinity`

  `Anti-affinity` は、条件に合う場所を避けて `Pod` を配置するための設定です。

  日本語では「反親和性」と呼ばれます。

  簡単にいうと、次のような意味です。

  ```text
  この条件に合うNodeやPodの近くには配置したくない
  ```

  例えば、次のようなケースで使います。

  * 同じアプリケーションの `Pod` を別々の `Node` に分散したい
  * 障害に備えて同じ `Node` に集中させたくない
  * 重い処理をする `Pod` 同士を同じ `Node` に配置したくない
  * 本番用 `Pod` と検証用 `Pod` を同じ `Node` に置きたくない

* `Affinity` と `Anti-affinity` の種類

  `Affinity` と `Anti-affinity` には、大きく分けて次の種類があります。

  | 種類 | 説明 |
  | ---- | ---- |
  | `nodeAffinity` | `Node` のラベルを見て、配置先の `Node` を制御する |
  | `podAffinity` | 他の `Pod` のラベルを見て、近くに配置する |
  | `podAntiAffinity` | 他の `Pod` のラベルを見て、離して配置する |

* `nodeAffinity`

  `nodeAffinity` は、`Node` に付いているラベルを条件にして、`Pod` の配置先を制御します。

  例えば、次のような `Node` ラベルがあるとします。

  ```text
  disktype=ssd
  ```

  この場合、`disktype=ssd` の `Node` に `Pod` を配置したい、という指定ができます。

  ```mermaid
  flowchart LR
      Pod["Pod"]
      Node1["Node A<br/>disktype=ssd"]
      Node2["Node B<br/>disktype=hdd"]

      Pod --> Node1
      Pod -.配置しない.-> Node2
  ```

* `nodeAffinity` の例

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: node-affinity-pod
  spec:
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
            - matchExpressions:
                - key: disktype
                  operator: In
                  values:
                    - ssd
    containers:
      - name: nginx
        image: nginx:1.25.3
  ```

  この `YAML` は、`disktype=ssd` というラベルが付いた `Node` にだけ `Pod` を配置します。

  * `affinity`

    `Pod` の配置ルールを定義する項目です。

  * `nodeAffinity`

    `Node` のラベルを条件にして配置先を制御します。

  * `requiredDuringSchedulingIgnoredDuringExecution`

    条件を必ず満たす `Node` にだけ配置する指定です。

  * `nodeSelectorTerms`

    `Node` を選ぶ条件を定義します。

  * `matchExpressions`

    ラベルの条件を指定します。

  * `key: disktype`

    対象にする `Node` ラベルのキーです。

  * `operator: In`

    指定した値のいずれかに一致する場合、条件に合うと判断します。

  * `values`

    許可する値の一覧です。

  * `ssd`

    `disktype=ssd` の `Node` を対象にします。

* `podAffinity`

  `podAffinity` は、すでに動いている `Pod` のラベルを条件にして、その `Pod` と近い場所に新しい `Pod` を配置します。

  例えば、`app=cache` の `Pod` と同じ `Node` に、`app=web` の `Pod` を配置したい場合に使います。

  ```mermaid
  flowchart LR
      subgraph NodeA["Node A"]
          Cache["Pod<br/>app=cache"]
          Web["Pod<br/>app=web"]
      end

      subgraph NodeB["Node B"]
      end

      Cache --- Web
  ```

* `podAffinity` の例

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: web-pod
    labels:
      app: web
  spec:
    affinity:
      podAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
                - key: app
                  operator: In
                  values:
                    - cache
            topologyKey: kubernetes.io/hostname
    containers:
      - name: nginx
        image: nginx:1.25.3
  ```

  この `YAML` は、`app=cache` の `Pod` が動いている `Node` と同じ `Node` に、`web-pod` を配置します。

  * `podAffinity`

    他の `Pod` のラベルを条件にして、近くに配置する設定です。

  * `labelSelector`

    対象にする `Pod` のラベル条件です。

  * `key: app`

    対象にする `Pod` ラベルのキーです。

  * `values: cache`

    `app=cache` の `Pod` を対象にします。

  * `topologyKey: kubernetes.io/hostname`

    どの単位で「近い」と判断するかを指定します。

    `kubernetes.io/hostname` は、同じ `Node` を意味します。

* `podAntiAffinity`

  `podAntiAffinity` は、すでに動いている `Pod` のラベルを条件にして、その `Pod` と離れた場所に新しい `Pod` を配置します。

  例えば、同じアプリケーションの `Pod` を別々の `Node` に分散させたい場合に使います。

  ```mermaid
  flowchart LR
      subgraph NodeA["Node A"]
          Pod1["Pod<br/>app=web"]
      end

      subgraph NodeB["Node B"]
          Pod2["Pod<br/>app=web"]
      end

      Pod1 -.->|別Nodeに分散| Pod2
  ```

* `podAntiAffinity` の例

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: web-deployment
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: web
    template:
      metadata:
        labels:
          app: web
      spec:
        affinity:
          podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              - labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - web
                topologyKey: kubernetes.io/hostname
        containers:
          - name: nginx
            image: nginx:1.25.3
  ```

  この `YAML` は、`app=web` の `Pod` 同士が同じ `Node` に配置されないようにします。

  `replicas: 2` の場合、可能であれば別々の `Node` に配置されます。

  * `podAntiAffinity`

    他の `Pod` のラベルを条件にして、離して配置する設定です。

  * `requiredDuringSchedulingIgnoredDuringExecution`

    条件を必ず満たすように配置します。

  * `labelSelector`

    離して配置したい対象の `Pod` をラベルで指定します。

  * `topologyKey: kubernetes.io/hostname`

    同じ `Node` に配置しない、という意味になります。

* `required` と `preferred` の違い

  `Affinity` や `Anti-affinity` には、強制ルールと希望ルールがあります。

  | 種類 | 意味 |
  | ---- | ---- |
  | `requiredDuringSchedulingIgnoredDuringExecution` | 条件を必ず満たす必要がある |
  | `preferredDuringSchedulingIgnoredDuringExecution` | できれば条件を満たしたい |

* `requiredDuringSchedulingIgnoredDuringExecution`

  条件を満たす `Node` がない場合、`Pod` は起動できません。

  ```text
  条件に合うNodeがある
    ↓
  Podを配置できる

  条件に合うNodeがない
    ↓
  PodはPendingのまま
  ```

* `preferredDuringSchedulingIgnoredDuringExecution`

  条件を満たす `Node` を優先しますが、必須ではありません。

  条件に合う `Node` がない場合でも、別の `Node` に配置される可能性があります。

  ```text
  条件に合うNodeがある
    ↓
  なるべくそこに配置する

  条件に合うNodeがない
    ↓
  他のNodeにも配置される可能性がある
  ```

* `Affinity` と `Anti-affinity` の使い分け

  | 目的 | 使う設定 |
  | ---- | ---- |
  | 特定の `Node` に配置したい | `nodeAffinity` |
  | 特定の `Pod` の近くに配置したい | `podAffinity` |
  | 特定の `Pod` と離して配置したい | `podAntiAffinity` |
  | 必ず条件を満たしたい | `requiredDuringSchedulingIgnoredDuringExecution` |
  | できれば条件を満たしたい | `preferredDuringSchedulingIgnoredDuringExecution` |

* 初心者向けの覚え方

  * `Affinity`

    「近くに置きたい」「条件に合う場所に置きたい」という設定です。

  * `Anti-affinity`

    「離して置きたい」「同じ場所に置きたくない」という設定です。

  * `nodeAffinity`

    `Node` のラベルを見て配置先を決めます。

  * `podAffinity`

    他の `Pod` のラベルを見て、近くに配置します。

  * `podAntiAffinity`

    他の `Pod` のラベルを見て、離して配置します。

* `Taint` / `Toleration` との違い

  | 仕組み | 主な役割 |
  | ---- | ---- |
  | `Taint` / `Toleration` | `Node` 側で「来てもよいPod」を制限する |
  | `Affinity` / `Anti-affinity` | `Pod` 側で「どこに置きたいか」を指定する |

  簡単にいうと、次のような違いです。

  ```text
  Taint / Toleration
    Node側の制限

  Affinity / Anti-affinity
    Pod側の配置希望・配置条件
  ```

## `Taint` と `Toleration`

  `Taint` と `Toleration` は、`Pod` を特定の `Node` に配置させない、または特定の条件を満たす `Pod` だけ配置できるようにする仕組みです。

  簡単にいうと、次のような役割です。

* `Taint`

  `Node` 側に付ける「制限」です。

* `Toleration`

  `Pod` 側に付ける「許可」です。

  `Node` に `Taint` が付いている場合、その `Taint` を許容できる `Toleration` を持つ `Pod` だけが、その `Node` に配置されます。

* イメージ

  ```text
  NodeにTaintがある
    ↓
  通常のPodは配置されない
    ↓
  Tolerationを持つPodだけ配置できる
  ```

* 構成図

  ```mermaid
  flowchart LR
      Pod1["Pod A<br/>Toleration なし"]
      Pod2["Pod B<br/>Toleration あり"]

      Node["Node<br/>Taint あり"]

      Pod1 -.配置できない.-> Node
      Pod2 --> Node
  ```

* `Taint`

  `Taint` は、`Node` に設定する制限です。

  `Taint` を付けることで、条件に合わない `Pod` がその `Node` に配置されないようにできます。

  例えば、次のような用途で使います。

  * 特定の用途専用の `Node` にしたい
  * GPU用の `Node` に通常の `Pod` を配置したくない
  * システム用の `Node` にアプリケーション用の `Pod` を配置したくない
  * 障害がある `Node` から `Pod` を退避させたい

* `Toleration`

  `Toleration` は、`Pod` に設定する許可です。

  `Toleration` を設定すると、対応する `Taint` が付いた `Node` にも配置できるようになります。

  ただし、`Toleration` は「その `Node` に必ず配置する」という意味ではありません。

  あくまで「その `Node` に配置されてもよい」という許可です。

* `Taint` の設定例

  ```bash
  kubectl taint nodes worker-node dedicated=batch:NoSchedule
  ```

  このコマンドは、`worker-node` という `Node` に `Taint` を付けます。

  * `kubectl`

    `Kubernetes` クラスタを操作するためのコマンドです。

  * `taint`

    `Node` に `Taint` を付けたり、削除したりするためのコマンドです。

  * `nodes worker-node`

    対象の `Node` を指定しています。

  * `dedicated=batch:NoSchedule`

    付与する `Taint` の内容です。

    * `dedicated`

      `Taint` のキーです。

    * `batch`

      `Taint` の値です。

    * `NoSchedule`

      `Taint` の効果です。  
      この `Taint` を許容できない `Pod` は、この `Node` に新しく配置されません。

* `Toleration` の設定例

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: batch-pod
  spec:
    tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "batch"
        effect: "NoSchedule"
    containers:
      - name: batch-pod
        image: nginx:1.25.3
  ```

  この `YAML` は、`dedicated=batch:NoSchedule` という `Taint` を許容できる `Pod` を作成します。

  * `tolerations`

    `Pod` に設定する `Toleration` です。

  * `key: "dedicated"`

    許容する `Taint` のキーです。

  * `operator: "Equal"`

    `key` と `value` が一致する `Taint` を許容する指定です。

  * `value: "batch"`

    許容する `Taint` の値です。

  * `effect: "NoSchedule"`

    許容する `Taint` の効果です。

* `Taint` の効果

  `Taint` には主に次の3種類の効果があります。

  | `effect` | 意味 |
  | ---- | ---- |
  | `NoSchedule` | `Toleration` がない `Pod` は新しく配置されない |
  | `PreferNoSchedule` | できるだけ配置しない |
  | `NoExecute` | `Toleration` がない既存の `Pod` も退避される |

* `NoSchedule`

  ```text
  新しくPodを配置するときに制限する
  ```

  `Taint` を許容できない `Pod` は、その `Node` に新しく配置されません。

  すでにその `Node` で動いている `Pod` は、基本的にはそのままです。

* `PreferNoSchedule`

  ```text
  できれば配置しない
  ```

  `Taint` を許容できない `Pod` は、できるだけその `Node` に配置されません。

  ただし、他に配置できる `Node` がない場合などは、配置される可能性があります。

* `NoExecute`

  ```text
  既存のPodも退避対象になる
  ```

  `Taint` を許容できない `Pod` は、その `Node` から退避されます。

  新しく配置されないだけでなく、すでに動いている `Pod` にも影響します。

* `Taint` の削除例

  ```bash
  kubectl taint nodes worker-node dedicated=batch:NoSchedule-
  ```

  末尾に `-` を付けることで、指定した `Taint` を削除できます。

* 初心者向けの覚え方

  * `Taint`

    `Node` 側の「この条件に合わない `Pod` は来ないでください」という制限。

  * `Toleration`

    `Pod` 側の「その制限があっても大丈夫です」という許可。

  * `Taint` は拒否する仕組み

    `Node` に `Taint` を付けると、条件に合わない `Pod` を避けられます。

  * `Toleration` は許可する仕組み

    `Pod` に `Toleration` を付けると、対応する `Taint` がある `Node` にも配置できます。

* 注意点

  `Toleration` を設定しても、その `Node` に必ず配置されるわけではありません。

  特定の `Node` に配置したい場合は、`nodeSelector` や `nodeAffinity` と組み合わせて使います。

  ```text
  Toleration
    ↓
  そのNodeに配置されてもよい

  nodeSelector / nodeAffinity
    ↓
  そのNodeに配置したい
  ```

## `Pod Priority` と `Preemption`

* `Pod Priority` と `Preemption`

  `Pod Priority` と `Preemption` は、`Pod` の重要度を決めて、重要な `Pod` を優先的にスケジュールするための仕組みです。

  簡単にいうと、次のような役割です。

  * `Pod Priority`

    `Pod` の優先度を決める仕組みです。

  * `Preemption`

    優先度の高い `Pod` を配置するために、優先度の低い `Pod` を退避させる仕組みです。

  例えば、クラスタ内のリソースが不足している場合、重要度の高い `Pod` を起動するために、重要度の低い `Pod` が削除されることがあります。

* イメージ

  ```text
  クラスタのリソースが不足している
    ↓
  優先度の高いPodを配置したい
    ↓
  優先度の低いPodを退避させる
    ↓
  優先度の高いPodを配置する
  ```

* 構成図

  ```mermaid
  flowchart LR
      HighPod["優先度の高いPod<br/>priority: high"]
      LowPod["優先度の低いPod<br/>priority: low"]
      Node["Node<br/>リソース不足"]

      LowPod --> Node
      HighPod -.配置したい.-> Node
      LowPod -.退避される.-> Evicted["削除 / 退避"]
      HighPod --> Node
  ```

* `Pod Priority`

  `Pod Priority` は、`Pod` に優先度を付ける仕組みです。

  優先度が高い `Pod` ほど、スケジューリング時に優先されます。

  優先度は、`PriorityClass` というリソースで定義します。

* `PriorityClass`

  `PriorityClass` は、`Pod` に設定する優先度の名前と値を定義するリソースです。

  例えば、次のような優先度を作成できます。

  * `high-priority`
  * `medium-priority`
  * `low-priority`

  優先度の値は数値で指定します。

  数値が大きいほど、優先度が高くなります。

* `PriorityClass` の例

  ```yaml
  apiVersion: scheduling.k8s.io/v1
  kind: PriorityClass
  metadata:
    name: high-priority
  value: 100000
  globalDefault: false
  description: "重要なPod用のPriorityClass"
  ```

  この `YAML` は、`high-priority` という名前の `PriorityClass` を作成します。

  * `apiVersion: scheduling.k8s.io/v1`

    `PriorityClass` を作成するための `API` バージョンです。

  * `kind: PriorityClass`

    作成するリソースの種類です。

  * `metadata.name`

    `PriorityClass` の名前です。

  * `value: 100000`

    優先度の値です。  
    数値が大きいほど優先度が高くなります。

  * `globalDefault: false`

    この `PriorityClass` をデフォルトの優先度として使うかどうかを指定します。

    `false` の場合、この `PriorityClass` を使うには `Pod` 側で明示的に指定する必要があります。

  * `description`

    `PriorityClass` の説明です。

* `Pod` で `PriorityClass` を指定する例

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: important-pod
  spec:
    priorityClassName: high-priority
    containers:
      - name: nginx
        image: nginx:1.25.3
  ```

  この `YAML` は、`important-pod` に `high-priority` という `PriorityClass` を指定しています。

  * `priorityClassName: high-priority`

    `Pod` に設定する `PriorityClass` の名前です。

    これにより、この `Pod` は `high-priority` の優先度を持ちます。

* `Preemption`

  `Preemption` は、優先度の高い `Pod` を配置するために、優先度の低い `Pod` を退避させる仕組みです。

  クラスタに十分な空きリソースがある場合は、`Preemption` は発生しません。

  `Preemption` が発生するのは、主に次のような場合です。

  * 優先度の高い `Pod` を作成した
  * しかし、どの `Node` にも空きリソースが足りない
  * 優先度の低い `Pod` を削除すれば配置できる

  このような場合、`Kubernetes` は優先度の低い `Pod` を削除して、優先度の高い `Pod` を配置しようとします。

* `Preemption` の流れ

  ```text
  優先度の高いPodを作成
    ↓
  Schedulerが配置先Nodeを探す
    ↓
  空きリソースが足りない
    ↓
  優先度の低いPodを削除すれば配置できるか確認
    ↓
  低優先度Podを削除
    ↓
  高優先度Podを配置
  ```

* `Preemption` の構成図

  ```mermaid
  flowchart TD
      A["高優先度Podを作成"]
      B["Schedulerが配置先を探す"]
      C["空きリソース不足"]
      D["低優先度Podを退避できるか確認"]
      E["低優先度Podを削除"]
      F["高優先度Podを配置"]

      A --> B
      B --> C
      C --> D
      D --> E
      E --> F
  ```

* `preemptionPolicy`

  `preemptionPolicy` を使うと、`Pod` が他の `Pod` を退避させるかどうかを制御できます。

  主に次の2つがあります。

  | 設定値 | 意味 |
  | ---- | ---- |
  | `PreemptLowerPriority` | 低優先度の `Pod` を退避させる可能性がある |
  | `Never` | 他の `Pod` を退避させない |

* `preemptionPolicy: Never` の例

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: important-but-no-preemption-pod
  spec:
    priorityClassName: high-priority
    preemptionPolicy: Never
    containers:
      - name: nginx
        image: nginx:1.25.3
  ```

  この `YAML` は、優先度は高いですが、他の `Pod` を退避させない `Pod` を作成します。

  * `priorityClassName: high-priority`

    優先度の高い `PriorityClass` を指定します。

  * `preemptionPolicy: Never`

    他の低優先度 `Pod` を退避させない設定です。

    優先度は高く扱われますが、配置するために他の `Pod` を削除することはありません。

* `Pod Priority` と `Preemption` の違い

  | 項目 | 説明 |
  | ---- | ---- |
  | `Pod Priority` | `Pod` の優先度を決める仕組み |
  | `PriorityClass` | 優先度の名前と値を定義するリソース |
  | `Preemption` | 高優先度 `Pod` のために低優先度 `Pod` を退避させる仕組み |
  | `preemptionPolicy` | `Preemption` を許可するか制御する設定 |

* 初心者向けの覚え方

  * `PriorityClass`

    優先度のルールを作る。

  * `priorityClassName`

    `Pod` に優先度を設定する。

  * `Pod Priority`

    どの `Pod` が重要かを決める。

  * `Preemption`

    重要な `Pod` を起動するために、重要度の低い `Pod` をどかす。

  * `preemptionPolicy: Never`

    優先度は高いが、他の `Pod` はどかさない。

* 注意点

  `Preemption` は便利ですが、低優先度の `Pod` が削除される可能性があります。

  そのため、次のような重要な処理では慎重に設定する必要があります。

  * データ処理中の `Job`
  * ユーザー影響があるアプリケーション
  * 再実行が難しいバッチ処理
  * 停止すると困る検証環境

  重要な `Pod` だけに高い優先度を付けるようにし、すべての `Pod` に高い優先度を付けないように注意します。
