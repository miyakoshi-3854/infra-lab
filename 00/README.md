# Phase: 0

## `Hello World` を返す `HTTP` サーバーをセットアップ

### コマンド
```bash
cd ./infra-lab/00/
uv run fastapi dev
curl http://127.0.0.1:8000/
```

### 準備

- `uv` をインストール
- `FastAPI` を依存関係に追加
- `00/main.py` に `/` ルーティングを記述
- `FastAPI` の `HTTP` サーバーを起動

### 実験

- `http://127.0.0.1:8000/` を `curl -iX` で叩く

### 結果

各レスポンスの違い:

| ステータス | 説明 |
| -- | -- |
| `200 OK` | ルーティングに追加したメソッドを叩く |
| `400 Bad Request` | 存在しないメソッドを指定する (`HTTP`パース層で弾かれる) |
| `404 Not Found` | 存在しないルーティングを叩く (ルーティング層で弾かれる) |
| `405 Method Not Allowed` | 許可されていないメソッドを指定する (ルーティング層で弾かれる) |

### 見た記事

- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- [FastAPI](https://fastapi.tiangolo.com/ja/#fastapi-mini-documentary)
- [Python デコレータ](https://www.learnpython.org/ja/Decorators)
- [curl](https://curl.se/)
