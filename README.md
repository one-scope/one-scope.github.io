# one-scope.github.io

OneScope コミュニティサイト

## 環境構築

### 必要環境

- [Visual Studio Code](https://code.visualstudio.com/)
- [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
- [Docker](https://docs.docker.com/)

### 環境構築

- このリポジトリ([https://github.com/one-scope/one-scope.github.io](https://github.com/one-scope/one-scope.github.io))をワークスペースとして Visual Studio Code を起動する
- コマンドパレット(Ctrl + Shift + P)から `Dev Containers: Reopen in Container` を実行する

**初回のみ** devcontainer 内で以下のコマンドを実行する。
```
$ npm install
```

### サイトプレビュー用サーバーの起動
```
$ hugo server
```

コマンドを実行後、http://localhost:1313/ にアクセスするとサイトが表示され、手元の更新が自動で反映される。  
(参考：hugoについて https://gohugo.io/documentation/)

## Blog記事の追加方法

### 流れ
`main` ブランチからブログ記事用のブランチ(例：`taro/blog`)を作成し、ブログを書く。  
ブログが書けたら push して`main`ブランチに向けてプルリクエストを作成する。  
プルリクエストが承認されたらマージできる。

### 記事を書く場所
`content/ja/blog` の中に `2000-01-01` のように日付を名前にしたディレクトリを作り、その中にブログ本体（マークダウン）を配置する。  
