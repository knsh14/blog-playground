---
title: "HugoのデプロイをGitHub Actionsで行う"
date: 2019-09-14T01:01:53+09:00
draft: false
---

HugoとGitHub Pagesを利用してブログや自分のページをデプロイしてる方は大勢います。  
自分も最近ブログを書こうと思い立ってHugoでブログを作ろうと考えました。  
せっかくなのでブログを書いてPRを投げてmasterにマージされたら自動で静的ファイルを作成して公開する仕組みが欲しくなりました。  
なので、GitHub Actionsを使って作ってみることにしました。

# 事前準備
* 記事を書く用のリポジトリ
* デプロイ先のリポジトリ
* GitHubのPersonal Access Token
    * repoの操作が可能な権限を付ける必要があります


記事を書く用のリポジトリはPrivateでも構いません。  
デプロイ先のリポジトリは`USERNAME.github.io`という名前にする必要があります  

# 記事のリポジトリにトークンを設定する
記事を書く用のリポジトリに事前準備で作成したPersonal Access Tokenを設定します。  
リポジトリのSettings -> Secretsを開き、「Add a new secret」を押して、Nameを「MY_GITHUB_ACCESS_TOKEN」Valueに先程のトークンを設定します。
この設定を入れないと、別のリポジトリにアクセスする権限を持てないので必ず設定する必要があります。  
詳しいドキュメントは[本家のドキュメント]( https://help.github.com/en/articles/virtual-environments-for-github-actions#creating-and-using-secrets-encrypted-variables )を参照してください。

# 記事のリポジトリにworkflowを設定する
記事のリポジトリのルートに移動して、`.github/workflows`ディレクトリを作成します。  
その中に次のようなワークフローを定義します。

```yaml
name: Go
on:
  push:
    branches:
    - master
jobs:

  build:
    name: Build
    runs-on: ubuntu-latest
    steps:

    - name: Set up Go 1.12
      uses: actions/setup-go@v1
      with:
        go-version: 1.12
      id: go

    - name: Check out code into the Go module directory
      uses: actions/checkout@v1

    - name: install hugo
      run: |
        go get -u github.com/gohugoio/hugo

    - name: setup public repo
      run: |
          git clone --depth 1 https://USERNAME:${{ secrets.MY_GITHUB_ACCESS_TOKEN }}@github.com/USERNAME/USERNAME.github.io.git public

    - name: Build
      run: |
        export PATH=$PATH:`go env GOPATH`/bin
        git submodule init
        git submodule update
        hugo

    - name: Push
      run: |
        export MSG=`git log --format=%B -n 1 HEAD`
        cd public
        git config --local user.name "NAME"
        git config --local user.email "EMAIL@gmail.com"
        git add .
        git commit -m "${MSG}"
        git push origin master
```

Goをベースにしたワークフローを利用します。
setup public repoのステージでHugoをビルドした結果の静的ファイルが吐き出されるディレクトリを公開用のリポジトリで管理できるようにします。  
Buildステージで環境変数のPATHにGoのバイナリのある場所も追加してHugoコマンドが使えるようにします。  
Pushステージで記事用のリポジトリのコミットメッセージを取得して公開用のリポジトリのコミットメッセージにします。  
以上で設定が完了したので記事用のリポジトリのmasterにpushするとActionsが起動して公開用のリポジトリに送信されます。

このブログもこのようにして公開されています。
GitHub Actionsを利用するとCircle CIなどに別で登録したり確認しに行く作業がなくなります。
GitHubだけで作業が完結できるのでかなり便利だなと感じました。
