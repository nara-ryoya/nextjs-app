---
title: 'Jekyllの仕組み'
date: '2021-03-05'
---


春休みにWebの勉強してみよう！と思い、友人から紹介されてJekyllでオリジナルのブログサイトを作ってみることにした。「簡単にできるよ」と言われたが、実際にテンプレートを自分でアレンジしようとすると意外と苦戦したので、メモとして残しておく。

# Jekyllの仕組み
Jekyllで自作ブログを作るにあたり、いろいろな記事を見て試行錯誤したのだが、どうにもらちがあかない。記事によって紹介されているやり方がまちまちで、「結局何をどうすれば自分の思い通りになるのか」が全く把握できなかった。

そこで、途中で方向転換して、「そもそもJekyllがどのように動いているのか」を完全に把握してしまうことに焦点を絞って試行錯誤してみた。そのうちに何となく仕組みを理解できたので、以下に書き留めておく。

## そもそもJekyllとは何なのか

Jekyllとは、カッコよくいうと「静的サイトジェネレーター」と言われるものである。具体的にいうと、「markdownを書いてコマンドを叩くと、あらかじめ用意しておいたテンプレートに合うようなhtmlとCSSを作成してくれる」というとんでもない便利ツールなのである。


## Jekyllのファイル構造(一例)
メインのディレクトリ構造は以下のようになる。

```
.
├── Gemfile
├── Gemfile.lock
├── README.md
├── _config.yml
├── _data
├── _includes
├── _layouts
├── _posts
├── _sass
├── _site
├── index.md
├── assets
│   ├── css
│   │   └── app.scss
│   └── js
│       └── app.js
├── node_modules
├── package-lock.json
└── package.json
```
おそらく、多くの記事で紹介されているように、コマンドで```jekyll new (プロジェクト名)```と叩くだけでは、「_layouts」や「_includes」というディレクトリは出てこないだろう。ただ、Jekyllのファイル生成のメインを担うのはこれらのディレクトリであり、自分でテンプレートをアレンジする際にはこれらのディレクトリを用意して変更する必要がある。

では、これらのディレクトリが用意されていない場合はどうなるのか。その場合は、ローカル内のライブラリの、特定のテーマに該当するディレクトリ内の「_layouts」や「_includes」などのディレクトリを参照してhtmlとcssを生成しているのだ。

もう少し分かりやすく説明する([こちらの記事](http://jekyllrb-ja.github.io/docs/themes/)を参考に書いた)。
以下のように、jekyllプロジェクトを立ち上げて、そのディレクトリ内に移動する。
```
jekyll new my-site
cd my-site
```

次に以下のようにコマンドを叩く。すると、ローカル内の*theme*に相当するディレクトリのパスが出力されるはずだ。*theme*にはJekyllのテーマを入れられる。Jekyllのデフォルトのテーマはminimaなので、```bundle info --path minima```となる。
```
bundle info --path theme
>>/Library/Ruby/Gems/2.6.0/gems/minima-2.5.1
```
finderでこのディレクトリに移動して見てみると、「_layouts」などのディレクトリがあるのが確認できると思う。各ディレクトリの詳細な役割は、README.mdに詳細に英語で書かれているので、ここでは各ディレクトリのおおまかな役割について書いていく。<br>

- _layouts 

    テンプレートの1番の大枠となるhtmlファイルがおかれているディレクトリ。中にはblog.html、default.htmlなどのファイルが置かれており、これらのファイル名が各レイアウト名に対応しており、markdownの冒頭でこれらのどのレイアウトを使うのかを指定する。
    例えば、index.mdの冒頭で、以下のようにしていしたら、index.htmlは_layouts内のblog.htmlテンプレートにしたがって生成されるのだ。

    ```
    ---
    layout: blog
    title: Ryoya Nara
    subtitle: 自作ブログ。
    ---
    ```

    ここで、layout以外にも、titleやsubtitleなどのを指定しているのだが、これらもhtml生成に関わる重要な変数である。それについては次の項目で説明する。<br>

- _includes

    例えば、_layoutsのdefault.htmlが以下のようになっているとする。
    ```html:default.html
    <html {% if site.fixed_navbar %} class="has-navbar-fixed-{{ site.fixed_navbar }}" {% endif %}>
    {% include head.html %}
    <body>
        {% include header.html %}
        {% unless page.hide_hero %}
            {% include hero.html %}
        {% endunless %}
    </body>
    </html>
    ```

    ここで、`{% raw %}{% include header.html %}{% endraw %}`のように埋め込まれているheader.htmlを_includesファイル内から参照してhtmlを作成している。つまり、_layouts内のテンプレートの各構成要素のさらなるテンプレートとなるファイルを保持しているのが_includesディレクトリであるわけだ。

    また、_includesの各ファイルの中で、ところどころ`{% raw %}{{ page.title}}{% endraw %}`のようにLiquidの変数が埋め込まれている箇所がある。先ほどのindex.mdの例であげた冒頭の記述は、これらの変数を指定してあげているのだ。 <br>

- _posts

    ブログの投稿のmdファイルを入れるところ。

    ```
    ---
    layout: post
    title:  "2A総括"
    date:   2021-02-26 14:15 +0900
    ---
    ```

    冒頭にこのように記述して、テンプレートに埋める変数に代入している。<br>

- _sass, assets, node_modules

    各cssを出力するためのテンプレートとなるscssファイルやcssファイル、jsファイルが置かれている(正直まだあまり仕組みが分かってない...)。<br>

- _site

    各テンプレートから作成されたhtmlやcssが出力されるディレクトリ。

- _config.yml

    各テンプレートに埋め込む変数に代入する値を決めるファイル。

## コマンドを叩いた際のファイル生成の流れ
1. _posts以下のmarkdownファイルや親ディレクトリ以下のmarkdownファイルを参照し、それに合うようなhtml, CSSファイルを作成する。

2. _site以下にhtml, CSSを出力する。

    <br><br>

## 最後に

正直最後の方は疲れてしまってだいぶ雑になってしまったが、まぁおおまかにこれくらい把握しておけば大体あとは自分でアレンジが聞くと思う。実際にいじってみるとめちゃくちゃ面白いので、もう少しきちんと勉強して頑張ってみたい。


