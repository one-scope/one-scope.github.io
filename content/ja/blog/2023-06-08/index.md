---
title: "【Go1.20】time.TimeにCompareが追加されました "
linkTitle: "【Go1.20】time.TimeにCompareが追加されました "
date: 2023-06-08T00:00:00+09:00
description: >
  Go1.20でtime.Timeに追加されたCompareについて紹介します。
tags:
  - golang
---

## 従来のtime.Timeの比較
`time.Time` 型は、Goの標準パッケージ `time` にある日時を表す構造体です。  
`time.Time` 型を比較するメソッドとして、これまでは `Before`、`After`、`Equal` が提供されていました。  
例えば、ある日時を表す `time.Time` 型の `date1` と `date2` について、`date1` が `date2` よりも過去の日時であるかの判定は、以下のように書けます。
```go
date1.Before(date2)   // date1 が date2 よりも過去の日時なら true、そうでなければ false
```

ここで、`date1` が `date2` 以前（`date1 <= date2`）であるかを判定する方法を考えます。  
前述の `Before` を使うだけでは、`date1 == date2`のときに `false` となるので、2つの日時が等しい時に `true` となる `Equal` と組み合わせて、以下のように判定できます。
```go
date1.Equal(date2) || date1.Before(date2)
```
ただ、この書き方は冗長なので、以下のように「未来でない」ことで判定することが多いです。
```go
!date1.After(date2)
```
とはいえこれも、判定内容が読み取りづらいという問題がありました。

## Compareを使ったtime.Timeの比較
Go1.20 で追加された `Compare` では、2つの `time.Time` 型を比較した結果を `int` 型の値で返します。  
具体的には、`date1.Compare(date2)` に対して、
- `date1` が `date2` より過去の日時なら、-1を返す。
- `date1` が `date2` と同じ日時なら、0を返す。
- `date1` が `date2` より未来の日時なら、1を返す。

これを用いると `date1 <= date2` の判定も、一般的な大小比較（たとえば `strings.Compare`）と同様の戻り値から判定できます。
```go
date1.Compare(date2) <= 0
```
