---
title: "【Go1.20】context.WithCancelCauseでContextにキャンセル原因を追加できるようになりました "
linkTitle: "【Go1.20】context.WithCancelCauseでContextにキャンセル原因を追加できるようになりました "
date: 2023-06-06T00:00:00+09:00
description: >
  Go1.20で追加されたcontext.WithCancelCauseについて紹介します。
tags:
  - golang
---

## context（コンテキスト）とは
context パッケージでは、Context 型が定義されています。context は、**一連の処理を複数のゴルーチンに渡って行う**場合に威力を発揮します。具体的には、ゴルーチン間での情報伝達を行う際に Context 型を第一引数にとるように設計することができます。  
異なるゴルーチン間で情報伝達を行うにはチャネルを使う方法がありますが、拡張性や伝達制御などの点において context を用いる方法はより優れています。

## context を用いた情報伝達の例
言葉だけでは分かりにくいと思うので、ゴルーチン間で情報伝達を行う簡単な例を見ていきましょう。
```go
func main() {
	ctx0 := context.Background()	// コンテキストの生成
	ctx1, cancel := context.WithCancel(ctx0)
	// ...何らかの処理
	go process1(ctx1)
	// ...何らかの処理
	if true { // 場合によってキャンセルする
		cancel()
	}
	time.Sleep(5*time.Second)
}

func process1(ctx context.Context){
	ctx2, cancel := context.WithCancel(ctx)
	// ...何らかの処理
	go process2(ctx2)
	// ...何らかの処理
	if false { // 場合によってキャンセルする
		cancel()
	}
}

func process2(ctx context.Context){
	for {
		select {
		case <-ctx.Done():
			fmt.Println("context canceled")
			return 
		default:
			// ...何らかの処理
		}
	}
}
```
このコードは、`main` 関数からゴルーチン(`process1`)を立て、さらに `process1` から別のゴルーチン(`process2`)を立てています。  
ここで注目したいことは、あるコンテキストがキャンセルされた場合に、**そこから派生しているゴルーチンの処理もキャンセルする**ということです。  
コンテキストをキャンセルする方法を具体的に見ていきましょう。  
コードの3行目で、親コンテキスト(`ctx0`)を引数にとった `context.WithCancel` 関数が子コンテキスト(`ctx1`)とキャンセル関数(`cancel`)を返しています。このキャンセル関数 `cancel` で子コンテキストをキャンセルできます。キャンセルすると何が起こるかというと、`main` 関数から派生している `process1` と、さらにそこから派生している `process2` もストップします。  
このように、あるコンテキストがキャンセルされた場合、そこから派生するゴルーチンの処理も芋づる式にキャンセルされていきます。  
すなわち、コンテキストがキャンセルされたという情報がそこから派生するゴルーチンに情報伝達されている、というわけです。

## キャンセルに原因を追加する
ここからが本題になります。  
ここまで見てきた従来のコンテキストのキャンセルの伝達では、どこでキャンセルされたのか？という情報は含まれていません。上の例でいえば、`process2` で `ctx` がキャンセルされた場合、その原因が `ct1` のキャンセルなのか `ct2` のキャンセルなのかは知る手段がないということです。  
この問題を解決できるのが、Go1.20で追加された `context.WithCancelCause` です。先ほどのコードを改変して、キャンセルの原因となる時点を特定できるようにしてみましょう。
```go
func main() {
	ctx0 := context.Background()	// コンテキストの生成
	ctx1, cancel := context.WithCancelCause(ctx0)
	// ...何らかの処理
	go process1(ctx1)
	// ...何らかの処理
	if true { // 場合によってキャンセルする
		cancel(errors.New("main"))
	}
	time.Sleep(5*time.Second)
}

func process1(ctx context.Context){
	ctx2, cancel := context.WithCancelCause(ctx)
	// ...何らかの処理
	go process2(ctx2)
	// ...何らかの処理
	if false { // 場合によってキャンセルする
		cancel(errors.New("process1"))
	}
}

func process2(ctx context.Context){
	for {
		select {
		case <-ctx.Done():
			fmt.Printf("context canceled by %s\n", context.Cause(ctx).Error())
			return 
		default:
			// ...何らかの処理
		}
	}
}
```
`context.WithCancel` の代わりに `context.WithCancelCause` を用います。すると、`cancel` 関数に引数として `error` を渡すことができます。`process2` では、`context.Cause` 関数を使って原因となったキャンセルの `error` を取り出す事ができます。  
上記のコードを実際に手元で実行してみると `context canceled by main` が出力され、コード内の `cancel` の発生位置を逆にしてみれば（`process1` でキャンセルが起こるようにすれば）、出力が `context canceled by process1` に変わることが確認できます。  
