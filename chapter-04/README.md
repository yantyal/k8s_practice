# Pod について

* `kube-system` 名前空間にある `Pod` 一覧を表示する

  ```bash
  kubectl get pod --namespace kube-system
  ```

  * メモ

    主に、Kubernetesクラスタ自体が正常に動いているか確認するために使う

&nbsp;

* クラスタが起動できているか確認する

  ```bash
  kubectl get nodes
  ```

* クラスタが起動できているか確認する(`kind` を使用している場合)

  ```bash
  kind get clusters
  ```

* `Pod` が存在しないことを確認する

  ```bash
  kubectl get pod --namespace default
  ```

  結果

  ```bash
  No resources found in default namespace.
  ```

* マニフェストを適用する

  ```bash
  kubectl apply --filename chapter-04/myapp.yaml --namespace default
  ```

  結果

  ```bash
  pod/myapp created
  ```

* `Pod` が作成できていることを確認する

  ```bash
  kubectl get pod --namespace default
  ```

  結果

  ```text
  NAME    READY   STATUS    RESTARTS   AGE
  myapp   1/1     Running   0          83s
  ```
