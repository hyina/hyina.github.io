---
layout: post
title: "Octopressはじめました"
date: 2013-10-04 20:38
comments: true
categories: memo
---
  

### はじめに
***
個人的なoctpressをはじめるためのメモ書きです。  
  
### 個人的環境
---
* Mac OS X Lion  
* ruby 2.0.0p247  
* git 1.8.4  
  
<!-- more -->
  
### 導入
---
[公式サイト](http://octopress.org/docs/setup/)を見てコマンド実行するだけ。
rubyのバージョンは1.9.3以上が必要なので注意。
`.rbenv-version`ってファイルをイジると問題ないような気もする。やってないけど(笑´∀｀)  
あとは、いつからか、githubのリポジトリをusername.github.com から username.github.io とするようになっているみたい。
  
デフォルトのテーマのままっていうのもあれなので、[サードパーティー制のテーマ](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes)から、bootstrap-themeを適用しました。  
下記のコマンドを実行でOK。テーマを適用するとサイドバーとかの修正した分の一部が初期化されるので、テーマを適用してからイジると良いかと思います。    
  
```
cd octopress
git clone https://github.com/bkutil/bootstrap-theme .themes/bootstrap
rake install['bootstrap']
rake generate
```
  

### ブログの設定
---
_config.ymlの編集すると、ブログタイトルなどは_config.ymlで設定できます。  
titleやauthor等を編集すると、ヘッダーやフッターが自動的に変わってくれます。  
内容については、ググるなり、[公式サイト](http://octopress.org/docs/configuring/)を見ると色々書いてありますが、下記くらいは修正しておくと良いかと思います。  
  
```
url: http://yoursite.com
title: My Octopress Blog
subtitle: A blogging framework for hackers.
author: Yout Name
simple_search: http://google.com/search
description:
date_format: "%Y-%m-%d" 
excerpt_link: "続きを読む" 
```
  
あと、ブログなのにコメント欄がないというのもあれなので、ブログのコメントサービス[Disqus](http://disqus.com/)を設定しときます。  
Disqusでユーザ登録する際に、ユーザIDを設定できます、それを、`_config.yml`に追記します。  
```
#Dispus account
disqus_short_name: USER_ID
disqus_show_comment_count: false
```
すると、`source/_include/disqus.html`によってdisqus用のコードを生成してくれます。  
    
### プレビュー
---
ここらで一度、どんな感じになっているか確認してみます。  
プレビューを行うには、下記のコマンドを実行後、[localhost:4000](http://localhost:4000)にアクセスします。  
  
```
$ rake preview
```  
プレビューを終了するには、ctrl+cでｗ
  

### デプロイ
---
OctopressをGithub Pagesにデプロイするための設定をします。  
  
```
rake setup_github_pages
```  
コマンドを叩くと、Github PagesのリポジトリのURLを入力してねと出るので、
リポジトリのURLを入力しましょう。  

入力したら、下記コマンドを入力するとデプロイできます。しばらくすると、`http://onigra.github.io/`にブログができあがります。反映には5〜10分くらいかかるみたいです。  
  
```
rake gen_deploy
```  
  

### 記事の作成
---
記事を作成するには、下記のコマンドを実行し、テンプレートを作成します。  
post titleは、実際のURLになるので、注意。
  
```
rake new_post['post title']
```
テンプレートは、`source/_posts/`以下に出来るので、お好みのエディタで編集します。  
```
---
layout: post
title: "ここにタイトルを記入"
date: 作成日が入っています。
comments: コメント欄を出す場合は、trueに。
categories: タグ
---
記事を書く。
下記を入れておくと、トップページではmore以下が省略され、続きを読むリンクが表示されます。
リンク文字列は、_config.yml の excerpt_link で変更可能です。

# 上の内容までがトップページに表示 #
<!-- more -->
# 下の内容が個別の記事のページに表示 #

```
  
Markdownで書いたものをリアルプレビューできるMouが有名みたいですが、僕は、Sublime Text 2で書いています。意味は特にないですｗ  
  

### Sublime Text 2でMarkdownのプレビューをする
---
需要があるかは不明ですが、忘却メモということで(・∀・)  
1. Ctrl + Shift + p  
2. Package Controll:Install Packages  
  
その後、「Markdown Preview」をインストールしたら使えます。  
[ここ](https://github.com/revolunet/sublimetext-markdown-preview#usage-)に使い方は書いてあるけど、Markdownのファイルを開いた状態で、  

1. Ctrl+Shift+p  
2. Markdown Preview : preview in Browser  
  
と実行すると、ブラウザで開かれてプレビューを見ることができます。
`/var/folders/`にファイルが作成されてるみたい…勝手に削除されるのかな？  
  
  
### htmlファイル生成
---
下記のコマンドを実行すると、htmlファイルが生成されます。  
```
rake generate
```
そのあとは、プレビューで確認して、最終確認をします。  
  

### 記事の投稿
---
プレビューで公開をしても問題ないとなったら、下記のコマンドを実行してデプロイします。  
```
rake deploy
```  

ちなみに、html生成＋デプロイは、下記のコマンド１行でいけます。

```
rake gen_deloy
```
  
あと、記事の削除は、`source/_posts/〜.markdown`を削除し、html生成+デプロイをすると、ブログからは消え去ります。  
  
  
### 感想
---
rubyを触ったことないからインストールから始めたので意外と時間はかかっちゃいました。あとは、テーマやらサイドバーとかの修正で手間取ったりorz  
とりあえず、markdownでブログを書けるのはかなり楽なのでやって損はないと思います。  
問題は、ブログを書き続ける継続力があるかどうか・・・かな(；・∀・)←
  


