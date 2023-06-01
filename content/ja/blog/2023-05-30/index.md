---
title: "【Go1.20】複数のエラーをwrapする（errors.Join / fmt.Errorf）"
linkTitle: "【Go1.20】複数のエラーをwrapする（errors.Join / fmt.Errorf）"
date: 2023-05-30T00:00:00+09:00
description: >
  Go1.20から扱えるようになった、複数のエラーをラッピングしたマルチエラーについて紹介します。
tags:
  - golang
---

## エラーを「wrap（ラップ）」するとは
エラーをラップするとは、元のエラーに追加で情報を付与し、新しいエラーとして生成することです。  
英単語の「wrap」の意味から包む、くるむというイメージをすると理解がしやすいかもしれません。

以下に簡単な例を示します。
```go
err1 := errors.New("error1")
err2 := fmt.Errorf("wrap %w", err1)
fmt.Println(err2.Error())
```
output
```
wrap error1
```
`err2`は、`err1`をラップして（くるんで）新たに生成されたエラーです。  
ラップには`fmt.Errorf`関数を使っています。`fmt.Errorf`では、フォーマット指定子に`%w`を用いて`err1`を置き換えています。  
出力を見れば、`err1`の内容を保持した（くるんだ）新たなエラー`err2`が生成できていることがわかります。

## 複数のエラーをwrapする（fmt.Errorf編）
ここからが本題になります。  
Go1.20から、複数のエラーをラップしたマルチエラーを作成できるようになりました。 
まずは前章で使用した`fmt.Errorf`を使ってマルチエラーを作成してみます。
```go
err1 := errors.New("error1")
err2 := errors.New("error2")
err3 := fmt.Errorf("wrap %w and %w", err1, err2)
fmt.Println(err3.Error())
```
output
```
wrap error1 and error2
```

基本的には１つのエラーをwrapする場合とほとんど変わっていませんね。  
`fmt.Errorf`の引数に複数のエラーを追加してマルチエラー`err3`を作成できました。  
また、`fmt.Errorf`で生成したマルチエラーの型は`*fmt.wrapErrors`型となります。

## 複数のエラーをwrapする（errors.Join編）
マルチエラーの作成にはもう一つ方法があり、それがGo1.20で追加された`errors.Join`関数です。 
```go
err1 := errors.New("error1")
err2 := errors.New("error2")
err3 := errors.Join(err1, err2)
fmt.Println(err3.Error())
``` 
output
```
error1
error2
```
`errors.Join`関数の使い方は非常に簡単で、引数にエラーをそのまま複数個並べるだけです。  
出力を見ると分かるように、エラーを出力した際は改行が入る仕様となっています。  
また、`errors.Join`で生成したマルチエラーの型は`*errors.joinError`型となります。`fmt.Errorf`で生成した場合と型が異なるため注意してください。

## マルチエラーの検査
これまでラップされたエラーを検査するのに使われていた`errors.Is`や`errors.As`が、マルチエラーにも適用できるようになりました。ここでは`errors.Is`を例にマルチエラーへの適用を取り上げてみます。

`erros.Is(err, target)`では、`err`を任意回Unwrapして得られるエラーの値が`target`に一致しているかを真理値で返します。    
マルチエラーの場合、`err`がwrapしているエラーの値が`target`に一致しているかを再帰的にすべて調べることになります。  

以下に簡単な使用例を示します。
```go
err1 := errors.New("error1")
err2 := errors.New("error2")
err3 := errors.New("error3")
err4 := errors.Join(err1, err2)

fmt.Println(errors.Is(err4,err2))
fmt.Println(errors.Is(err4,err3))
```
output
```
true
false
```

`err4`は`err2`をwrapしているので1つ目の出力は`true`になります。  
一方で、`err4`は`err3`をwrapしてはいないので2つ目の出力は`false`になります。  

最後に、少し複雑な例も示してみます。
```go
err1 := errors.New("error1")
err2 := errors.New("error2")
err3 := errors.Join(err1, err2)
err4 := errors.New("error4")
err5 := errors.New("error5")
err6 := errors.Join(err4, err5)
err7 := errors.Join(err3, err6)

fmt.Println(errors.Is(err7,err3))
fmt.Println(errors.Is(err7,err4))
```
output
```
true
true
```

`err`がwrapしているエラーの中に`target`が一致しているかの判定は、再帰的にすべて行われることに注意してください。  
2つ目の出力について、`err4`は`err7`がラップしている`err6`がラップしているため、結果は`true`になります。