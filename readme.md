# はじめに

データベースの設定はそれぞれ個人で違う設定になります。まずはその設定をしてください。
データベースのパスワードやデータベース名、ユーザー名などはそれぞれ好きなものを入力してください

```:node_server/db_app/env
# node_server/db_app/env

MYSQL_ROOT_PASSWORD=root_password       # rootユーザーのパスワード
MYSQL_DATABASE=sample_db                # データベース名
MYSQL_USER=my_user　　　　　　　　　　　# ユーザー名
MYSQL_PASSWORD=aaaaaa                   # 上のユーザーのパスワード

```

次に上で設定した内容に合わせるようにphpmyadminの設定をしましょう。

```:node_server/docker-compose.yml
# node_server/docker-compose.yml

[...]

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    restart : always
    ports:
      - 8080:80
    environment:
      - PMA_ARBITRARY=1
      - PMA_HOST=db_app
      - PMA_USER=my_user               # envファイルでMYSQL_USERで設定したもの 
      - PMA_PASSWORD=aaaaaa            # envファイルでMYSQL_PASSWORDで設定したもの

[...]

```

これで設定は終了です。実際に使って見ましょう。
docker-compose.ymlファイルのあるディレクトリで以下のコマンドを実行してみましょう。

```shell-session
$ sudo docker-compose build
$ sudo docker-compose up -d
```

これですべてのコンテナが起動できるはずです。

試しに
http://localhost:8080/
を開いて見てください。phpmyadminを開くことができているはずです。

# Express.jsを試してみる

まずはコンテナの中に入りましょう。

```shell-session:
$ sudo docker ps -a    # コンテナを確認
$ sudo docker exec -it node_server_db_app_1 /bin/bash
```

コンテナに入ることができたらかんたんなサーバーを作成してみます。

```
root@2f8cae89f1d7:/# cd /myapp                      # ローカルと共有されているディレクトリに移動
root@2f8cae89f1d7:/# npm init                       # 全部デフォルトでOK
root@2f8cae89f1d7:/# npm install express --save     # expressのインストールと依存関係リスト追加
```

次にローカルのディレクトリ/node_server/express_app/myappにapp.jsという名前でファイルを作成し、
以下のように編集してみましょう。

```javascript:/node_server/express_app/myapp/app.js
// node_server/express_app/myapp/app.js

const express = require('express')
const app = express()

app.get('/', (req, res) => res.send('Hello World!'))

app.listen(3000, () => console.log('Example app listening on port 3000!'))

```

ローカル側で作成したこのファイルはコンテナ側にも反映されるはずです。
これを実行してみましょう。

```shell-session
$ cd /myapp
$ node app.js
```