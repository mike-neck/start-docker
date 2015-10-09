もっとDockerfile
===

このドキュメントではより実践的なDockerfileの書き方や、`docker`コマンドの使い方が記述されています。

dockerコマンド
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
library/ubuntu:latest: The image you are pulling has been verified. Important:
image verification is a tech preview feature and should not be relied on to
provide security.
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

Dockerfile
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

##### マップされるポート番号を指定してコンテナを起動する

マップされるポート番号を`-p`オプションで指定することも可能です。例えば、ポート番号`31030`にマップする場合は次のようにコンテナを起動します。

```
docker run -d -t -p 31030:11211 mikeneck/memcached
```

### ssh接続できるコンテナを作る

次にssh接続可能なコンテナを作る`Dockerfile`を作ります。

---

ディレクトリー

```
current/
└── ssh
    ├── Dockerfile
    └── authorized_keys
```

あらかじめ、公開鍵を`ssh`ディレクトリーの中に`authorized_keys`というファイル名でコピーしておきます。

`Dockerfile`

```
FROM ubuntu

RUN \
  apt-get update && \
  apt-get install -y openssh-server && \
  apt-get clean && \
  mkdir -p /var/run/sshd

RUN \
  useradd -m mike && \
  mkdir -p /home/mike/.ssh && \
  chown mike /home/mike/.ssh && \
  chmod 700 /home/mike/.ssh

ADD ./authorized_keys /home/mike/.ssh/authorized_keys

RUN \
  chown mike /home/mike/.ssh/authorized_keys && \
  chmod 600 /home/mike/.ssh/authorized_keys

CMD ["/usr/sbin/sshd","-D"]
```

イメージの作成

```
docker build -t mikeneck/sshd ssh/
```

イメージを作成した後は`-d`と`-p 22`オプションを指定した上で、コンテナを起動します。

```
docker run -d -p 22 mikeneck/sshd
```

`docker port`または`docker ps`でマッピングされているポートを調べた上で、`ssh`で接続します。

```
$ ssh -p 32774 192.168.99.100
The authenticity of host '[192.168.99.100]:32774 ([192.168.99.100]:32774)' can't be established.
ECDSA key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.99.100]:32774' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 14.04 LTS (GNU/Linux 3.19.0-25-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

$ whoami
mike
$ exit
Connection to 192.168.99.100 closed.
```

### jdk8/maven3.3.3が乗っかっているイメージを作成する

次にjdk8とmaven3.3.3がインストールされているイメージを作成します。

---

ディレクトリー

```
current/
└── java8
    ├── Dockerfile
    └── project
        ├── pom.xml
        └── src
            ├── main
            │   ├── java
            │   │   └── com
            │   │       └── example
            │   │           └── DemoApplication.java
            │   └── resources
            │       └── application.properties
            └── test
                └── java
                    └── com
                        └── example
                            └── DemoApplicationTests.java
```

`Dockerfile`

```
FROM ubuntu

RUN \
  apt-get update && \
  apt-get install -y \
    software-properties-common \
    unzip \
    curl

RUN \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  add-apt-repository -y ppa:webupd8team/java && \
  apt-get update && \
  apt-get install -y oracle-java8-installer && \
  apt-get clean && \
  rm -rf /var/cache/oracle-jdk8-installer

RUN \
  groupadd -r build && \
  useradd -m -g build mvn

RUN \
  curl -L \
    http://ftp.yz.yamagata-u.ac.jp/pub/network/apache/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.zip \
    -o /tmp/maven3.zip

RUN \
  unzip -q tmp/maven3.zip -d tmp/maven && \
  mv tmp/maven/apache-maven-3.3.3/ opt/maven && \
  rm -rf tmp/*

USER mvn
WORKDIR /home/mvn

ENV JAVA_HOME /usr/lib/jvm/java-8-oracle
ENV PATH=/opt/maven/bin:$PATH
EXPOSE 8080

CMD ["/bin/bash"]
```

イメージのビルド

```
docker build -t mikeneck/mvn java8
```

`java8`ディレクトリーには`project`ディレクトリーがあり、その中にmavenプロジェクトがありますので、コンテナでそのプロジェクトのテストをしてみます。

```
docker run -it -v /path/to/current/java8/project:/home/mvn/project mikeneck/mvn
```

`-v`オプションはホストのディレクトリーをコンテナ内のディレクトリーに割り当てるオプションです。上記のコマンドではマシン上の`path/to/current/java8/project`ディレクトリーをコンテナの`/home/mvn/project`ディレクトリーに割り当てます。

コンテナを起動した後に、`project`ディレクトリーに移動して、`mvn test`を実行します。

```
$ cd project
$ mvn test
[INFO] Scanning for projects...
Downloading: https://repo.maven.apache.org/maven2/org/springframework/boot/sprin
g-boot-starter-parent/1.2.6.RELEASE/spring-boot-starter-parent-1.2.6.RELEASE.pom

...

[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building demo 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------

...

[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] Copying 0 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ demo ---

...

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.example.DemoApplicationTests

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.2.6.RELEASE)
 Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.787 sec - in
 com.example.DemoApplicationTests

 Results :

 Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

 [INFO] ------------------------------------------------------------------------
 [INFO] BUILD SUCCESS
 [INFO] ------------------------------------------------------------------------
 [INFO] Total time: 01:13 min
 [INFO] Finished at: 2015-10-09T07:50:32+00:00
 [INFO] Final Memory: 22M/54M
 [INFO] ------------------------------------------------------------------------
$ ls -la
total 8
drwxr-xr-x 1 mvn staff  170 Oct  9 07:50 .
drwxr-xr-x 5 mvn build 4096 Oct  9 07:49 ..
-rw-r--r-- 1 mvn staff 1352 Oct  9 07:48 pom.xml
drwxr-xr-x 1 mvn staff  136 Oct  9 07:15 src
drwxr-xr-x 1 mvn staff  272 Oct  9 07:50 target
```

なお、コンテナに割り当てられたディレクトリー内にて生成されたファイル・ディレクトリーはマシン上のディレクトリーにも残ります。
