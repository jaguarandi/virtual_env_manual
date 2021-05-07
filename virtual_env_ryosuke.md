# 環境構築手順書

## バージョン一覧
ツール|バージョン
-|-
php|7.3
Nginx|1.19.10
MySQL|5.7.34
Laravel|6.0
CentOS|7.9.2009
<br>

## 環境構築の流れ

### 目次
  ##### vagrantディレクトリ作成〜ログイン
  - [vagrant用ディレクトリの用意](#vagrant用ディレクトリの用意)
  - [Vagrantfileの編集](#vagrantfileの編集)
  - [Vagrant プラグインのインストール](#vagrant-プラグインのインストール)
  - [VagrantでゲストOSの起動、ログイン](#vagrantでゲストosの起動-ログイン)
  ##### ツールのインストール
  - [パッケージをインストール](#パッケージをインストール)
  - [PHPのインストール](#phpのインストール)
  - [composerのインストール](#composerのインストール)
  - [Laravelのプロジェクト作成](#laravelのプロジェクト作成)
  - [Laravelの認証機能実装](#laravelの認証機能実装)
  - [操作権限の付与](#操作権限の付与)
  - [データベースのインストールとパスワード設定](#データベースのインストールとパスワード設定)
  - [データベースの作成](#データベースの作成)
  ##### システムの設定
  - [ファイヤーウォールの設定](#ファイヤーウォールの設定)
  - [SELinuxの設定](#selinuxの設定)
  - [Nginxのインストール](#nginxのインストール)
  - [Laravelを動かす](#laravelを動かす)
  ##### まとめ
- [環境構築の所感](#環境構築の所感)
- [参考サイト](#参考サイト)


### vagrant用ディレクトリの用意
自分の作業用ディレクトリ下に**vagrant_practice**という名前のディレクトリを作成<br>
作成したフォルダの中でboxを使用する。

```
$ mkdir vagrant_practice

$ cd vagrant_practice

$ vagrant init centos/7
```
<br>

### Vagrantfileの編集
**Vagrantfile**において以下の3点を編集しコメントを外す。
```
### 変更点1 コメントを外す
config.vm.network "forwarded_port", guest: 80, host: 8080

### 変更点2 ipを変更
config.vm.network "private_network", ip: "192.168.33.19"

### 変更点3 以下に編集
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```
<br>

### Vagrant プラグインのインストール
vagrant_practiceディレクトリ下で、**vagrant-vbguest**というプラグインをインストールする。
```
$ vagrant plugin install vagrant-vbguest
```
<br>

### VagrantでゲストOSの起動、ログイン
Vagrantfileが存在するディレクトリ下で、以下のコマンドで起動しログインする。
```
### 起動
$ vagrant up

### ログイン
$ vagrant ssh
```
<br>

### パッケージをインストール
ゲストOS内にログインした状態で、開発にあたって必要なソフトウェア、コマンドのパッケージをインストールする。
```
$ sudo yum -y groupinstall "development tools"
# ゲストOSにログインした状態で行う
```
<br>

### PHPのインストール
外部パッケージツールをダウンロードし、そこからPHPをインストールする<br>
今回はPHPの**バージョン7.3**をインストールする。
```
$ sudo yum -y install epel-release wget

$ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm

$ sudo rpm -Uvh remi-release-7.rpm

$ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip

$ php -v
```
<br>

### composerのインストール
次にcomposerのインストールを行う。
```
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

$ php composer-setup.php

$ php -r "unlink('composer-setup.php');"

### どのディレクトリでもcomposerを使用できるようfileの移動を行う
$ sudo mv composer.phar /usr/local/bin/composer

$ composer -v
```
<br>

### Laravelのプロジェクト作成
vagrantディレクトリまで移動後以下のコマンドを実行し、Laravelの**バージョン6.0**でプロジェクトを作成する。
```
$ cd /vagrant

$ composer create-project laravel/laravel --prefer-dist laravelApp 6.0
```
<br>

### Laravelの認証機能実装
laravel/uiパッケージを利用して、認証に必要なルートやビュー等の雛形ファイルを生成する。
```
$ cd /vagrant/laravelApp

$ cd composer require laravel/ui "^1.0" --dev

$ php artisan ui vue --auth
```
<br>

### 操作権限の付与
Laravelを動かすのにパーミッションの設定が必要なので、**storage**ディレクトリ下と**bootstrap/cache**ディレクトリをwebサーバから書き込み可能にする。
```
$ cd /vagrant/laravelApp

$ sudo chmod -R 777 storage

$ sudo chmod -R 777 bootstrap/cache
```
<br>

### データベースのインストールとパスワード設定
MySQLの**バージョン5.7**をインストールする。<br>
rpmにリポジトリを追加し、インストールを行う。
```
$ sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm

$ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm

$ sudo yum install -y mysql-community-server
```
次に初期パスワードを取得する。
```
$ sudo cat /var/log/mysqld.log | grep 'temporary password'

~ A temporary password is generated for root@localhost: hogehoge
# hogehoge: この部分のランダムな文字列が初期パスワードとなっている。
```

MySQLを起動し接続を行い、先程のパスワードを入力する。<br>
ログイン後、パスワードの変更を行う。
```
$ mysql -u root -p

$ Enter password:
mysql >

＃パスワードの変更
mysql > set password = "新たなpassword";
```
<br>

### データベースの作成
Laravelのアプリケーションで使用するデータベースを作成する。
```
mysql > create database laravel_app;
```

laravelAppディレクトリ下の.envファイルを編集する。
```
$ sudo vi /vagrant/laravelApp/.env

### .envファイル内
#----------------------
### DB_DATABASEを作成したデータベース名に変更
DB_DATABASE=laravel_app

### DB_PASSWORDを登録したパスワードに変更
DB_PASSWORD=登録したパスワード
#----------------------
```

laravelAppディレクトリに移動してmigrateを実行する。
```
$ cd /vagrant/laravelApp

$ php artisan migrate
```
<br>

### ファイヤーウォールの設定
ファイヤーウォールに対して、Vagrantfileで記述されているゲストのポート(ここではguest:80)を経由したhttp通信によるアクセスを許可する。
```
### ファイヤーウォールの起動
$ sudo systemctl start firewalld.service

$ sudo firewall-cmd --add-service=http --zone=public --permanent

### ファイヤーウォールに反映
$ sudo firewall-cmd --reload
```
<br>

### SELinuxの設定
viエディタを使用してSELinuxの設定を変更する。
```
$ sudo vi /etc/selinux/config
```

下記の部分を**SELINUX=disabled**に書き換えて無効にする。
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
SELINUX=disabled
```
<br>

### Nginxのインストール
viエディタを使用して以下のファイルを作成する。
```
$ sudo vi /etc/yum.repos.d/nginx.repo

### 書き込む内容
#----------
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
#----------
```

内容を保存後、Nginxのインストールを行い、起動する。
```
$ sudo yum install -y nginx

$ nginx -v

$ sudo systemctl start nginx
```
<br>

### Laravelを動かす
Nginxの設定ファイルを編集する。
```
$ sudo vi /etc/nginx/conf.d/default.conf

server {
  listen       80;
  server_name  192.168.33.19; # Vagrantfileでコメントを外した箇所のipアドレスを記述
  # ApacheのDocumentRootにあたります
  root /vagrant/laravelApp/public; # 追記
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  # 省略

  # 該当箇所のコメントを解除し、必要な箇所には変更を加える
  # 下記は root を除いたlocation { } までのコメントを解除

  location ~ \.php$ {
  #    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
      include        fastcgi_params;
  }

  # 省略
```

次に**php-fpm**の設定ファイルを編集する。
```
$ sudo vi /etc/php-fpm.d/www.conf

# 変更箇所は以下の通り
;24行目近辺
user = apache
# ↓ 以下に編集
user = vagrant

group = apache
# ↓ 以下に編集
group = vagrant
```

設定を反映させるためにゲストOSを再起動する必要があるので、一度ログアウトして以下のコマンドを実行する。
```
$ exit

$ vagrant reload

$ vagrant ssh

### nginx、php-fpmの起動
$ sudo systemctl start nginx
$ sudo systemctl start php-fpm
```
<br>

## 環境構築の所感
- 今回のserver lessonでは仮想環境の構築を行うという性質上、今までのカリキュラムと比べてコマンドでの操作が多かった為、今どこでで何の操作を行っているのかを一つ一つ把握しながら作業を進めていくことが必要だと感じた。<br>

- 一言で仮想環境の構築といっても、OS上に仮想OSを立ち上げるホスト型やコンテナという単位で環境を立ち上げるコンテナ型など方法が複数存在し、開発の際には目的や用途によって使い分けているということが理解できた。
<br>
<br>

## 参考サイト
Vagrantで共有フォルダのエラーがでるのでその対応 yk 5656 diary
<br>
https://yk5656.hatenablog.com/entry/20201202/1609916685

Vagrant No VirtualBox Guest Additions installation found Fixed DevopsRoles.com
<br>
https://www.devopsroles.com/vagrant-no-virtualbox-guest-additions-installation-found-fixed/

CentOS7のPHPを5.6／7.0／7.1／7.2／7.3系にバージョンアップする - Qiita
<br>
https://qiita.com/heimaru1231/items/84d0beca81ca5fdcffd0

CentOS7にComposerをインストールする - Qiita
<br>
https://qiita.com/inakadegaebal/items/d370bcb1627fce2b5cd1

laravel & vue.js エラー解消方法 - Qiita
<br>
https://qiita.com/naka46/items/e562e38764441d2b5b4a

マークダウン記法 一覧表・チートシート - Qiita
<br>
https://qiita.com/kamorits/items/6f342da395ad57468ae3

インストール 6.x Laravel
<br>
https://readouble.com/laravel/6.x/ja/installation.html#configuration

認証 6.x Laravel
<br>
https://readouble.com/laravel/6.x/ja/authentication.html

きれいな手順書をつくるための7つのポイント | Enjoy IT Life
<br>
https://nishinatoshiharu.com/tidy-procedure/#7