# Goアプリをビルドするためのステージ
# AS builder と書くことで、このステージに builder という名前を付けている
# 後のステージで「builderステージからファイルをコピーする」と指定できる
FROM golang:1.21 AS builder

# コンテナ内の作業ディレクトリを /app にする
# 以降の COPY や RUN は、この /app を基準に実行される
WORKDIR /app

# ホスト側の現在のディレクトリのファイルを、コンテナ内の /app にコピーする
# 例: main.go や go.mod などがコピーされる
COPY . .

# CGOを無効にする設定
# scratch イメージのような最小構成の環境でも動かしやすいバイナリを作るために指定する
ENV CGO_ENABLED=0

# Goアプリをビルドして、hello という名前の実行ファイルを作成する
# -o hello は「出力ファイル名を hello にする」という意味
RUN go build -o hello .

# 実行用のステージ
# scratch は中身がほぼ空の、Docker公式の最小イメージ
# OSの基本コマンドやシェルも入っていないため、非常に軽量
FROM scratch

# builderステージで作成した /app/hello を、このステージの /hello にコピーする
# --from=builder により、前の builder ステージからコピーしている
COPY --from=builder /app/hello /hello

# コンテナ起動時に実行するコマンドを指定する
# この場合、/hello という実行ファイルを起動する
ENTRYPOINT ["/hello"]