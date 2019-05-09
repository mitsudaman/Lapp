#
## \<laravel>


# 最終的な目標
・docker で サービスを稼働する
・複数サービスを稼働するか否か　
　 → 複数サービスはまだ先　
　 → 分けたほうが管理しやすい
　 → MVPで行こう！　まずは勉強
・laradock使うか
→ 





▼公式
http://laradock.io/


    1 - Clone Laradock inside your PHP project:

    git clone https://github.com/Laradock/laradock.git

    2 - Enter the laradock folder and rename env-example to .env.
    (cd laradock)
    cp env-example .env

    3 - Run your containers:

    docker-compose up -d nginx mysql phpmyadmin redis workspace 

    4 - Open your project’s .env file and set the following:

        DB_HOST=mysql
        REDIS_HOST=redis
        QUEUE_HOST=beanstalkd

    5 - Open your browser and visit localhost: http://localhost.

    That's it! enjoy :)

ここら辺でこけたりすることもあるのでその場合は一度docker内をクリーンな状態にして再度試す。
    - イメージを全削除する
    docker images -aq | xargs docker rmi

    - 未使用ボリューム削除
    docker volume prune

    - 未使用ネットワーク削除
    docker network prune

ちなみに2019/01/09時点でこの手順を試すとエラーになった。
https://github.com/laradock/laradock/issues/1940

原因はGithubに上がっているソース自体に問題があったため。
・Github上のissuesで同じ問題が起きている人がいないか
・Pull request　で問題を解消するためのコードを提案している人がいないか
を確認し、いる場合は大人しく待つ

※2019/01/10時点でpull requestがマージされて解消されています。

このように元からエラーになる場合はいくらdockerをクリーンな状態にしてやり直しても同じ結果になるので原因をしっかり追求する(githubを読む)ことをオススメします。

## (1)コンテナ起動
    docker-compose up -d nginx mysql phpmyadmin redis workspace 

## (2)コンテナ内に入る
    docker exec -it laradock_workspace_1 /bin/bash
 
 or 

    docker-compose exec workspace bash

## (3)プロジェクト作成

/var/www# composer create-project laravel/laravel Lapp

composer create-project laravel/laravel Lapp

すると初めに
Do not run Composer as root/super user! See https://getcomposer.org/root for details
と黄色文字で出るが問題ないですしばし待ちましょう

## (4)NGINXのportを変える

    - NGINX_HOST_HTTP_PORT=80
    + NGINX_HOST_HTTP_PORT=8001

### ・ブラウザで動作確認
http://localhost:8001

### ・.envのローカルファイルとdockerのコンテナのディレクトリが紐づいてるとこの設定

        - APP_CODE_PATH_HOST=../
        + APP_CODE_PATH_HOST=../lapp/

        - DATA_PATH_HOST=~/.laradock/data
        + DATA_PATH_HOST=.laradock/data

▼Laradockが上手く動かなくて困った話

https://qiita.com/skmt719/items/a296d81150fd7319e71a

### ・コンテナから脱出 停止＆再起動
    exit
    docker-compose stop
    docker-compose up -d nginx mysql phpmyadmin redis workspace

    今回はビルドしないのでめっちゃ早いです。
    

## laravelは起動した！
## nuxt.js を　導入してほしそうにこっちを見ている
## 導入しますか？　
## →はい　

#

## <nuxt.js>


### ・nuxt はポート3000 を使うので、あらかじめ開けておく
docker-compose.yml

      ports:
        - "${WORKSPACE_SSH_PORT}:22"
        - 3000:3000 ← これ

### ・コンテナ再起動
`docker-compose up -d nginx mysql workspace `

### ・workspace コンテナへ再アクセス。

`docker-compose exec workspace bash`

### ・テンプレートを使ってnuxtを導入
`vue init nuxt-community/starter-template nuxt`

    /var/www# vue init nuxt-community/starter-template nuxt

        Command vue init requires a global addon to be installed.
        Please run yarn global add @vue/cli-init and try again.

### ・global に追加
`yarn global add @vue/cli-init`
した後も同じエラーがでるので
`npm install -g @vue/cli-init`


### ・[再]テンプレートを使ってnuxtを導入
vue init nuxt-community/starter-template nuxt

? Project name nuxt
? Project description Nuxt.js project
? Author 

とそれぞれ聞かれるのでエンターでやり過ごす

### ・削除＆ファイル移動
    /var/www# rm package.json # laravelインストール時に生成されるやつは不要。
    /var/www# mv nuxt/package.json package.json #代わりにこっち使う。
    /var/www# mv nuxt/nuxt.config.js nuxt.config.js

### ・npm intall
    /var/www# npm install

### ・yarn intall
    /var/www# yarn install

### ・package.json 編集

    {
        "name": "nuxt",
        ~ 省略 ~
        "scripts": {
            "dev": "nuxt",                                                  <- before
            "dev": "HOST=0.0.0.0 PORT=3000 node_modules/nuxt/bin/nuxt",     <- after
            "dev": "HOST=0.0.0.0 PORT=3000 nuxt",                           <- after2
            "build": "nuxt build",
            "start": "nuxt start",
            "generate": "nuxt generate",
            "lint": "eslint --ext .js,.vue --ignore-path .gitignore .",
            "precommit": "npm run lint"
        },
        ~ 省略 ~
    }

### ・nuxt起動

    # yarn dev

### ・Nuxtへアクセス
http://localhost:3000/

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~





▼ローカルにある Laravel を公開サーバーにデプロイ
https://laraweb.net/environment/3192/



▼ConoHaを使ってLaravelを超絶簡単にインストールしてみた

https://www.youtube.com/watch?v=oBJFCTu7H1s


ssh root@118.27.4.161

cd /var/www/html/laravel

[root@118-27-4-161 laravel]# ls
app        composer.json  database      public     routes      tests
artisan    composer.lock  package.json  readme.md  server.php  vendor
bootstrap  config         phpunit.xml   resources  storage     webpack.mix.js
[root@118-27-4-161 laravel]# 



-----------------