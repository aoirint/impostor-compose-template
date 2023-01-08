# impostor-compose-template

- Docker Hub: <https://hub.docker.com/r/aeonlucid/impostor>
- <https://github.com/Impostor/Impostor/blob/6b6fa081250d235d1a7f0d4f8765c2816bb5f394/docs/Running-the-server.md>

## デプロイ

サーバ向けDocker EngineおよびDocker Compose V2が導入されたLinuxサーバ（Ubuntu）を想定する。

- <https://docs.docker.com/engine/install/ubuntu/>

`template.env`を`.env`にコピーする。以下、設定例。

- `HOST_PORT`は、必要があればバインドするアドレスに書き換える。通常、`0.0.0.0:22023`でよい。
    - NAT下にないVPSなどの場合、ポート番号の変更はここでする
    - サーバやVPSサービスにパケットフィルタがある場合、設定したポート番号でのUDP通信を許可しておく

```env
HOST_PORT=0.0.0.0:22023
```

`template.config.json`を`config/config.json`にコピーする。以下、設定例。

- 以下の場合、AntiCheatは無効化する
    - うまくプレイできない（ルームの作成、参加時、試合開始時などに弾かれる）
    - MODを導入している
- `PublicIp`はWAN側IPアドレスに書き換える
- `PublicPort`はWAN側ポート番号に書き換える

```json
{
  "Server": {
    "PublicIp": "127.0.0.1",
    "PublicPort": 22023,
    "ListenIp": "0.0.0.0",
    "ListenPort": 22023
  },
  "AntiCheat": {
    "Enabled": false,
    "BanIpFromGame": false
  }
}
```

### 起動

```shell
sudo docker compose pull
sudo docker compose up -d
```

### ログの確認

```shell
sudo docker compose logs -t -f
```

### 終了

```shell
sudo docker compose down
```

### Config作成の際に参考になるリンク

- <https://github.com/Impostor/Impostor/blob/6b6fa081250d235d1a7f0d4f8765c2816bb5f394/docs/Server-configuration.md>
- <https://github.com/Impostor/Impostor/blob/6b6fa081250d235d1a7f0d4f8765c2816bb5f394/src/Impostor.Server/config-full.json>


## 接続

### Among Us v2022.12.8/v2022.12.14における注意点

Among Usの仕様変更により、UDPのみによる接続が難しくなりました（Among Us公式では、2022.8.23sの時点で既にHTTPマッチングサーバが使われるようになっているため、今後の利用可能性は不透明です）。

このバージョンに対応したImpostor 1.8.0では、HTTPマッチングサーバのカスタム実装である[Impostor.Httpプラグイン](https://github.com/Impostor/Impostor.Http)を導入することが[推奨されています](https://github.com/Impostor/Impostor/releases/tag/v1.8.0)。ただし、クライアントにカスタムサーバーを設定する機能があるMODを導入している場合、HTTPマッチングサーバに非対応の可能性があります。

#### 問題

ホストは、「ゲームを作成する」でマッチングルームを作成するUIを開いたときに、「カスタムサーバのIPアドレスとポート番号宛てにHTTPS通信を試み、失敗した」旨のエラーダイアログがゲーム画面上に表示されます。このエラーダイアログのメッセージには、カスタムサーバのIPアドレスとポート番号が表示されます。そして、Among Usアカウントがサインアウトし、マッチングルームを作成できません。

#### 回避策

ホストおよびゲストは、以下の手順を踏むことで、エラーダイアログの表示およびマッチングできない問題を回避できます。

1. 以下の「Among Us 2022.10.25時点の接続手順」に従って、カスタムサーバを設定する
2. タイトル画面から「オンライン」を開き、リージョンを公式サーバに設定する
3. ホストとして公式サーバで「ゲームを作成する」を開き、なにもせずに戻る
4. リージョンをカスタムサーバに設定する
5. 以後、通常の手順に従ってゲームを作成または参加する。

さらなる注意点として、この手順に従った場合でも、マッチングルームに入室した後で、上記のエラーダイアログが画面右下に表示される場合があります。
この現象は再現手順が不明なため、回避策がわかりません。

#### Among Us 2022.10.25時点の接続手順

<details>

### カスタムサーバー設定機能のあるMOD

MODの機能を使って設定する。

### バニラ

`C:\Users\%USERNAME%\AppData\LocalLow\Innersloth\Among Us\regionInfo.json`を編集する。

`Regions`に以下を追加する。

ホスト名`myserver.example.com`の場合、

```json
{
  "$type": "DnsRegionInfo, Assembly-CSharp",
  "Fqdn": "myserver.example.com",
  "DefaultIp": "myserver.example.com",
  "Port": 22023,
  "UseDtls": false,
  "Name": "custom",
  "TranslateName": 1003
}
```

IPアドレス`127.0.0.1`の場合、

```json
{
  "$type": "DnsRegionInfo, Assembly-CSharp",
  "Fqdn": "127.0.0.1",
  "DefaultIp": "127.0.0.1",
  "Port": 22023,
  "UseDtls": false,
  "Name": "custom",
  "TranslateName": 1003
}
```

JSONを吐き出してくれるツールがあったが、公式サーバーの情報が古そうだったので個人的に非推奨（ <https://impostor.github.io/Impostor/> ）。

</details>


#### Among Us 2022.8.23s時点のregionInfo.jsonの設定例

<details>

```json
{
  "CurrentRegionIdx": 3,
  "Regions": [
    {
      "$type": "StaticHttpRegionInfo, Assembly-CSharp",
      "Name": "North America",
      "PingServer": "matchmaker.among.us",
      "Servers": [
        {
          "Name": "Http-1",
          "Ip": "https://matchmaker.among.us",
          "Port": 443,
          "UseDtls": true,
          "Players": 0,
          "ConnectionFailures": 0
        }
      ],
      "TranslateName": 289
    },
    {
      "$type": "StaticHttpRegionInfo, Assembly-CSharp",
      "Name": "Europe",
      "PingServer": "matchmaker-eu.among.us",
      "Servers": [
        {
          "Name": "Http-1",
          "Ip": "https://matchmaker-eu.among.us",
          "Port": 443,
          "UseDtls": true,
          "Players": 0,
          "ConnectionFailures": 0
        }
      ],
      "TranslateName": 290
    },
    {
      "$type": "StaticHttpRegionInfo, Assembly-CSharp",
      "Name": "Asia",
      "PingServer": "matchmaker-as.among.us",
      "Servers": [
        {
          "Name": "Http-1",
          "Ip": "https://matchmaker-as.among.us",
          "Port": 443,
          "UseDtls": true,
          "Players": 0,
          "ConnectionFailures": 0
        }
      ],
      "TranslateName": 291
    },
    {
      "$type": "DnsRegionInfo, Assembly-CSharp",
      "Fqdn": "127.0.0.1",
      "DefaultIp": "127.0.0.1",
      "Port": 22023,
      "UseDtls": false,
      "Name": "custom",
      "TranslateName": 1003
    }
  ]
}
```

</details>

