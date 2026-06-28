# Kubernetes クラスタを構築する

1. `kubectl` のインストール

    [kubectl の Windows インストール手順](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl-windows/)

    * `kubectl` コマンドが使用可能か確認

        ```bash
        kubectl version --client
        ```

    * メモ

      `コマンドプロンプト` で `curl` を使用

   &nbsp;

2. `kind` のインストール

    [kind の Windows インストール手順](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

    * `kind` コマンドが使用可能か確認

        ```bash
        kind version
        ```

    * メモ

      `PowerShell` で `winget` を使用

   &nbsp;

3. `Kubernetes` クラスタを構築

   * デフォルトの `Kubernetes` イメージでクラスタを構築

      ```bash
      kind create cluster
      ```

   * メモ

      `kind` は `Docker` コンテナを使って `Kubernetes` クラスタ(環境)を作る

   &nbsp;

4. `Kubernetes` クラスタと接続を確認

   * 下記のコマンドを実行

      ```bash
      kubectl cluster-info --context kind-kind
      ```

   &nbsp;

5. `Kubernetes` クラスタを削除

   * 下記のコマンドを実行

      ```bash
      kind delete cluster
      ```
