# Phase: 0

## `Hello World` を返す `HTTP` サーバーをセットアップ

### コマンド
```bash
cd ./infra-lab/00/
uv run fastapi dev
curl http://127.0.0.1:8000/
```

### やったこと

- `uv` をインストール
- `FastAPI` を依存関係に追加
- `00/main.py` に `/` ルーティングを記述
- `FastAPI` の `HTTP` サーバーを起動
- サーバーを `curl` で叩く

### 見た記事

- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- [FastAPI](https://fastapi.tiangolo.com/ja/#fastapi-mini-documentary)
- [Python デコレータ](https://www.learnpython.org/ja/Decorators)
- [curl](https://curl.se/)
