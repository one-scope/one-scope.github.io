---
title: "Golang 実践入門"
linkTitle: "Golang 実践入門"
weight: 10
---

{{% pageinfo %}}
ドキュメントは作成中です。
{{% /pageinfo %}}

実務で Golang を使うための最短ルートを記載する目的です。
使用頻度の低いものなどはどんどん省いていきます。

より Golang らしいコードを書きたい場合は以下のリソースを参照してください。
- [100 Go Mistakes and How to Avoid Them](https://github.com/teivah/100-go-mistakes)
- [Effective Go](https://go.dev/doc/effective_go)
- [Go by Example](https://gobyexample.com/)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Go Walkthrough](https://medium.com/go-walkthrough)
- [GOLANGBOT.COM](https://golangbot.com/)
- [How to avoid Go gotchas](https://divan.dev/posts/avoid_gotchas/)
- [Idiomatic Go](https://dmitri.shuralyov.com/idiomatic-go)

# 基礎

## Hello World

リポジトリの直下で以下を実行し、モジュールを初期化します。  
モジュールは、プロジェクトのようなもので依存しているパッケージや Golang のバージョンを管理するものです。  
成功すると `go.mod` ファイルが作成されます。

```bash
$ go mod init github.com/{your-userid}/{your-repository}
```

次に、ソースコードを記述するファイルをつくります。  
Golang のソースコードには拡張子 `.go` をつけます。  
`main` パッケージの `main` 関数がプログラムのエントリポイントとなります。  
エントリポイントのあるファイルは一般的に `main.go` というファイル名をつけます。

```go
package main

func main() {
    println("Hello OneScope.")
}
```

`main.go` ファイルがつくれたら、以下のコマンドでソースコードをビルドします。  
成功するとリポジトリ名と同じ名前の実行ファイルが出力されます。  
この実行ファイルは、OSやCPUアーキテクチャが同じならどこでも動きます。

```bash
$ go build
```

## 型とゼロ値

Golang にはいろいろな型がありますが、とりあえず覚えるべきものだけ列挙します。

|型名   |概要                  |ゼロ値|
|-------|----------------------|------|
|bool   |真偽値を取り扱う      |false |
|int    |整数を取り扱う        |0     |
|float64|浮動小数点数を取り扱う|0     |
|string |文字列を取り扱う      |""    |

※ スライス、マップ、ポインタのゼロ値は `nil` です。

## 変数

変数を宣言するには主に２種類の方法があります。

変数を宣言し初期化するには次のように `変数名 := 値` と記述します。

```go
  v := 10
```

初期化せず、ただ変数を宣言する場合は、`var 変数名 型` と記述します。  
このとき、変数の値はその型のゼロ値です。

```go
 var v int
```

## 制御構文

### if と else

単純な if 文です。条件式に()は必要ありません。

```go
if v == 0 {
    println("v is zero")
}
```

if ～ else 文です。

```go
if v == 0 {
    println("v is zero")
} else {
    println("v is not zero")
}
```

else if 文です。

```go
if v == 0 {
    println("v is zero")
} else if v == 1 {
    println("v is one")
} else {
    println("v is not zero and one")
}
```

Golang では if 文で(C言語の for 文のように)初期化子を記述できます。  
この場合、変数 `v` のスコープは if ～ else を抜けるまで（この例の0～4行目）です。

```go
if v := GetValue(); v == 0 {
    println("v is zero")
} else {
    println("v is not zero")
}
```

### switch

```go
switch v {
    case 0:
    case 1:
    case 2:
    default:
}
```

```go
switch v {
    case 0, 1:
    case 2:
    default:
}
```

```go
switch v := GetValue(); v {
    case 0, 1:
    case 2:
    default:
}
```

```go
switch {
    case v % 2 == 0:
    default:
}
```

### for と for range

```go
for i := 0; i < 10; i++ {
    println(i)
}
```

```go
for true {
    println("infinity")
}
```

```go
s := []int{1, 2, 3}
for i, v := range s {
    println(i, v)
}
```

```go
s := []int{1, 2, 3}
for i := range s {
    println(i, s[i])
}
```

```go
s := []int{1, 2, 3}
for _, v := range s {
    println(v)
}
```

```go
m := map[string]int{"壱": 1, "弐": 2, "参": 3}
for k, v := range m {
    println(k, v)
}
```

```go
m := map[string]int{"壱": 1, "弐": 2, "参": 3}
for k := range m {
    println(k, v[k])
}
```

```go
m := map[string]int{"壱": 1, "弐": 2, "参": 3}
for _, v := range m {
    println(v)
}
```

### break と continue

## 関数

### 基本的な関数

引数を受け取らず、何も返さない関数です。

```go
func PrintHello() {
    println("Hello")
}
```

int 型の引数 `v` を受け取り、何も返さない関数です。

```go
func PrintInt(v int) {
    println(v)
}
```

int 型の引数 `v` を受け取り、int 型の戻り値を返す関数です。

```go
func Inclement(v int) int {
    return v + 1
}
```

### 複数の戻り値を返す関数

int 型の引数 `v` を受け取り、int 型と error 型の戻り値を返す関数です。  
Golang ではエラーが発生したかどうかを、最終戻り値の error 型が `nil` かどうかで判断する文化があります。  
一般的に、error 型が非 `nil` の場合、その他の戻り値はゼロ値とします。

```go
func Inclement(v int) (int, error) {
    if v < 0 {
        return 0, errors.New("v is not positive.")
    }
    return v + 1, nil
}
```

### 型引数を受け取る関数

型引数 `V` をとり、`V` 型の引数 `v1`、`v2` を受け取り、大きいほうを返す関数です。  
`v1` と `v2` が比較可能でなければならないため、型引数 `V` に型制約 `constraints.Ordered` を指定します。

```go
func Max[V constraints.Ordered](v1 V, v2 V) V {
    if v1 > v2 {
        return v1
    }
    return v2
}
```

## 構造体

## map と slice

```go
m := map[string]int{}
```

```go
m := map[string]int{}
m["key"] = 100
```

```go
m := map[string]int{"key": 100}
delete(m, "key")
```

```go
m := map[string]int{"key": 100}
for k, v := range m {
    println(k, v)
}
```

```go
s := []int{}
```

```go
s := make([]int, 0, 100)
```

## パッケージ

## エラーハンドリング

## 標準パッケージ

### os パッケージ

### path パッケージと filepath パッケージ

### fmt パッケージ

### io パッケージ

### log パッケージ

### strings パッケージと bytes パッケージ

### strconv パッケージ

## 構造体メソッド

## インターフェース

### io.Reader を満たす MyReader をつくる

### MyReader を直接変更せず、io.ReadCloser を満たすように拡張するユーティリティをつくる

### 標準パッケージが提供する interface を知る

## 外部パッケージ

### go mod init

### go get

### go mod tidy

### go mod vendor

## error 型をつくる

## CLI をつくる

### flag パッケージ

### 標準出力と標準エラー出力（一般）

### オプション引数とコマンド引数（一般）

### 標準入力とパイプ（一般）

### github.com/urfave/cli パッケージでサブコマンドをつくる

### リリースする

クロスビルド

github Release ページ

git tag

セマンティックバージョニング

## WebServer をつくる

### net/http パッケージ

### http.Handler と http.HandlerFunc

### URL クエリパラメータ（一般）

### Status Code（一般）

200
401
403
404
500

### Response Header（一般）

Content-Type
Content-Length

### Response Body（一般）

### github.com/go-chi/chi パッケージでのルーティング

パスパラメータ

### Request Header（一般）

### Request Body（一般）

io.LimitReader

### REST API を実装する

encoding/json パッケージ

### リリースする

Dockerize する

マルチステージビルド

.dockerignore

volume の永続化

ポートの expose

docker-compose

github Release ページ

git tag

セマンティックバージョニング

## go キーワード

### 関数を非同期で実行する

### chan による待ち合わせ

### sync.WaitGroup による待ち合わせ

### golang.org/x/sync/errgroup パッケージによる待ち合わせ

## 排他制御

### ミューテックス

sync.Mutex、sync.RWMutex

### セマフォ

chan を使ったセマフォ

### sync/atomic パッケージ

### golang.org/x/sync/singleflight パッケージ 

## context パッケージ

### withCancel でキャンセルする

### WithCancelCause でキャンセルされた理由を伝搬する

## 関数を受け取る関数

## 関数を返す関数

## 関数メソッド

## reflect パッケージ

## type アサーション

## type switch

## chan と select

## context.WithCancel
