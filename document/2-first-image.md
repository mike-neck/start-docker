Docker イメージを起動したり、作ったり
===

このドキュメントではDockerイメージを起動したり、Dockerイメージを作ったりします

### Hello World

[インストール](1-install.md)で起動したターミナルにて、次のコマンドを実行します。

```
docker run hello-world
```

次のように表示されます。

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
535020c3e8ad: Pull complete
af340544ed62: Pull complete
Digest: sha256:a68868bfe696c00866942e8f5ca39e3e31b79c1e50feaee4ce5e28df2f051d5c
Status: Downloaded newer image for hello-world:latest

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/
```

最初の2行

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
```

では、`hello-world`というイメージがローカルにないので、Docker Hubにある`hello-world`イメージを`pull`してきます。

次の2行

```
535020c3e8ad: Pull complete
af340544ed62: Pull complete
```

は、中間イメージ(後述)を`pull`するメッセージです。

`Hello from Docker`以降のメッセージは`hello-world`イメージが実行するコマンドです。

---

このコマンドにより、ローカルに`hello-world`というイメージが保存されます。

それを確認するために、次のコマンドを実行してみます。

```
docker images
```

`docker images`コマンドはローカルにあるDockerイメージを表示するコマンドです。この実行結果は次のようになります。

```
REPOSITORY              TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
hello-world             latest              af340544ed62        8 weeks ago         960 B
```

---

ここで再び`docker run hello-world`を実行してみます。

```
~ $ docker run hello-world

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/
```

初回起動時に表示されていた最初の6行ほどが表示されなくなりました。これはDocker Hubからイメージを`pull`しなくなったためです。

### whalesay

Docker Hubには様々な人がつくったイメージが集まっています。そこから、`whalesay`イメージを探します。

* [Docker Hub](https://hub.docker.com/)にアクセスします
* 右の方にある「Search」に`whalesay`と入力して検索します

![Docker Hub検索](images/2-first/docker-hub.png?raw=true)

* `docker/whalesay`というイメージが見つかりるので、クリックします。

![docker/whalesay](images/2-first/whalesay.png?raw=true)

* `docker/whalesay`イメージの詳細情報が表示されます。

![docker/whalesay](images/2-first/repo.png?raw=true)

ここに書かれている方法で、このイメージを実行します。

```
docker run docker/whalesay cowsay boo
```

すると次のように表示されます。

```
Unable to find image 'docker/whalesay:latest' locally
latest: Pulling from docker/whalesay
e9e06b06e14c: Pull complete
a82efea989f9: Pull complete
37bea4ee0c81: Pull complete
07f8e8c5e660: Pull complete
676c4a1897e6: Pull complete
5b74edbcaa5b: Pull complete
1722f41ddcb5: Pull complete
99da72cfe067: Pull complete
5d5bd9951e26: Pull complete
fb434121fc77: Already exists
Digest: sha256:178598e51a26abbc958b8a2e48825c90bc22e641de3d31e18aaf55f3258ba93b
Status: Downloaded newer image for docker/whalesay:latest
 _____
< foo >
 -----
    \
     \
      \
                    ##        .
              ## ## ##       ==
           ## ## ## ##      ===
       /""""""""""""""""___/ ===
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
       \______ o          __/
        \    \        __/
          \____\______/
```

次はこのイメージを元にオリジナルのDockerイメージを作成します

### Dockerfileを作成する

* 適当なディレクトリーを作成して、移動します。

```
$ mkdir my-docker
$ cd my-docker
```

* `Dockerfile`をまず作成します。

```
touch Dockerfile
```

* お好みのエディターで、`Dockerfile`を次のように編集します。

```
FROM docker/whalesay                 #(1)

RUN apt-get -y update && \           #(2)
  apt-get install -y fortunes        #(3)

CMD /usr/games/fortune -a | cowsay   #(4)
```

* `my-docker`ディレクトリーにいる状態で、次のコマンドでDockerイメージを作成します(最後のピリオドを忘れないで下さい)。

```
docker build -t docker-whale .
```

次のように表示されます。

```
Sending build context to Docker daemon 2.048 kB
Step 0 : FROM docker/whalesay:latest
 ---> fb434121fc77
Step 1 : RUN apt-get -y update && apt-get install -y fortunes
 ---> Running in f538425f631d
Ign http://archive.ubuntu.com trusty InRelease
...
Setting up fortunes (1:1.99.1-7) ...
Processing triggers for libc-bin (2.19-0ubuntu6.6) ...
 ---> 1eacb147f8f6
Removing intermediate container f538425f631d
Step 2 : CMD /usr/games/fortune -a | cowsay
 ---> Running in 2cd41c9dae02
 ---> 446c9c19f799
Removing intermediate container 2cd41c9dae02
Successfully built 446c9c19f799
```

`Successfully 〜`と表示されていることから、イメージの作成に成功したことがわかります。

* `docker images`でローカルにあるDockerイメージを表示します。

```
docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
docker-whale           latest              446c9c19f799        3 minutes ago       274 MB
hello-world            latest              af340544ed62        8 weeks ago         960 B
docker/whalesay        latest              fb434121fc77        4 months ago        247 MB
```

`docker-whale`というイメージがあることがわかります。

* `docker run docker-whale` コマンドでこのイメージを実行します。

```
docker run docker-whale
 ________________________________
/ "First things first -- but not \
| necessarily in that order"     |
|                                |
\ -- The Doctor, "Doctor Who"    /
 --------------------------------
    \
     \
      \
                    ##        .
              ## ## ##       ==
           ## ## ## ##      ===
       /""""""""""""""""___/ ===
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
       \______ o          __/
        \    \        __/
          \____\______/
```

上記のように表示されます。

---

このように、`Dockerfile`によって、ある本のイメージから`RUN`コマンドで何かしらのアプリケーションをインストールした上で、新しいイメージを作成することができます。
