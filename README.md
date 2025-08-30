# GitLab on 自宅サーバ
自宅サーバに設置して LAN 内で利用する GitLab サーバと、外部に公開する GitLab Pages のファイル類を置いておくリポジトリです。

## docker-compose.yml の概要
### ```gitlab``` サービス
Comunity Edition の GitLab イメージを利用して GitLab サーバを建てています。
接続ポートの設定 (```port```) やデータの保存先など (```volumes```) は環境やお好みに合わせて変更してください。

```yml
gitlab:
  image: gitlab/gitlab-ce:latest
  hostname: gitlab.yukoweb.net
  container_name: gitlab_container
  restart: always
  shm_size: '256m'
  ports:
    - "8080:80" # GitLab UI
    - "2222:22" # Git over SSH
  volumes:
    - ./config:/etc/gitlab
    - ./log:/var/log/gitlab
    - ./data:/var/opt/gitlab
```

## アプリケーションの起動
ビルドの必要な Dockerfile は利用していないので、イメージを Pull してきて、コンテナをバックグラウンドで立ち上げれば OK です。

```bash
$ docker compose pull
$ docker compose up -d
```

## GitLab の初期設定
GitLab を起動して ```http://<ip_address>:<port>/``` にアクセスすると、以下のようなページが表示されます。

<div align="center">
    <img src="./images/gitlab_signin.png" width=500>
</div>

デフォルトのユーザ名は "root"、初期パスワードは ```/etc/gitlab/initial_root_password``` に記載があります。
以下のコマンドでコンテナに入って確認できます。

```bash
$ docker container exec -it gitlab_container bash
```

ログイン後、サイドバーの「Settings」→「Preferences」にある「Localization」で日本語設定も可能です。
また、「Overview」→「Users」から一般ユーザの作成などもできるようになっています。

## GitLab に SSH でログインする
SSH の設定ファイル ```~/.ssh/config``` に以下を設定します。

```config
Host gitlab
  HostName <GitLab の IP アドレス>
  User <GitLab のユーザ名>
  IdentityFile ~/.ssh/id_rsa
  Port <GitLab に設定した SSH 接続用のポート>
```

接続テストをすると、以下のようなメッセージが表示されます。

```bash
$ ssh -T git@gitlab
Welcome to GitLab, @yuko!
```

