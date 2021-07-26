# himastrtmp-docker

[**nginx-rtmp-module**](https://github.com/arut/nginx-rtmp-module) モジュールを有効にした[**Nginx**](http://nginx.org/en/)を[**Docker**](https://www.docker.com/) で実行するためのDockerfileとdocker-composeで実行するためのdocker-compose.ymlです。

## 説明

[tiangolo/nginx-rtmp-docker](https://github.com/tiangolo/nginx-rtmp-docker)をベースにひまスト用にHLS配信を行うカスタマイズを加えています。それ以外はオリジナルのままです。

## 使い方

* Dockerとdocker-composeが動作する環境であることが前提です。

```bash
git clone https://github.com/shipwebdotjp/himastrtmp-docker.git
cd himastrtmp-docker
docker-compose up -d --build
```

* TCPポート80と1935を使用しますので、ファイアウォールやルーターなどの設定で、2つのポートを開放して、Dockerホストに到達できるように設定してください。

## 注意
80ポートを既に使用している場合(ApacheやNginxなどのWebサーバーがすでに稼働している場合)は同じポートをDockerで使用することはできないため、エラーが発生します。

## OBS Studio での配信（プッシュ）

* [OBS Studio](https://obsproject.com/)の設定を開き、「配信」タブを開きます。
* サービス　→　カスタム
* サーバー　→  `rtmp://<DockerホストのIPアドレスかホスト名>/himastrtmp`
* ストリームキー　→　`live`
* 「OK」で設定を閉じ、「配信開始」ボタンで配信を開始します。

## ひまスト配信コンソールでの設定
* 配信設定タブで「外部サーバー」を選択
* 外部サーバー URL　→ `rtmp://<DockerホストのIPアドレスかホスト名>/himastrtmp/live`
* 「ENTER」キーを押してURLの変更を反映させます。
* 「HLSのプレイリスト取得に成功しました。」と表示され配信が再生されれば成功です。

## うまく動かない場合

Nginxの待ち受けがうまく出来ているか下記のコマンドで確認できます。
```bash
docker-compose ps
      Name               Command          State                                   Ports
------------------------------------------------------------------------------------------------------------------------
himastrtmp_web_1   nginx -g daemon off;   Up      0.0.0.0:1935->1935/tcp,:::1935->1935/tcp,
                                                  0.0.0.0:80->80/tcp,:::80->80/tcp
```
HLS配信のアクセスログ表示コマンド
```bash
docker-compose logs -f
```
live-xx.tsファイルへのアクセスがあれば、ファイルへのアクセスが行われていることになります。


## 遅延

HLSでの配信はどうしても遅延が発生します。  
`nginx.conf`でパラーメータを調整して、一つ一つの動画ファイルの断片を小さくすることで遅延を少なくできますが、その分負荷が高くなります。

```conf
hls_playlist_length 6s;
hls_fragment 3s;
```
この設定で、OBSの設定でキーフレームを1に設定した場合遅延は10秒ほどになりました。

## License

This project is licensed under the terms of the MIT License.
