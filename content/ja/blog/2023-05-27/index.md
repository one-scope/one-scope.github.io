---
title: "【Go1.19】url.URL に JoinPath が追加されました"
linkTitle: "【Go1.19】url.URL に JoinPath が追加されました"
date: 2023-05-27T00:00:00+09:00
description: >
  Go1.19で net/url に追加された JoinPath について紹介します。
tags:
  - golang
---

ベースURLを設定値としてもっておいて、そこにパスを結合して実際のリクエストURLを組み立てる。よくある話だと思います。  
Go1.19でこの処理をより便利に書けるようになったので紹介します。

## TL;DR
Go1.19以降、リクエストURLを組み立てるときは`JoinPath`メソッドを使いましょう。

```go
baseURL, _ := url.Parse("https://localhost:8080/")
requestURL := baseURL.JoinPath("api/users")
fmt.Println(requestURL.String())
```

## アプローチ（ダメな例）
ベースURLを`string`として持っておいて、パスを結合するための`path.Join`を使ってリクエストURLをつくろうとしています。

```go
baseURL := "https://localhost:8080/"
requestURL := path.Join(baseURL, "api/users")
fmt.Println(requestURL)
```
一見よさそうですが、うまくいきません。   
`https:/localhost:8080/api/users`  
プロトコルを指定するスラッシュふたつが正規化されて、スラッシュひとつになってしまいます。

## これまでのアプローチ
ベースURLを`url.URL`としてパースして、パス部分だけを変更し、あらためてURL文字列として組み立てる方法です。

```go
baseURL := "https://localhost:8080/"
requestURL, _ := url.Parse(baseURL)
requestURL.Path = path.Join(requestURL.Path, "api/users")
fmt.Println(requestURL.String())
```
うまくいきます。  
`https://localhost:8080/api/users`

しかし、面倒です。  
パスを組み立てるたびに毎回`url.Parse`のエラー処理を考えなければいけないです。  
だからといってベースURLを`url.URL`として保持するとパスを組み立てるたびに`url.URL`のコピーが必要となり、ディープコピーできるかどうかなど考える必要がありそれも面倒です。

## Go1.19以降のアプローチ
ベースURLは`url.URL`で保持しておきます。  
Go1.19で`url.URL`に追加された`JoinPath`メソッドで、パスを追加した新たな`url.URL`をつくります。

```go
baseURL, _ := url.Parse("https://localhost:8080/")
requestURL := baseURL.JoinPath("api/users")
fmt.Println(requestURL.String())
```
うまくいきます。  
`https://localhost:8080/api/users`  
パスを組み立てるたびにエラー処理を考える必要もないですし、`url.URL`のコピーも不要です。

ちなみに、`string`のベースURLに直接パスを結合する`url.JoinPath`も提供されています。  
こちらは内部で`url.Parse`をしているので第２戻り値として`error`を返します。従って、パスを組み立てるたびにエラー処理を考える必要があります。
```go
baseURL := "https://localhost:8080/"
requestURL, _ := url.JoinPath(baseURL, "api/users")
fmt.Println(requestURL)
```
