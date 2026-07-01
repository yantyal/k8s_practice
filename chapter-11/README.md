# PowerShellでBase64エンコード・デコードする方法

PowerShellでは、`.NET` の機能を使って文字列やファイルをBase64形式に変換できます。

Base64は**暗号化ではありません**。  
Base64に変換した文字列は、簡単に元に戻せます。

---

## 1. 文字列をBase64エンコードする

### コマンド

```powershell
$text = "hello"
$bytes = [System.Text.Encoding]::UTF8.GetBytes($text)
$base64 = [Convert]::ToBase64String($bytes)
$base64
```

### 実行結果

```text
aGVsbG8=
```

### 説明

* `$text`

  Base64エンコードしたい文字列です。

* `[System.Text.Encoding]::UTF8.GetBytes($text)`

  文字列を`UTF-8`のバイト配列に変換します。

* `[Convert]::ToBase64String($bytes)`

  バイト配列をBase64文字列に変換します。

---

## 2. Base64文字列をデコードする

### コマンド

```powershell
$base64 = "aGVsbG8="
$bytes = [Convert]::FromBase64String($base64)
$text = [System.Text.Encoding]::UTF8.GetString($bytes)
$text
```

### 実行結果

```text
hello
```

### 説明

* `$base64`

  デコードしたいBase64文字列です。

* `[Convert]::FromBase64String($base64)`

  Base64文字列をバイト配列に戻します。

* `[System.Text.Encoding]::UTF8.GetString($bytes)`

  バイト配列を`UTF-8`の文字列に戻します。

---

## 3. 1行でBase64エンコードする

### コマンド

```powershell
[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("hello"))
```

### 実行結果

```text
aGVsbG8=
```

---

## 4. 1行でBase64デコードする

### コマンド

```powershell
[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("aGVsbG8="))
```

### 実行結果

```text
hello
```

---

## 5. 日本語をBase64エンコードする

### コマンド

```powershell
$text = "こんにちは"
$base64 = [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes($text))
$base64
```

### 実行結果

```text
44GT44KT44Gr44Gh44Gv
```

---

## 6. 日本語のBase64をデコードする

### コマンド

```powershell
$base64 = "44GT44KT44Gr44Gh44Gv"
$text = [System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($base64))
$text
```

### 実行結果

```text
こんにちは
```

---

## 7. ファイルをBase64エンコードする

ファイルの中身をBase64文字列に変換する場合は、次のようにします。

### コマンド

```powershell
$bytes = [System.IO.File]::ReadAllBytes("C:\work\sample.txt")
$base64 = [Convert]::ToBase64String($bytes)
$base64
```

### 説明

* `[System.IO.File]::ReadAllBytes("C:\work\sample.txt")`

  指定したファイルをバイト配列として読み込みます。

* `[Convert]::ToBase64String($bytes)`

  ファイルのバイト配列をBase64文字列に変換します。

---

## 8. Base64文字列をファイルに戻す

Base64文字列をデコードして、ファイルとして保存する場合は、次のようにします。

### コマンド

```powershell
$base64 = "Base64文字列をここに入れる"
$bytes = [Convert]::FromBase64String($base64)
[System.IO.File]::WriteAllBytes("C:\work\decoded.txt", $bytes)
```

### 説明

* `[Convert]::FromBase64String($base64)`

  Base64文字列をバイト配列に戻します。

* `[System.IO.File]::WriteAllBytes("C:\work\decoded.txt", $bytes)`

  バイト配列をファイルとして書き込みます。

---

## 9. Base64エンコード結果をファイルに保存する

Base64エンコードした結果をテキストファイルに保存したい場合は、次のようにします。

### コマンド

```powershell
$bytes = [System.IO.File]::ReadAllBytes("C:\work\sample.txt")
$base64 = [Convert]::ToBase64String($bytes)
$base64 | Out-File -FilePath "C:\work\sample_base64.txt" -Encoding UTF8
```

### 説明

* `Out-File`

  コマンドの実行結果をファイルに出力します。

* `-Encoding UTF8`

  出力ファイルの文字コードを`UTF-8`にします。

---

## 10. Base64文字列が入ったファイルを読み込んでデコードする

### コマンド

```powershell
$base64 = Get-Content -Path "C:\work\sample_base64.txt" -Raw
$bytes = [Convert]::FromBase64String($base64)
[System.IO.File]::WriteAllBytes("C:\work\decoded.txt", $bytes)
```

### 説明

* `Get-Content -Raw`

  ファイルの内容を1つの文字列として読み込みます。

* `[Convert]::FromBase64String($base64)`

  読み込んだBase64文字列をバイト配列に戻します。

* `[System.IO.File]::WriteAllBytes(...)`

  デコードした内容をファイルとして保存します。

---

## 11. よく使うコマンドまとめ

### 文字列をBase64エンコード

```powershell
[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("変換したい文字列"))
```

### Base64文字列をデコード

```powershell
[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("Base64文字列"))
```

### ファイルをBase64エンコード

```powershell
[Convert]::ToBase64String([System.IO.File]::ReadAllBytes("C:\work\sample.txt"))
```

### Base64文字列をファイルに戻す

```powershell
[System.IO.File]::WriteAllBytes("C:\work\decoded.txt", [Convert]::FromBase64String($base64))
```

---

## 12. 注意点

* Base64は暗号化ではありません。
* Base64にした文字列は簡単に元に戻せます。
* パスワードや秘密情報をBase64にしただけでは安全ではありません。
* 文字列を扱う場合は、基本的に`UTF-8`を使うとよいです。
* 日本語を扱う場合も`UTF-8`を使えば問題ありません。
* ファイルを扱う場合は、文字列ではなくバイト配列として処理します。

---

## 13. まとめ

| 操作 | コマンド |
| --- | --- |
| 文字列をBase64エンコード | `[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("hello"))` |
| Base64文字列をデコード | `[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("aGVsbG8="))` |
| ファイルをBase64エンコード | `[Convert]::ToBase64String([System.IO.File]::ReadAllBytes("C:\work\sample.txt"))` |
| Base64文字列をファイルに戻す | `[System.IO.File]::WriteAllBytes("C:\work\decoded.txt", [Convert]::FromBase64String($base64))` |
