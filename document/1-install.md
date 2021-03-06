インストール方法
===

このドキュメントにはDockerをMac OS Xにインストールする手順が記されています。

### Macのバージョンを確認する

* メニューバーのリンゴマークから「このMacについて」を選択する
* 次のとおり、MacのバージョンがOS X 10.8(Mountain Lion)以上であることを確認する

![Macのバージョンを確認](images/1-install/osx.png?raw=true "Mac OS X 10.8以上であることを確認する")

### Docker Toolboxをダウンロードする

* [Docker Toolboxのページ](https://www.docker.com/toolbox)に移動する
* 「Download(Mac)」をクリックしてダウンロードする
* `DockerToolbox-version.pkg`ファイルがダウンロードされる

### Docker Toolboxをインストールする

* ダウンロードした`DockerToolbox-version.pkg`ファイルを起動して、あとはよきにはからえでインストールする
* インストールが完了すると、launchpadに次のアプリケーションが追加されている

![Docker TerminalとKitematicとVirtual-Box](images/1-install/apps.png?raw=true "3つアプリが追加されている")

### Docker Quick Terminalを起動してみる

* 上記のイメージの左にある`Docker Quick Terminal`を起動する
* 次のような順番でターミナルが表示されていく。最終的にクジラのイメージが表示されれば起動完了。

![Docker 起動](images/1-install/launch1.png?raw=true)

![Docker 起動中](images/1-install/launch2.png?raw=true)

![Docker 起動完了](images/1-install/launch3.png?raw=true)
