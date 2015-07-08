# 掲示板を公開する

[Ruby on Rails で簡単な掲示板を作る](../rails_keijiban) で作ったアプリを、以下のようにインターネットに公開してみましょう。

https://keijiban-sample.herokuapp.com/

# Step 1. Heroku のアカウントを作る

Rails のアプリケーションを公開するために、[Heroku](https://www.heroku.com/ というサービスを使います。
これは、Rails などで作られたウェブアプリケーションを公開するためのサービスです。

まず、Heroku のトップページ右上の Sigh Up から Heroku のアカウントを作ってください。

# Step 2. Heroku Toolbelt をインストールする

Heroku を使って開発するために、[Heroku Toolbelt](https://toolbelt.heroku.com/) をインストールしましょう。
これは、Heroku を使ってサービスを公開するために必要なツールです。

https://toolbelt.heroku.com/ から自分の環境にあった toolbelt をインストールしてください。

インストールが成功すると、ターミナル (コマンドプロンプト) から `heroku` というコマンドが使えるようになります。

# Step 3. heroku コマンドでログインをする

ターミナルで `heroku login` というコマンドを入力すると、メールアドレスとパスワードを求められるので、指示に従って入力しましょう。 (パスワードは表示されませんが、見えないだけで入力は受け付けているのでそのまま書き込んでください)

```
$ heroku login
Enter your Heroku credentials.
Email: adam@example.com
Password (typing will be hidden):
Authentication successful.
```

`Authentication successful.` と表示されたらログイン成功です。

# Step 4. heroku コマンドを使って掲示板用の Heroku プロジェクトを作る

前回作った掲示板のディレクトリ (フォルダ) に移動して、

```
heroku create keijiban-(あなたの好きな名前)
```

というコマンドを入力してください。

成功すると、Heroku のアカウントに掲示板プロジェクトが作成され、あなたのアプリを公開するための URL が発行されます。

# Step 5. Heroku にデータベースを用意する

実はこのままでは Heroku 上でうまく動きません。

Heroku では今まで使っていた sqlite というデータベースアプリケーションが使えず、代わりに PostgreSQL というアプリケーションを使います。

あなたのアプリ用の PostgreSQL のデータベースを用意するため、以下のコマンドを入力してください。

```
heroku addons:create heroku-postgresql:hobby-dev
```

続いて、掲示板アプリ側のコードを変更します。

`heroku config` というコマンドを入力することで、以下の様な Heroku の設定情報が表示されます。

```
=== keijiban-sample Config Vars
DATABASE_URL:             postgres://aaaa:bbbb@ec2-11-11-11-11.compute-1.amazonaws.com:2222/cccc
LANG:                     en_US.UTF-8
RACK_ENV:                 production
RAILS_ENV:                production
RAILS_SERVE_STATIC_FILES: enabled
SECRET_KEY_BASE:          xxxxx
```

ここで、我々にとって重要なのは、 `DATABASE_URL` の

```
postgres://aaaa:bbbb@ec2-11-11-11-11.compute-1.amazonaws.com:2222/cccc
```

という文字列で、これはデータベースの設定情報です。
この情報は人によって違います。

これを Rails アプリのデータベース設定ファイルである `config/database.yml` に反映させます。


`database.yml` の

```
production:
  <<: *default
  database: db/production.sqlite3
```

を次のように変更します。

```
production:
  encoding: utf-8
  adapter: postgresql
  port: 2222
  database: cccc
  host: ec2-11-11-11-11.compute-1.amazonaws.com
  username: aaaa
  password: bbbb
```

これは、 `DATABASE_URL` が `postgres://aaaa:bbbb@ec2-11-11-11-11.compute-1.amazonaws.com:2222/cccc` だったときの例です。

この文字列は次のような形式になっているので、それを `database.yml` に反映させます。

```
postgres://(username):(password)@(host):(port)/(database)
```

続いて、データベースをアプリから使うためのライブラリをインストールする設定を行ないます。

これには、 Rails アプリで使用するライブラリを設定している `Gemfile` というファイルを編集します。

ローカルで使っていた sqlite のための設定である `gem ‘sqlite3’` は `group :development, :test do` 以下に置き直してください。

これによってローカルの開発環境でのみ、 sqlite を使うようになります。

続いて、Heroku 上の環境 (production, 本番環境と呼びます) のための設定を書きます。
これは、以下のようにしてください。

```
group :production do
  gem 'pg'
  gem 'rails_12factor'
end
```

編集が終わったらファイルを保存して、 `bundle install` というコマンドを入力してください。
これで、ライブラリがインストールされます。

# Step 6. Heroku にアプリケーションを送る

続いて、アプリを Heroku にアップロードしましょう。
Heroku へのアップロードには [Git](http://git-scm.com/)  という[バージョン管理システム](https://ja.wikipedia.org/wiki/%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3%E7%AE%A1%E7%90%86%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0)を使います。

Git は Mac では標準で入っています。
Windows をお使いの方は、[ここから](http://git-scm.com/downloads)インストールしてください。

初めて Git を使う方は、まず以下のようにターミナルに打ち込んで名前とメールアドレスを登録してください。

```
git config --global user.name "your-name"
git config --global user.email "your-email"
```

`heroku create` すると Git のレポジトリ (プロジェクトのファイルを保存する場所) が作られています。

以下のコマンドを入力して、今まで作ったすべてのファイルを保存しましょう。

```
git add .
git commit -m ‘Save all files’
```

その後、`git push heroku` コマンドを入力してください。
少し時間がかかったあと、すべてのファイルが Heroku に送られます。

# Step 7. Heroku のデータベースの設定をする

これで、 https://keijiban-(あなたの好きな名前).herokuapp.com にアクセスすることができます。
しかし、おそらくはエラーメッセージが表示されることでしょう。

これは、Heroku に送ったアプリのデータベースが設定されていないことに起因します。

アプリを作った時にやったように `rake db:migrate` コマンドでデータベースを設定する必要があります。

Heroku 上でコマンドを走らせるためには `heroku run` コマンドを使います。

ここでは以下のコマンドを入力します。

```
heroic run rake db:migrate
```

これで先ほどの URL にアクセスすると、今まで手元で動いていた掲示板アプリがインターネット上で表示されるようになっているはずです。
