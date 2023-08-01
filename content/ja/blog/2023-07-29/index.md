---
title: "【Go1.21】2つの値を比較するcmpパッケージ "
linkTitle: "【Go1.21】2つの値を比較するcmpパッケージ "
date: 2023-07-29T00:00:00+09:00
description: >
  Go1.21で追加されたcmpパッケージについて紹介します。
tags:
  - golang
---

## golangにおける2つの値の比較
2つの値が等価かどうか判定する際、数値型や真理値型のような値であれば単純に `==` 演算子で判定できます。  
一方、golangでは `==` とは別に `reflect.DeepEqual` というメソッドが用意されています。これを用いると、`map` 型や`slice` 型を比較できたり、ポインタ型を渡したときにその中身が等価かどうかを判定できます。  
今回紹介するcmpパッケージは、 この `reflect.DeepEqual` メソッドよりも様々な点で便利に値の比較ができるパッケージです。

## cmpパッケージ
ここでは、簡単な構造体の等価判定を例に、cmpパッケージの利用例を見ていきましょう。  
cmpパッケージでは、`cmp.Diff` と `cmp.Equal` の2種類の関数が用意されています。`cmp.Diff` では等価判定をした結果の2つの値の差分がわかりやすく表示され、`cmp.Equal` では等価かどうかをたんに真理値で返します。  
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
cmpによる比較では、様々なオプションを指定できます。 オプションは `cmp.Diff(x, y, opt)` のように引数に指定します。

#### `AllowUnexported` オプション  
cmpでは、`reflect.DeepEqual` と異なり、エクスポートされていないフィールドは比較対象になりません。何もオプションを指定せずにエクスポートされていないフィールドを比較しようとすると、エラーになってしまいます。  
エクスポートされていないフィールドを許容するには、`AllowUnexported` オプションを指定します。

例
```go
type Profile struct {
	Name string
	Id   int
	age  int // エクスポートされていない
}

func main() {
	a := Profile{"Taro", 1003, 20}
	b := Profile{"Taro", 1003, 23}
	opt := cmp.AllowUnexported(Profile{})

	fmt.Println(cmp.Diff(a, b, opt))
}
```
上のコードをオプション `opt` なしで実行するとエラーになりますが、`AllowUnexported` オプションを指定することで フィールド `age` も比較対象になり正常に実行できます。

#### `Transform` オプション  
特定の型の値を加工し任意の型に変換する関数を指定して、加工後の値を比較するよう指定します。  
例えば、文字列型の値に対して大文字小文字を区別せずに判定するには、`cmp.Transform` オプションに文字列をすべて大文字（または小文字）に統一する関数を渡します。以下がその例です。

例
```go
type Profile struct {
	Name string
	Id   int
	Age  int
}

func main() {
	a := Profile{"Taro", 1003, 20}
	b := Profile{"tAro", 1003, 23}
	opt := cmp.Transformer("ToUpper", func(s string) string {  // 第一引数は出力に使われる名前
		return strings.ToUpper(s)
	})
	fmt.Println(cmp.Diff(a, b, opt))
}
```
出力結果
```
  main.Profile{
        Name: Inverse(ToUpper, string("TARO")),
        Id:   1003,
-       Age:  20,
+       Age:  23,
  }
```

出力をみると、`Name` フィールドに対して `ToUpper` を適用した結果が `TARO` になったことが示されています。先頭に `-` も `+` もついていないので、元の文字列は異なるものの加工後が等価なため差分としては検出されていないことがわかります。  
これをオプションなしで実行すると、以下のように異なる文字列として検出されていることがわかります。  

オプションを指定しなかった場合の出力結果
```
  main.Profile{
-       Name: "Taro",
+       Name: "tAro",
        Id:   1003,
-       Age:  20,
+       Age:  23,
  }
```

この他にも、出力フォーマットを自分で指定できる `Reporter` オプションなど、様々なオプションが利用できます。  
また、オプションは複数指定することができます。
