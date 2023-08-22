---
title: "【Go1.21】新しいビルトイン関数clearが追加されました "
linkTitle: "【Go1.21】新しいビルトイン関数clearが追加されました "
date: 2023-08-07T00:00:00+09:00
description: >
  Go1.21でビルトイン関数として追加されたclear関数について紹介します。
tags:
  - golang
---

## 概要
`clear` 関数は、`map` 型、 `slice` 型または型パラメータを引数として取り、すべての要素を削除するまたはゼロ値にする関数です。

### `map` 型への適用
`map` 型へ `clear` を適用すると、すべての要素が削除されます。

- 例
```go
  m := map[string]int{"Alice": 10, "Bob": 14, "Cathy": 9}
  fmt.Println(m)
  fmt.Println("clear適用前の長さ：" + strconv.Itoa(len(m)))
  clear(m)
  fmt.Println(m)
  fmt.Println("clear適用後の長さ：" + strconv.Itoa(len(m)))
```
output
```
map[Alice:10 Bob:14 Cathy:9]
clear適用前の長さ：3
map[]
clear適用後の長さ：0
```

### `slice` 型への適用
`slice` 型へ `clear` を適用すると、すべての要素がゼロ値に置き換わります。

- 例１
```go
  s := []int{2, 4, 5, 1, 3}
  fmt.Println(s)
  fmt.Println("clear適用前の長さ：" + strconv.Itoa(len(s)))
  clear(s)
  fmt.Println(s)
  fmt.Println("clear適用後の長さ：" + strconv.Itoa(len(s)))
```
output
```
[2 4 5 1 3]
clear適用前の長さ：5
[0 0 0 0 0]
clear適用後の長さ：5
```

- 例２
```go
  s := []string{"a", "c", "e", "b", "d"}
  fmt.Println(s)
  fmt.Println("clear適用前の長さ：" + strconv.Itoa(len(s)))
  clear(s)
  fmt.Println(s)
  fmt.Println("clear適用後の長さ：" + strconv.Itoa(len(s)))
```
output
```
[a c e b d]
clear適用前の長さ：5
[    ]
clear適用後の長さ：5
```
例２のように文字列スライスに対して適用した場合、結果は「 `clear` 適用前と同じ長さで要素がすべて空文字列のスライス」となる事に注意してください。
`clear` 適用後のoutputの3行目は一見長さ0のスライスにも見えますが長さ5です。
