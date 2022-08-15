
---
title:alpineでDockerを利用してvuls環境を構築する
---
# alpineでDockerを利用してvuls環境を構築する

## 目的

- vulsをdockerで使いたかったんだYO

## 順序

- alpine をKMV上にインストールする
- alpine にdockerをインストールする
- セガXD社のブログ記事を参考にvuls環境を構築する
- 定期更新できるように仕込む

## 参考記事

- セガXD社のブログ記事

  https://segaxd.co.jp/blog/0ddbef566b58c27dcc63ed81a46b08082a25d07b.html

  https://segaxd.co.jp/blog/9b6b424a5b485cc58afde7cb6bbae1010113d5a7.html

## 構築の実際

### alpine のインストール

省略...

### alpine にdocker をインストール

まずリポジトリを修正する

```
# vi /etc/apk/repositories
http://dl-cdn.alpinelinux.org/alpine/v3.15/main
http://dl-cdn.alpinelinux.org/alpine/v3.15/community
http://dl-cdn.alpinelinux.org/alpine/edge/testing
```

apk でパッケージを追加、開始
```
apk add docker docker-compose
service docker start
rc-update add docker
```

### セガXD社のブログ記事を参考にdocker環境を構築する

ほぼコピペですスミマセン

フォルダを掘る
```
mkdir /opt/vuls && cd /opt/vuls
mkdir nginx && cd nginx
cd /opt/vuls
```

docker-compose.ymlをこさえる

nginxとvulsrepoについては、再起動後に使いたいので restart:alwaysフラグを立てた

あと、横着なのでvulsが参照する認証鍵のパスは /root/.ssh に変更した(あんまりよくない)
```
# vi /opt/vuls/docker-compose.yml
version: '3'
services:
  nginx:
    build: nginx
    ports:
      - "80:80"
    depends_on:
      - vulsrepo
    restart: always

  # vuls本体
  vuls:
    image: vuls/vuls
    volumes:
      - ~/.ssh:/root/.ssh:ro
      - ./:/vuls
      - ./vuls-log:/var/log/vuls

  cve:
    image: vuls/go-cve-dictionary
    volumes:
      - ./:/vuls
      - ./vuls-log:/var/log/vuls

  oval:
    image: vuls/goval-dictionary
    volumes:
      - ./:/vuls
      - ./vuls-log:/var/log/vuls

  gost:
    image: vuls/gost
    volumes:
      - ./:/vuls
      - ./vuls-log:/var/log/vuls

  go-exploitdb:
    image: vuls/go-exploitdb
    volumes:
      - ./:/vuls
      - ./vuls-log:/var/log/vuls

  go-msfdb:
    image: vuls/go-msfdb
    volumes:
      - ./:/vuls
      - ./vuls-log:/var/log/vuls

  vulsrepo:
    image: ishidaco/vulsrepo
    volumes:
      - ./results:/vuls/results/
      - ./:/vuls
      - ./vuls-log:/var/log/vuls
    ports:
      - "5111:5111"
    restart: always
```

nginx用のdockerfileをこさえる
```
# vi /opt/vuls/nginx/Dockerfile
FROM nginx
COPY ./default.conf /etc/nginx/conf.d/default.conf

# vi /opt/vuls/nginx/default.conf
server{
listen 80;
server_name localhost;

location / {
proxy_pass http://vulsrepo:5111;
}
}
```

とりあえずdocker-compose でまとめて起動することを確認する
```
/usr/bin/docker-compose -f /opt/vuls/docker-compose.yml  up -d
/usr/bin/docker-compose -f /opt/vuls/docker-compose.yml ps
```

### CVEリストの初回採取

年末年始が厄介だが、そこはよしなに(CVE/JVN)
```
# vi /opt/vuls/cvelist.sh
# cvn
for i in `seq 2002 $(date +"%Y")`; do \
/usr/bin/docker-compose -f /opt/vuls/docker-compose.yml run --rm cve fetch nvd $i \
    --dbpath /vuls/cve.sqlite3 ;done
# jvn
for i in `seq 1998 $(date +"%Y")`; do \
/usr/bin/docker-compose -f /opt/vuls/docker-compose.yml run --rm cve fetch jvn $i \
    --dbpath /vuls/cve.sqlite3; done
# oval
/usr/bin/docker-compose -f /opt/vuls/docker-compose.yml run --rm oval fetch ubuntu 18 20 22 \
    --dbpath /vuls/oval.sqlite3
/usr/bin/docker-compose -f /opt/vuls/docker-compose.yml run --rm oval fetch redhat 7 8 9 \
    --dbpath /vuls/oval.sqlite3
# gost
/usr/bin/docker-compose -f /opt/vuls/docker-compose.yml run --rm gost fetch redhat \
    --dbpath /vuls/gost.sqlite3
/usr/bin/docker-compose -f /opt/vuls/docker-compose.yml run --rm gost fetch ubuntu \
    --dbpath /vuls/gost.sqlite3

# expliotdb
/usr/bin/docker-compose -f /opt/vuls/docker-compose.yml run --rm go-exploitdb fetch exploitdb \
    --dbpath /vuls/go-exploitdb.sqlite3
# msfdb
/usr/bin/docker-compose -f /opt/vuls/docker-compose.yml run --rm go-msfdb fetch msfdb \
    --dbpath /vuls/go-msfdb.sqlite3
```

### ジョブの定期実行

予めconfig.tomlを作る
```
# mkdir /opt/vuls/repo
# vi /opt/vuls/repo/config.toml
[servers]

[servers.repo]
host            = "192.168.12.34"
port            = "22"
user            = "vuls-user"
keyPath         = "/root/.ssh/id_rsa"
```

scanしてrepoするシェルを書いておく
```
# vi /opt/vuls/scanrepo.sh
/usr/bin/docker-compose -f /opt/vuls/docker-compose.yml  run --rm vuls scan  \
    -config=./repo/config.toml
/usr/bin/docker-compose -f /opt/vuls/docker-compose.yml  run --rm vuls report \
    --ignore-unfixed  -config=./repo/config.toml
```


あとはcrontabにおまじないを書いておく
```
# crontab -e 
44      0       *       *       *       /opt/vuls/cvelist.sh
55      1       *       *       *       /opt/vuls/scanrepo.sh
```

毎朝 2:00ごろにはscan/repo の結果が参照可能なはず...webで