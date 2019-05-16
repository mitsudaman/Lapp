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




# Laravel

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

## ・ブラウザで動作確認
http://localhost:8001

## ・.envのローカルファイルとdockerのコンテナのディレクトリが紐づいてるとこの設定

        - APP_CODE_PATH_HOST=../
        + APP_CODE_PATH_HOST=../lapp/

        - DATA_PATH_HOST=~/.laradock/data
        + DATA_PATH_HOST=.laradock/data

▼Laradockが上手く動かなくて困った話

https://qiita.com/skmt719/items/a296d81150fd7319e71a

## ・コンテナから脱出 停止＆再起動
    exit
    docker-compose stop
    docker-compose up -d nginx mysql phpmyadmin redis workspace

    今回はビルドしないのでめっちゃ早いです。



# <nuxt.js>


## ・nuxt はポート3000 を使うので、あらかじめ開けておく
docker-compose.yml

      ports:
        - "${WORKSPACE_SSH_PORT}:22"
        - 3000:3000 ← これ

## ・コンテナ再起動
`docker-compose up -d nginx mysql workspace `

## ・workspace コンテナへ再アクセス。

`docker-compose exec workspace bash`

## ・テンプレートを使ってnuxtを導入
`vue init nuxt-community/starter-template nuxt`

    /var/www# vue init nuxt-community/starter-template nuxt

        Command vue init requires a global addon to be installed.
        Please run yarn global add @vue/cli-init and try again.

## ・global に追加
`yarn global add @vue/cli-init`
した後も同じエラーがでるので
`npm install -g @vue/cli-init`


## ・[再]テンプレートを使ってnuxtを導入
vue init nuxt-community/starter-template nuxt

? Project name nuxt
? Project description Nuxt.js project
? Author 

とそれぞれ聞かれるのでエンターでやり過ごす

## ・削除＆ファイル移動
    /var/www# rm package.json # laravelインストール時に生成されるやつは不要。
    /var/www# mv nuxt/package.json package.json #代わりにこっち使う。
    /var/www# mv nuxt/nuxt.config.js nuxt.config.js

## ・npm intall
    /var/www# npm install

## ・yarn intall
    /var/www# yarn install

## ・package.json 編集

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

## ・nuxt起動

    # yarn dev

## ・Nuxtへアクセス
http://localhost:3000/



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



-----------------------------------------------------

# mysql

▼Laradockで簡単アプリ作成しようとしたら、速攻でDB周り／日本語問題で躓いたけど、チャチャッと解決
https://qiita.com/yamazaki/items/d9d3c56f8058ec1e5f65

▼よく使うMySQLコマンド集
https://qiita.com/CyberMergina/items/f889519e6be19c46f5f4



(1)laradock/.envを修正

    MYSQL_VERSION=latest
    MYSQL_DATABASE=default
    MYSQL_USER=default
    MYSQL_PASSWORD=secret
    MYSQL_PORT=3306
    MYSQL_ROOT_PASSWORD=root

    ↓↓↓↓↓↓↓↓↓↓


    MYSQL_VERSION=5.7
    MYSQL_DATABASE=todo
    MYSQL_USER=mysql
    MYSQL_PASSWORD=mysql
    MYSQL_PORT=3306
    MYSQL_ROOT_PASSWORD=root


▼LaradockのMySQLに接続できなくてはまった話
https://qiita.com/dnrsm/items/4bd078c17bb0d6888647

(2) .envを反映させる

## volumeなど　削除 
docker rmi laradock_mysql
docker volume rm laradock_mysql
rm -rf ~/.lardock/data/mysql ※消えなかったので手動削除で対応

## 再ビルド
docker-compose build --no-cache mysql

## docker-compose upして、mysqlのインスタンスに入る
docker-compose exec mysql bash

## mysqlの対象のDBに入れれば想定通り
mysql -u default -p 
secret


(3) Lapp/.envを更新

    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=homestead
    DB_USERNAME=homestead
    DB_PASSWORD=secret

    ↓↓↓↓↓↓↓↓↓↓

    DB_CONNECTION=mysql
    DB_HOST=mysql
    DB_PORT=3306
    DB_DATABASE=todo
    DB_USERNAME=mysql
    DB_PASSWORD=mysql


▼Laradock+MySQL マイグレーションするまで
https://qiita.com/kengo_9990/items/4fdbfbd47b2ba32f9f35

▼Laravelの開発環境をLaradockを使って構築する
https://qiita.com/ucan-lab/items/90f74ce801618830e4fc#laradock-mysql%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8A

(4)migration いっちゃう
    root@13f4f487de87:/var/www# php artisan migrate
    Migration table created successfully.
    Migrating: 2014_10_12_000000_create_users_table
    Migrated:  2014_10_12_000000_create_users_table
    Migrating: 2014_10_12_100000_create_password_resets_table
    Migrated:  2014_10_12_100000_create_password_resets_table
    Migrating: 2019_05_13_001641_create_folders_table
    Migrated:  2019_05_13_001641_create_folders_table



--------------------------------------------------------------------

# 最初の操作
    docker-compose up -d nginx mysql
    docker-compose exec workspace bash



# おどれーたこと
・value 属性の値には old('title') の実行結果を展開しています。入力エラーがあったとき、入力値はセッションに一時的に保存されます。Laravel が提供する old 関数はそのセッション値を取得します。引数は取得したい入力欄の name 属性です。

# 気になったこと
入門Laravelチュートリアル
（7）
TaskController.php
public function edit(int $id, int $task_id, EditTask $request)
{
    ~~~
}
のところ。EditTaskクラスが定義されていないので怒られた。

use App\Http\Requests\CreateTask;
と同様に
use App\Http\Requests\EditTask;
を記述するとOK。

