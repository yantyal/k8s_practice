# トラブルシューティング

* リソースを取得する

  ```bash
  kubectl get pod --namespace default
  ```

  結果

  ```text
  NAME    READY   STATUS    RESTARTS   AGE
  myapp   1/1     Running   0          16m
  ```

  * `Namespace` とは

    単一のクラスタ内のリソース群を分離するために使うリソース

* リソースを取得する（リソース名を指定）

  ```bash
  kubectl get pod myapp --namespace default
  ```

  結果

    ```bash
    NAME    READY   STATUS    RESTARTS   AGE
    myapp   1/1     Running   0          19m
    ```

* リソースを取得する（IPアドレスやNode情報を取得する）

  ```bash
  kubectl get pod --output wide --namespace default
  ```

  結果

    ```bash
    NAME    READY   STATUS    RESTARTS   AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
    myapp   1/1     Running   0          20m   10.244.0.5   kind-control-plane   <none>           <none>
    ```

* リソースを取得する（YAMLファイル形式で取得する）

  ```bash
  kubectl get pod --output yaml --namespace default
  ```

  結果

    ```yaml
    apiVersion: v1
    items:
    - apiVersion: v1
    kind: Pod
    metadata:
        annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
            {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"myapp"},"name":"myapp","namespace":"default"},"spec":{"containers":[{"image":"blux2/hello-server:1.0","name":"hello-server","ports":[{"containerPort":9080}]}]}}
        creationTimestamp: "2026-06-28T09:03:43Z"
        generation: 1
        labels:
        app: myapp
        name: myapp
        namespace: default
        resourceVersion: "1889"
        uid: 130201cc-db71-4342-988c-30b0bfc4997c
    spec:
        containers:
        - image: blux2/hello-server:1.0
        imagePullPolicy: IfNotPresent
        name: hello-server
        ports:
        - containerPort: 9080
            protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
            name: kube-api-access-l6h2b
            readOnly: true
        dnsPolicy: ClusterFirst
        enableServiceLinks: true
        nodeName: kind-control-plane
        preemptionPolicy: PreemptLowerPriority
        priority: 0
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        serviceAccount: default
        serviceAccountName: default
        terminationGracePeriodSeconds: 30
        tolerations:
        - effect: NoExecute
        key: node.kubernetes.io/not-ready
        operator: Exists
        tolerationSeconds: 300
        - effect: NoExecute
        key: node.kubernetes.io/unreachable
        operator: Exists
        tolerationSeconds: 300
        volumes:
        - name: kube-api-access-l6h2b
        projected:
            defaultMode: 420
            sources:
            - serviceAccountToken:
                expirationSeconds: 3607
                path: token
            - configMap:
                items:
                - key: ca.crt
                path: ca.crt
                name: kube-root-ca.crt
            - downwardAPI:
                items:
                - fieldRef:
                    apiVersion: v1
                    fieldPath: metadata.namespace
                path: namespace
    status:
        conditions:
        - lastProbeTime: null
        lastTransitionTime: "2026-06-28T09:03:43Z"
        observedGeneration: 1
        status: "True"
        type: PodReadyToStartContainers
        - lastProbeTime: null
        lastTransitionTime: "2026-06-28T09:03:43Z"
        observedGeneration: 1
        status: "True"
        type: Initialized
        - lastProbeTime: null
        lastTransitionTime: "2026-06-28T09:03:48Z"
        observedGeneration: 1
        status: "True"
        type: Ready
        - lastProbeTime: null
        lastTransitionTime: "2026-06-28T09:03:48Z"
        observedGeneration: 1
        status: "True"
        type: ContainersReady
        - lastProbeTime: null
        lastTransitionTime: "2026-06-28T09:03:43Z"
        observedGeneration: 1
        status: "True"
        type: PodScheduled
        containerStatuses:
        - containerID: containerd://b401f002ae998ba32ad56c320af38cb7beb2dc1f9c5aa2e4526411fd3b60f606
        image: docker.io/blux2/hello-server:1.0
        imageID: docker.io/blux2/hello-server@sha256:35ab584cbe96a15ad1fb6212824b3220935d6ac9d25b3703ba259973fac5697d
        lastState: {}
        name: hello-server
        ready: true
        resources: {}
        restartCount: 0
        started: true
        state:
            running:
            startedAt: "2026-06-28T09:03:48Z"
        user:
            linux:
            gid: 0
            supplementalGroups:
            - 0
            uid: 0
        volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
            name: kube-api-access-l6h2b
            readOnly: true
            recursiveReadOnly: Disabled
        hostIP: 172.18.0.2
        hostIPs:
        - ip: 172.18.0.2
        observedGeneration: 1
        phase: Running
        podIP: 10.244.0.5
        podIPs:
        - ip: 10.244.0.5
        qosClass: BestEffort
        resources: {}
        startTime: "2026-06-28T09:03:43Z"
    kind: List
    metadata:
    resourceVersion: ""
    ```

* リソースを取得する（YAMLファイル形式で出力する）

  ```bash
  kubectl get pod --output yaml --namespace default > chapter-05/pod.yaml
  ```

* リソースを取得する（JSON形式で出力する）

  ```bash
  kubectl get pod myapp --output jsonpath='{.spec.containers[].image}'
  ```

  結果

  ```bash
  'blux2/hello-server:1.0'
  ```

* リソースの詳細を取得する

  ```bash
  kubectl describe pod myapp
  ```

* コンテナのログを取得する

  ```bash
  kubectl logs myapp
  ```

  複数コンテナが存在する場合はコンテナを指定する

  ```bash
  kubectl logs myapp --container hello-server --namespace default
  ```

* 特定の `Deployment` にひもづく `Pod` のログを取得する

  ```bash
  kubectl logs deploy/<Deployment名>
  ```

* ラベルを指定して参照する `Pod` を絞り込む

  ```bash
  kubectl get pod --selector app=myapp
  ```

  結果

    ```text
    NAME          READY   STATUS    RESTARTS      AGE
    myapp         1/1     Running   1 (12m ago)   170m
    myapp-label   1/1     Running   0             39s
    ```

* ラベルを指定して複数の `Pod` のログを取得する

  ```bash
  kubectl logs --selector app=myapp
  ```

* デバッグ用のサイドカーコンテナを立ち上げる

  ```bash
  kubectl debug --stdin --tty myapp --image=curlimages/curl:8.4.0 --target=hello-server --namespace default -- sh
  ```

  各オプションの意味は以下のとおり。

  * `kubectl debug`

    デバッグ用の一時的なコンテナを起動するコマンド。

  * `--stdin`

    標準入力を有効にするオプション。  
    キーボードからコマンドを入力できるようにする。

  * `--tty`

    疑似ターミナルを割り当てるオプション。  
    シェルを対話的に操作できるようにする。

  * `myapp`

    デバッグ対象の Pod 名。

  * `--image=curlimages/curl:8.4.0`

    デバッグ用コンテナで使用するイメージ。  
    今回は `curl` コマンドが使える軽量イメージを指定している。

  * `--target=hello-server`

    デバッグ対象のコンテナ名。  
    `myapp` Pod の中にある `hello-server` コンテナを対象にする。

  * `--namespace default`

    対象の Pod が存在する Namespace。  
    今回は `default` Namespace を指定している。

  * `-- sh`

    デバッグ用コンテナ内で起動するコマンド。  
    今回は `sh` を起動して、コンテナ内で操作できるようにしている。

  `shell` が立ち上がったらコマンドを実行する。

  ```bash
  curl localhost:8080
  ```

* `busybox` を使って DNS 名前解決を確認する

  ```bash
  kubectl --namespace default run busybox --image=busybox:1.36.1 --restart=Never --command -- nslookup google.com
  ```

  このコマンドは、一時的に `busybox` `Pod` を起動し、`Pod` の中から `google.com` の名前解決ができるか確認するためのコマンド。

  * `kubectl run`

    Pod を起動するコマンド。

  * `--namespace default`

    `default` `Namespace` で `Pod` を起動する。

  * `busybox`

    作成する `Pod` 名。

  * `--image=busybox:1.36.1`

    `Pod` で使用するコンテナイメージ。

  * `--restart=Never`

    再起動しない単発の `Pod` として実行する。

  * `--command --`

    `--` より後ろを、コンテナ内で実行するコマンドとして扱う。

  * `nslookup google.com`

    `google.com` の DNS 名前解決を確認する。

* 実行結果をログで確認する

  ```bash
  kubectl logs busybox --namespace default
  ```

* `busybox` `Pod` を削除する

  ```bash
  kubectl delete pod busybox --namespace default
  ```

* コンテナにログインする

  ```bash
  kubectl delete pod busybox --namespace default
  ```

* `curl` 用の確認 `Pod` を起動する

  ```bash
  kubectl --namespace default run curlpod --image=curlimages/curl:8.4.0 --command -- /bin/sh -c "while true; do sleep infinity; done;"
  ```

  このコマンドは、`curl` を実行するための `Pod` を起動し続けるためのコマンド。

  * `kubectl run`

    Pod を作成して起動する。

  * `--namespace default`

    `default` `Namespace` に `Pod` を作成する。

  * `curlpod`

    作成する `Pod` 名。

  * `--image=curlimages/curl:8.4.0`

    `curl` コマンドが使えるコンテナイメージを指定する。

  * `--command --`

    `--` より後ろを、コンテナ内で実行するコマンドとして扱う。

  * `/bin/sh -c "while true; do sleep infinity; done;"`

    コンテナ内でシェルを起動し、終了しない処理を実行する。  
    これにより `Pod` がすぐ終了せず、あとから `kubectl exec` で中に入れるようになる。

* `curlpod` の中に入る

  ```bash
  kubectl --namespace default exec --stdin --tty curlpod -- /bin/sh
  ```

  * `exec`

    起動中の `Pod` 内でコマンドを実行する。

  * `--stdin`

    標準入力を有効にする。

  * `--tty`

    疑似ターミナルを割り当てる。

  * `curlpod`

    接続先の Pod 名。

  * `-- /bin/sh`

    Pod 内で `/bin/sh` を起動する。

* `myapp` に接続確認する（IPアドレスは `get pod myapp --output wide --namespace default` で確認済み）

  ```bash
  curl 10.244.0.5:8080
  ```

* `myapp` `Pod` にポートフォワードする

  ```bash
  kubectl port-forward myapp 5555:8080 --namespace default
  ```

  このコマンドは、PC の `localhost:5555` へのアクセスを、`Kubernetes` 上の `myapp` `Pod` の `8080` ポートへ転送するためのコマンド。

  * `kubectl port-forward`

    ローカルPCのポートと、`Kubernetes` 上の `Pod` や `Service` のポートをつなぐコマンド。

  * `myapp`

    ポートフォワード先の `Pod` 名。

  * `5555:8080`

    `5555` がローカルPC側のポート、`8080` が `Pod` 側のポート。  
    つまり、`localhost:5555` にアクセスすると、`Pod` の `8080` ポートへ転送される。

  * `--namespace default`

    `myapp` `Pod` が存在する `Namespace` を指定する。

* 別のターミナルから接続確認する

  ```bash
  curl localhost:5555
  ```
