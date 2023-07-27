---
title: "【Go1.21】slices / mapsパッケージが追加されます "
linkTitle: "【Go1.21】slices / mapsパッケージが追加されます "
date: 2023-07-27T00:00:00+09:00
description: >
  Go1.21で追加される予定のslices / mapsパッケージについて紹介します。
tags:
  - golang
---

Go1.21で、任意の型のスライスとマップについて一般的な操作ができるパッケージが追加される予定です。本記事ではその中からいくつか便利な関数を紹介します。

## slicesパッケージ
### スライスの等値判定
スライスは、`==`演算子で等しいかどうかを調べることはできません。2つのスライスが等しいかどうかを判定するには、`slices.Equal`関数が使えます。
```go
	slice1 := []int{4, 2, 1, 5, 3}
	slice2 := []int{4, 2, 1, 5, 3}
	slice3 := []int{5, 2, 1, 2, 3}

	fmt.Println(slices.Equal(slice1, slice2))
	fmt.Println(slices.Equal(slice1, slice3))
```
output
```
true
false
```

### スライスのソート
`slices.Sort`関数が使えます。
```go
	slice1 := []int{4, 2, 1, 5, 3}

	slices.Sort(slice1)
	fmt.Println(slice1)
```
output
```
[1 2 3 4 5]
```

この他に、sortパッケージの`sort.Slice`関数を使うことでもソートすることができますが、`slices.Sort`関数の方がより高速です。これは、内部で採用しているソートアルゴリズムが改良されたことによるものです。  
また、`slices.Sort`関数は安定ではありません。すなわち、ソート前の並び順は保証されません。安定ソートを行う関数としては`slices.SortStableFunc`が用意されていますが、自分で比較関数を指定する必要があります。

### スライス内で重複する要素をまとめる
スライス内に複数回登場する要素を1つにまとめるには、`slices.Compact`関数が使えます。
```go
	slice1 := []int{0, 1, 1, 2, 3, 5, 8}

	slice1 = slices.Compact(slice1)
	fmt.Println(slice1)
```
output
```
[0 1 2 3 5 8]
```
`slices.Compact`関数は、修正した新たなスライスを返します。`slices.Compact`を行った後に、元のスライスをそのまま参照してしまうと期待した結果が得られないので注意が必要です。（以下、失敗例）
```go
	slice1 := []int{0, 1, 1, 2, 3, 5, 8}

	slices.Compact(slice1)
	fmt.Println(slice1)
```
output
```
[0 1 2 3 5 8 8]
```

### スライス内の二分探索
スライス内の二分探索も自分で書く必要はありません。`slices.BinarySearch`関数が使えます。
```go
	slice1 := []int{1, 2, 3, 4, 5}

	n, found := slices.BinarySearch(slice1, 4)
	fmt.Println("4:", n, found)
	n, found = slices.BinarySearch(slice1, 0)
	fmt.Println("0:", n, found)
```
output
```
4: 3 true
0: 0 false
```

## mapsパッケージ
### マップの等値判定
スライスと同様、マップも`==`演算子は使えません。`maps.Equal`関数を使って2つのマップが等しいか判定できます。
```go
	map1 := map[string]int{
		"Alice": 15,
		"Bob":   22,
		"Cathy": 34,
	}
	map2 := map[string]int{
		"Alice": 15,
		"Bob":   22,
		"Chris": 34,
	}
	map3 := map[string]int{
		"Alice": 15,
		"Bob":   22,
		"Cathy": 34,
	}

	fmt.Println(maps.Equal(map1, map2))
	fmt.Println(maps.Equal(map1, map3))
```
output
```
false
true
```

また、`maps.EqualFunc`関数を使うことで、値の比較関数を指定することができます。この場合、キーの比較は単純な`==`で行われます。  
以下は、値（文字列）が等しいかを調べる際に、大文字小文字を区別しないようにした上で2つのマップが等しいか判定する例です。
```go
	map1 := map[int]string{
		23: "red",
		44: "green",
		81: "blue",
	}
	map2 := map[int]string{
		23: "red",
		44: "blue",
		81: "green",
	}
	map3 := map[int]string{
		23: "RED",
		44: "green",
		81: "BLUE",
	}

	eq1 := maps.EqualFunc(map1, map2, func(v1 string, v2 string) bool {
		return strings.ToLower(v1) == strings.ToLower(v2)
	})
	eq2 := maps.EqualFunc(map1, map3, func(v1 string, v2 string) bool {
		return strings.ToLower(v1) == strings.ToLower(v2)
	})

	fmt.Println(eq1)
	fmt.Println(eq2)
```
output
```
false
true
```
このような等値判定は、slicesパッケージにも用意されています。

### マップからキーと値のペアを削除する
`maps.DeleteFunc`関数を使います。削除するかどうかを判定する関数を引数に指定し、trueとなったペアが削除されます。
```go
	map1 := map[int]string{
		23: "red",
		44: "green",
		81: "blue",
		90: "red",
	}

	maps.DeleteFunc(map1, func(key int, value string) bool {
		return k%2 == 0
	})

	fmt.Println(map1)

	maps.DeleteFunc(map1, func(key int, value string) bool {
		return v == "red"
	})

	fmt.Println(map1)
```
output
```
map[23:red 81:blue]
map[81:blue]
```