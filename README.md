# impostor-compose-template

- Docker Hub: <https://hub.docker.com/r/aeonlucid/impostor>
- <https://github.com/Impostor/Impostor/blob/6b6fa081250d235d1a7f0d4f8765c2816bb5f394/docs/Running-the-server.md>

## Config

- <https://github.com/Impostor/Impostor/blob/6b6fa081250d235d1a7f0d4f8765c2816bb5f394/docs/Server-configuration.md>
- <https://github.com/Impostor/Impostor/blob/6b6fa081250d235d1a7f0d4f8765c2816bb5f394/src/Impostor.Server/config-full.json>

## Deploy

`template.env`を`.env`にコピーする。以下、設定例。

```env
HOST_PORT=0.0.0.0:22023
```

`template.config.json`を`config/config.json`にコピーする。以下、設定例。

- MODを使用する場合・うまくプレイできない場合、AntiCheatは無効化する（弾かれる）
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

docker-composeで起動する。

```shell
docker-compose pull
docker-compose up -d
```

docker-composeでログを確認する。

```shell
docker-compose logs -t -f
```

docker-composeで終了する。

```shell
docker-compose down
```
