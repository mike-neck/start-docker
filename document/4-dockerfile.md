もっとDockerfile
===

このドキュメントではより実践的なDockerfileの書き方や、`docker`コマンドの使い方が記述されています。

`docker`コマンド
===

`Dockerfile`の前に、`docker`コマンドを覚えていきます。

### ubuntu

ubuntuのイメージ上で`echo`コマンドを実行します。

```
docker run ubuntu /bin/echo Hello, from Ubuntu
```

`docker run image-name`の後にコンテナで実行するコマンドを入力します。実行結果は次のようになります。

```
$ docker run ubuntu /bin/echo Hello, from Ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
d3a1f33e8a5a: Pull complete
c22013c84729: Pull complete
d74508fb6632: Pull complete
91e54dfb1179: Pull complete
library/ubuntu:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Digest: sha256:73fbe2308f5f5cb6e343425831b8ab44f10bbd77070ecdfbe4081daa4dbe3ed1
Status: Downloaded newer image for ubuntu:latest
Hello, from Ubuntu
```

現在、ローカルには`ubuntu`のイメージはないので、Docker Hubからイメージを`pull`してきます。
`d3a1f33e8a5a`といったIDは中間イメージです。
最終的にイメージのダウンロードが完了した後に、`ubuntu`のイメージで`/bin/echo ...`の実行結果が表示されます。

### コンテナにターミナルで接続

次に`ubuntu`のイメージを起動してターミナルに接続します。

```
docker run -i -t ubuntu /bin/bash
```

`docker run`コマンドのオプションは次のとおりです。

* `-i` 標準入力を有効にする
* `-t` コンテナにターミナルを割り当てる

これを実行すると、イメージが起動されそのコンテナに接続された状態になります。

```
$ docker run -i -t ubuntu /bin/bash
root@a75236856d0a:/#
```

接続したコンテナで各種コマンドを実行できます。

```
root@a75236856d0a:/# whoami
root
root@a75236856d0a:/# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  0.1  18172  3256 ?        Ss   06:12   0:00 /bin/bash
root        18  0.0  0.1  15572  2152 ?        R+   06:12   0:00 ps aux
root@a75236856d0a:/# exit
exit
```

### バックグラウンドでコンテナを起動する

バックグラウンドでコンテナを起動することもできます。
例えば、2秒おきに`Hello`という文字列を表示するワンライナーをバックグラウンドで起動してみます。

```
docker run -d -t ubuntu /usr/bin/perl -e 'while(1) {print "Hello\n"; sleep 2}'
```

`-d`というオプションがバックグラウンドで起動するオプションです。

これを実行すると、コンテナのIDが表示されます。

```
$ docker run -d -t ubuntu /usr/bin/perl -e 'while(1) {print "Hello\n"; sleep 2}'
b70431f3edcbc8286f3b545d7142d8d625f5b71418f651bed02c955ee0a013b0
```

### コンテナの状態を取得する

##### 起動中のコンテナの一覧を取得する

起動中のコンテナの一覧を取得するためのコマンドが`docker ps`です。

```
$ docker ps
CONTAINER ID    IMAGE       COMMAND                  CREATED              STATUS              PORTS       NAMES
b70431f3edcb    ubuntu      "/usr/bin/perl -e 'wh"   About a minute ago   Up About a minute               sick_almeida
```

先ほど起動した`b70431f3edcb`というコンテナが表示されています。

##### 起動中のコンテナのターミナルに表示された文字列を表示する

`docker logs`コマンドで起動中のコンテナのターミナルに出力された文字列を表示することができます。

なお、このコマンドを実行する際は、コンテナのIDまたはコンテナの名前(上記の`NAMES`)を指定します。

```
$ docker logs b70431f3edcb
Hello
Hello
Hello

...

Hello
Hello
~ $
```

##### 起動中のコンテナの詳細な情報を取得する

コンテナの詳細な情報をJSON形式で表示するコマンド`docker inspect`があります。これもコンテナのIDまたはコンテナの名前を指定します。

```
$ docker inspect b70431
[
{
    "Id": "b70431f3edcbc8286f3b545d7142d8d625f5b71418f651bed02c955ee0a013b0",
    "Created": "2015-10-08T06:35:31.928417513Z",
    "Path": "/usr/bin/perl",
    "Args": [
        "-e",
        "while(1) {print \"Hello\\n\"; sleep 2}"
    ],
    "State": {
        "Running": true,
        "Paused": false,
        "Restarting": false,
        "OOMKilled": false,
        "Dead": false,
        "Pid": 1884,
        "ExitCode": 0,
        "Error": "",
        "StartedAt": "2015-10-08T06:35:32.000895386Z",
        "FinishedAt": "0001-01-01T00:00:00Z"
    },

...

        "Image": "ubuntu",
        "Volumes": null,
        "VolumeDriver": "",
        "WorkingDir": "",
        "Entrypoint": null,
        "NetworkDisabled": false,
        "MacAddress": "",
        "OnBuild": null,
        "Labels": {}
    }
}
]
```

なお、このJSONで表示されるIPアドレスはVirtual Box内で起動しているLinuxが認識しているIPアドレスで、Macから接続できるIPアドレスでない点に注意が必要です。

##### 起動中のコンテナのターミナルに接続する

起動中のコンテナのターミナルへの接続は`docker attach`によって可能です。

```
$ docker attach b70431f3
Hello
Hello

...

Hello
^C~ $
```

### 起動中のコンテナを終了する

`docker kill`で起動中のコンテナを終了することができます。

```
$ docker kill b70431f3
b70431f3
```

`Dockerfile`
===

次に`Dockerfile`を作っていきます。

### memcachedの入っているubuntuのイメージ作成

好みのイメージを作成する`Dockerfile`を作っていく方法です。

##### `Dockerfile`からイメージの作成

`ubuntu`をベースにして、`memcached`がインストールされたイメージを作成できる`Dockerfile`を
現在のディレクトリーの下の`memcached`というディレクトリーに作成します。

ディレクトリー

```
current/
└── memcached
    └── Dockerfile
```

`Dockerfile`

```
FROM ubuntu

RUN apt-get update
RUN \
  apt-get install -y memcached && \
  apt-get clean

EXPOSE 11211

CMD ["/usr/bin/memcached", "-vv"]
USER memcache
```

イメージを作成するコマンド

```
docker build -t mikeneck/memcached memcached/
```

イメージを作成するコマンドを実行すると、次のようなログが表示されます。

```
docker build -t mikeneck/memcached memcached/
Sending build context to Docker daemon 2.048 kB
Step 0 : FROM ubuntu
 ---> 91e54dfb1179
Step 1 : RUN apt-get update
 ---> Running in 34dabbd92da4
Ign http://archive.ubuntu.com trusty InRelease
Get:1 http://archive.ubuntu.com trusty-updates InRelease [64.4 kB]

...

Reading package lists...
 ---> 993dfbf39a01
Removing intermediate container 34dabbd92da4
Step 2 : RUN apt-get install -y memcached &&   apt-get clean
 ---> Running in e8279dbae462
Reading package lists...
Building dependency tree...
Reading state information...
The following extra packages will be installed:
  libevent-2.0-5 libsasl2-2 libsasl2-modules libsasl2-modules-db

...

Processing triggers for ureadahead (0.100.0-16) ...
 ---> 5ddbb0a5b1b4
Removing intermediate container e8279dbae462
Step 3 : EXPOSE 11211
 ---> Running in 0b6e50ef3cfc
 ---> 65feb71561d4
Removing intermediate container 0b6e50ef3cfc
Step 4 : CMD /usr/bin/memcached -vv
 ---> Running in f042958d491d
 ---> 90a12ae35bbe
Removing intermediate container f042958d491d
Step 5 : USER memcache
 ---> Running in 67d4618991a8
 ---> f1b644cb1bf5
Removing intermediate container 67d4618991a8
Successfully built f1b644cb1bf5
```

##### コンテナの起動

作成したイメージを起動するコマンドは次のとおりです。

```
docker run -d -t -p 11211 mikeneck/memcached
```

* 起動したコンテナはバックグラウンドで実行されたままにするので`-d`をつけています。
* `-p`オプションはアクセス許可するポート番号です。

実行すると次のようになります。

```
$ docker run -d -t -p 11211 mikeneck/memcached
6df62c13588452bc23e4e2f8c46523231950fdbfc4517f17a97b90f8691d5704
```

##### ポートマッピング

起動したコンテナはポート11211で接続を待機しますが、Virtual Box内部で起動しているOSからしかアクセスできません。
Virtual Box外部(つまりMac)からアクセス可能にするためのポートが自動でマッピングされます。
それを調べるコマンドが`docker port`コマンドです。コンテナIDとポート番号を指定して実行します。

```
$ docker port 6df62c1 11211
0.0.0.0:32769
```

なお、このIPアドレス`0.0.0.0`はVirtual Boxで起動しているOSのものですので、Macから`localhost:32769`にアクセスしてもアクセスすることができません。
Virtual Boxで起動しているOSのIPアドレスを調べるために`docker-machine`コマンドを使います。

```
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM
default   *        virtualbox   Running   tcp://192.168.99.100:2376
```

ここに表示されているIPアドレス`192.168.99.100`がVirtual Boxで起動しているOSのIPアドレスですので、
こちらのポート`32769`にアクセスするとコンテナの`memcached`にアクセスすることができます。

```
docker attach 6df62c1
```

でコンテナのターミナルを表示する状態にした後、新たにターミナルから`192.168.99.100:32769`に`telnet`で接続します。

```
$ telnet 192.168.99.100 32769
Trying 192.168.99.100...
Connected to 192.168.99.100.
Escape character is '^]'.
stats
STAT pid 1
STAT uptime 1058
STAT time 1444358995
STAT version 1.4.14 (Ubuntu)
STAT libevent 2.0.21-stable

...

STAT reclaimed 0
END
quit
Connection closed by foreign host.
```

コンテナのターミナルの方は次のように表示されています。

```
$ docker attach 6df62c1
<30 new auto-negotiating client connection
30: Client using the ascii protocol
<30 stats
<30 quit
<30 connection closed.
```
