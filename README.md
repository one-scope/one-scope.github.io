# one-scope.github.io

one-scope コミュニティサイト

## 環境構築
このgitリポジトリ([https://github.com/one-scope/one-scope.github.io](https://github.com/one-scope/one-scope.github.io))をクローンして、devcontainerに入る。

devcontainerに入ることができたら、**初回のみ**以下のコマンドを実行する。
```
npm install
```

## 起動
```
hugo server
```

コマンドを実行後、http://localhost:1313/ にアクセスするとサイトが表示され、手元の更新が自動で反映される。  
(参考：hugoについて https://gohugo.io/documentation/)

## Blog記事の追加方法

### 流れ
`main`ブランチから`xx(名前)/blog`ブランチ(例：`taro/blog`)を作成し、ブログを書く。  
ブログが書けたらpushして`main`ブランチに向けてプルリクエストを作成する。  
プルリクエストが承認されたらマージできる。

### 記事を書く場所
`content/ja/blog`の中に`2000-01-01`のように作成した日付を名前にしたディレクトリを作り、その中にブログ本体（マークダウン）を配置する。  
マークダウンのテンプレは`content/ja/blog/index.md`にある。