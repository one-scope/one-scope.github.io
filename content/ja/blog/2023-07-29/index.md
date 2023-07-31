---
title: "【Go1.21】2つの値を比較するcmpパッケージ "
linkTitle: "【Go1.21】2つの値を比較するcmpパッケージ "
date: 2023-07-29T00:00:00+09:00
description: >
  Go1.21で追加される予定のcmpパッケージについて紹介します。
tags:
  - golang
---

## golangにおける2値の比較
2つの値が等価かどうか判定する際、数値型や真理値型のような値であれば単純に `==` 演算子で判定できます。  
一方、golangでは `==` とは別に `reflect.DeepEqual` というメソッドが用意されています。これを用いると、`map` 型や`slice` 型を比較できたり、ポインタ型を渡したときにその中身が等価かどうかを判定できます。  
今回紹介するcmpパッケージは、 この `reflect.DeepEqual` メソッドよりも様々な点で便利に2値の比較ができるパッケージです。

## cmpパッケージ
ここでは、簡単な構造体の等価判定を例に、cmpパッケージの利用例を見ていきましょう。  
cmpパッケージでは、`cmp.Diff` と `cmp.Equal` の2種類の関数が用意されています。`cmp.Diff` では等価判定をした結果の差分がわかりやすく表示され、`cmp.Equal` では等価かどうかをたんに真理値で返します。  
以下は、2つの構造体の等価判定を、`reflect.DeepEqual` 、`cmp.Diff` 、 `cmp.Equal` のそれぞれで行った結果です。  

ソースコード(抜粋)
```go
  type Profile struct {
    Name string
    Id   int
    Age  int
  }

  func main() {
    a := Profile{"Taro", 1003, 20}
    b := Profile{"Taro", 1003, 23}
    // fmt.Println(reflect.DeepEqual(a, b))
    // fmt.Println(cmp.Diff(a, b))
    // fmt.Println(cmp.Equal(a, b))
  }
```

`reflect.DeepEqual` での実行結果
```
false
```
`cmp.Diff` での実行結果
```
 main.Profile{
        Name: "Taro",
        Id:   1003,
-       Age:  20,
+       Age:  23,
  }
```
`cmp.Diff(x, y)` の結果は上のように、差分のある行の先頭に `-` と `+` が付いて表示されます。`-` がある行が引数 `x` に、`+` のある行が引数 `y` に対応しています。  

`cmp.Equal` での実行結果
```
false
```

### オプションを指定する
cmpによる比較では、様々なオプションを指定できます。  

- AllowUnexportedオプション  
エクスポート