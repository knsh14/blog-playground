---
title: "Textlint Reviewdog"
date: 2019-09-27T12:30:29+09:00
draft: true
---

この記事では[reviewdog]( https://github.com/reviewdog/reviewdog )と[textlint]( https://github.com/textlint/textlint )を組み合わせてGitHub Actionsで利用する方法を説明します。

# モチベーション
技術書典のRe:Viewで書籍を書くためのサンプルリポジトリでは、textlintとreviewdogによって誤字や、不適切な表現をチェックしています。

私は文章力を向上させるためにこのブログを更新しています。
そのため、技術書典と同等のレベルで内容のチェックをすることは非常によい訓練になります。
TechBoosterが公開している[Re:VIEW Template]( https://github.com/TechBooster/ReVIEW-Template )ではCircleCIを利用してPull Requestにコメントを付けています。
しかし私はこのブログをデプロイするためにGitHub Actionsを利用しています。
そこでtextlintのチェックもGitHub Actionsで行えるように設定しました。

口語に近い文章を書いてしまってもすり抜けてしまう箇所もあるため、完璧ではありません。

# textlintのインストール

初めにnpmをインストールします。
私は[nodebrew]( https://github.com/hokaccha/nodebrew )を利用してインストールしました。

次に`npm init`コマンドでモジュールを定義します。
このコマンドを実行して質問に答えていくと`package.json`が作成されます。

`npm install`コマンドでtextlintとtextlintのためのプラグインなどをインストールします。

```
npm install textlint textlint-filter-rule-comments textlint-filter-rule-whitelist textlint-plugin-review textlint-rule-max-ten textlint-rule-preset-ja-technical-writing textlint-rule-preset-japanese textlint-rule-prh
```

`.textlintrc` や設定ファイルなどは[ReVIEW-Template]( https://github.com/TechBooster/ReVIEW-Template )から持ってきます。
この設定ファイルを使うことで技術書典と同じレベルでチェックできます。

# workflowの設定
リポジトリのルートに `.github/workflow` ディレクトリを作成します。
このディレクトリに `textlint.yaml` を作成し、次のワークフローを設定します。
```yaml
name: textlint

on:
  pull_request:
    branches:
    - master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - name: Use Node.js
      uses: actions/setup-node@v1
      with:
        node-version: 12.x
    - name: npm install
      run: |
        npm install
    - name: install reviewdog
      run: |
          curl -sfL https://raw.githubusercontent.com/reviewdog/reviewdog/master/install.sh| sh -s  v0.9.13
    - name: lint
      env:
        REVIEWDOG_GITHUB_API_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      run: |
          $(npm bin)/textlint -f checkstyle content/posts/*.md | ./bin/reviewdog -f=checkstyle -name=textlint -reporter=github-pr-review
```

masterにむけたPull Requestが更新されるたびにworkflowが実行されるようにしています。
`npm install`を毎回キャッシュを使わずにインストールしていますが、現時点で実行時間が30秒程度なのであまり問題になっていません。


