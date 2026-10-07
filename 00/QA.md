# Phase 0 質疑応答

Phase 0「分かるようになること」の理解度チェック。Claude からの質問に自分で回答し、フィードバックをもらった記録。

- **問い**: Claude からの質問
- **回答**: 自分の回答(原文のまま)
- **フィードバック**: もらった指摘の要点
- **残課題**: まだ確かめていないこと

## 進捗

| # | テーマ | 状態 |
| -- | -- | -- |
| Q1 | リクエストからレスポンスまでの流れ | ✅ |
| Q2 | Python の値と HTTP レスポンスの違い | ✅ |
| Q3 | 400 と 405 は誰が返しているか | ✅ |
| Q4 | サーバーはなぜ終了しないのか | 🔁 残課題あり |
| Q5 | `127.0.0.1:8000` の意味 | 🔁 残課題あり |
| Q6 | 同じ port で2つ起動するとどうなるか | ✅ |
| Q7 | 図の矢印の間に隠れている層 | ⬜ 未回答 |
| Q8 | `fastapi dev` を本番で使わない理由 | ⬜ 未回答 |

---

## A. Web アプリとは何か

### Q1. `curl http://127.0.0.1:8000/` から `Hello World` が表示されるまで

**問い**

curl を叩いてから `Hello World` が表示されるまでに何が起きているか、順番に説明する。登場人物は curl、ネットワーク(loopback)、サーバープロセス、FastAPI のコード。

**回答**

1. curl で自分自身(`127.0.0.1` / `localhost`)の 8000 番に対してリクエストを投げる(loopback)
2. サーバー(uvicorn / HTTP 層)に登録されている `/` が応答、登録した関数戻り値の `Hello World` が帰る

curl が事前にやっていること:

- URL のエンコード
- 電子証明書の確認、セキュリティチェック?

**フィードバック**

- FastAPI は「クライアント」ではない。クライアントはリクエストを送る側(curl)で、FastAPI はサーバー側のアプリ(framework)。
- レスポンスの `server: uvicorn` を手がかりに、uvicorn という登場人物を見つけられた。
- 証明書の確認は HTTPS のときだけで、今回の `http://` では発生しない。URL エンコードも curl は基本的に自動ではやらない。

**残課題**

- [x] curl に通信の過程を表示させるオプションを付けて叩き、リクエストを送る前に何が確立されているかを確認する(HTTP は何の上に乗っているか)

#### 残課題の確認: HTTP は何の上に乗っているか

**回答**

`curl -v` で通信の過程を表示した。

- `*`: curl 自身の報告
- `>`: 送ったもの(リクエスト)
- `<`: 受け取ったもの(レスポンス)

```bash
❯ curl -v http://127.0.0.1:8000/ 
*   Trying 127.0.0.1:8000...
* Connected to 127.0.0.1 (127.0.0.1) port 8000
> GET / HTTP/1.1
> Host: 127.0.0.1:8000
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< date: Wed, 07 Oct 2026 00:22:34 GMT
< server: uvicorn
< content-length: 13
< content-type: application/json
< 
* Connection #0 to host 127.0.0.1 left intact
"Hello World"%
```

HTTP リクエストを送る前に OS により、TCP 接続が確立している

curl が実際にやっていること:

- OS の TCP 層に対して socket(通信の窓口)の作成を依頼
- 指定した port へ接続してもらうように依頼

**フィードバック**

- 正解。`* Connected to ...` が `>` より前に出ていることが根拠になる。
- 流れは socket を作る → `connect()` で kernel に依頼 → kernel が 3-way handshake → 成功が返る → `* Connected to ...` → HTTP のバイト列を socket に書き込む(`>`)。
- handshake をしているのは curl ではなく kernel。curl は依頼して結果を待つだけ。Q6 の `bind()` がサーバー側の依頼で、`connect()` がクライアント側の依頼。
- `*` の行に HTTP が出てこないのは、TCP の接続が IP と port だけで成り立ち、上で何を話すかを知らないから。だから同じ TCP の上に SSH や DB 接続も乗る。
- kernel から見ると、HTTP はその接続の上を流れるただのバイト列。

**残課題**

- [ ] curl が kernel に依頼している瞬間を `strace` で見る(Phase 1 で)

### Q2. Python の値と、curl が受け取ったものの違い

**問い**

`return ("Hello World")` は Python の文字列を返しているだけ。curl が実際に受け取ったものは何か。「Python の値」と「HTTP レスポンス」の違いは何か。

**回答**

1. curl が受け取ったものは `content-type` から確認できる。今回は JSON 形式。

```bash
content-type: application/json
```

2. HTTP レスポンスは下記の部分

```bash
HTTP/1.1 200 OK
date: Fri, 02 Oct 2026 12:56:08 GMT
server: uvicorn
content-length: 13
content-type: application/json
```

3. Python の値は下記の部分

```bash
"Hello World"%
```

追加で考えたこと(`content-length: 13` の内訳):

```text
Hello World = 11
"" = 2
11 + 2 = 13
```

**フィードバック**

- ネットワークを流れるのはバイト列だけ。Python の `str` オブジェクトはプロセスの外に出られない。
- 流れは `str` オブジェクト → FastAPI が JSON に serialize(`""` で囲む)→ バイト列に encode → socket → curl。
- なので回答3の `"Hello World"` は「Python の値」ではなく、**body(JSON に serialize された値)**。`""` を付けたのは FastAPI。
- HTTP レスポンスは status line、headers、空行、body の4つで構成されている。body もレスポンスの一部。

**残課題**

- [x] `return "こんにちは"` にしたら `content-length` はいくつになるか予想し、試す(文字数とバイト数の違い)
- [x] 末尾の `%` の正体を、zsh で `echo hi` と `echo -n hi` を比べて確認する
- [x] 戻り値を数値や dict に変えて、body と `content-length` を比べる

#### 残課題の確認: 文字数と byte 数の違い

**回答**

予想: `こんにちは` の `content-length` は恐らく 10。JIS では日本語1文字 2 byte なので。

実際: UTF-8 変換されており、ほとんどの場合日本語は 3 byte になる。(`content-length: 17` = 5文字 × 3 byte + `""` 2 byte)

発展: Python の `str` `"🍣"` を返して、`curl --trace-ascii` で通信を byte 単位で表示した。

```bash
❯ curl --trace-ascii - http://127.0.0.1:8000/
== Info:   Trying 127.0.0.1:8000...
== Info: Connected to 127.0.0.1 (127.0.0.1) port 8000
=> Send header, 77 bytes (0x4d)
0000: GET / HTTP/1.1
0010: Host: 127.0.0.1:8000
0026: User-Agent: curl/8.5.0
003e: Accept: */*
004b: 
<= Recv header, 17 bytes (0x11)
0000: HTTP/1.1 200 OK
<= Recv header, 37 bytes (0x25)
0000: date: Wed, 07 Oct 2026 01:18:24 GMT
<= Recv header, 17 bytes (0x11)
0000: server: uvicorn
<= Recv header, 19 bytes (0x13)
0000: content-length: 6
<= Recv header, 32 bytes (0x20)
0000: content-type: application/json
<= Recv header, 2 bytes (0x2)
0000: 
<= Recv data, 6 bytes (0x6)
0000: "...."
"🍣"== Info: Connection #0 to host 127.0.0.1 left intact
```

流れ:

1. FastAPI(Starlette)が JSON にシリアライズして、UTF-8 変換し 4 byte に
2. uvicorn が HTTP レスポンス組み立て、返却
3. OS が TCP で通信
4. curl が受け取り、そのまま出力
5. terminal が UTF-8 をデコードして表示する

**結論**

文字として扱っているのは:

- エンコードしている Starlette
- デコードしている terminal

間にある uvicorn、OS、curl は何を知っているのか:

- ただの byte 列ということだけ知っている。中身は知らない

**フィードバック**

- 結論は正解。文字を知っているのは両端だけで、間は byte 列を運ぶだけ。Q1 の「TCP から見た HTTP はただのバイト列」と同じ構造が、一つ上の層(HTTP から見た body)でも起きている。
- 「中身」は **body の中身** と限定するとより正確。各層は自分の担当部分だけを読み、残りはバイト列として渡している。

| 登場人物 | 知っていること | 知らないこと |
| -- | -- | -- |
| uvicorn | HTTP の書式(status line、headers) | body が何の文字か |
| OS | IP と port、届いたかどうか | 上が HTTP かどうか |
| curl | headers と、`content-length` から分かる body の長さ | body の 6 byte が何の文字か |

- UTF-8 は可変長(ASCII は 1 byte、日本語の多くは 3 byte、絵文字は 4 byte)。ASCII と互換にするための設計。JIS(Shift_JIS)なら日本語は 2 byte なので、予想は根拠があった。encoding が1つではないことが外れた理由。
- `content-length` を数えているのは uvicorn ではなく Starlette。Starlette は ASGI で「status + headers」と「body(すでにバイト列)」を uvicorn に渡す。uvicorn が足すのは `server` と `date`(HTTP サーバーとしての情報)。
- `--trace-ascii` の出力から読み取れたこと:
  - body が `"...."`: `🍣` の 4 byte は ASCII で表示できないので `.` になる。curl は文字として見ていない。
  - `HTTP/1.1 200 OK` が 15文字なのに 17 bytes: 各行末の `\r\n`(2 byte)も含むから。
  - `Recv header, 2 bytes` の空行: headers と body の区切り。
  - `content-length: 6` と `Recv data, 6 bytes` が一致: curl は `content-length` の分だけ body を読む。

**残課題**

- [ ] Starlette の `JSONResponse.render` を読み、encode と `content-length` の設定箇所を確認する
- [ ] uvicorn の `server` / `date` header を消すオプションを付けて起動し、`content-*` だけ残ることを確かめる
- [ ] `--trace` で `🍣` の 4 byte の実際の値(16進数)を見る

#### 残課題の確認: 末尾の `%` の正体

**回答**

`echo hi` と `echo -n hi` を比べると、`-n`(改行なし)の方だけ `hi%` になった。単純に zsh の改行無し表示で `%` 使われているだけ。

**フィードバック**

- 正解。出力が改行で終わっていないとき、zsh は `%` を付けてから改行し、次の prompt が重ならないようにしている。
- ここから、サーバーの body は改行で終わっていない(`"Hello World"` の 13 byte だけで、`\n` は付かない)ことも分かる。`%` は body の一部ではない。

#### 残課題の確認: 様々な戻り値を試す

**予想**

返す文字の長さがそのまま `content-length` になる。プログラミングにおいて int などはそのまま書けるから。
シリアライズされない。

**回答**

JSON にエンコードする時に、全ての文字が必ずシリアライズされ、エンコードされる。
文字列をシリアライズするときに `""` がつけられ、`content-length` が +2 される

**フィードバック**

- 結論は正解。シリアライズは型に関係なく毎回起きていて、型ごとに JSON の書き方のルールが違うだけ。

| Python | body(JSON) | `content-length` |
| -- | -- | -- |
| `"Hello World"` | `"Hello World"` | 13 |
| `123` | `123` | 3 |
| `{"apple": 3}` | `{"apple":3}` | 11 |
| `True` | `true` | 4 |
| `None` | `null` | 4 |

- 「全ての文字が」より「全ての**値**が」が正確。シリアライズされるのは Python の値(オブジェクト)で、それが JSON のテキストになり、そのテキストが UTF-8 で encode されて byte になる。順番は「値 → serialize → テキスト → encode → byte」。
- int も「そのまま」ではない。メモリ上の int オブジェクトが `1` `2` `3` という3文字に書き換えられている。数値は JSON では引用符なしで書くので `""` が付かないだけ。
- dict の `:` の後の空白が消えるのは、Starlette が空白を詰めて書く設定にしているから。
- `True` → `true`、`None` → `null` は、Python と JSON で綴りが違うので、書き換えが起きていることが一番はっきり分かる例。JSON は大文字・小文字を区別するので、`Null` ではなく `null`。

### Q3. 400 と 405 は誰が返しているか

**問い**

400 は「HTTP パース層で弾かれる」、405 は「ルーティング層で弾かれる」。それぞれの層はどのソフトウェアが担当しているか。TCP の接続を受けているのは FastAPI 自身か。

**回答**

1. 400 は HTTP 層(uvicorn)が担当
2. 405 はルーティング層(FastAPI)が担当

考察: 400 がルーティング層に到達しないことから逆説的に考えた。

**フィードバック**

- 正解。最初は「400 は OS が担当」と答えていたが、OS(kernel)が扱うのは TCP まで。届いたバイト列が HTTP として正しいかどうかは見ていない。
- 400 のレスポンスにも `server: uvicorn` が付いていることが判断の根拠になる。
- Q6 の「OS が拒否する」と並べると、層ごとに拒否する担当が違うことが分かる。

---

## B. サーバー上でプログラムが動くとはどういうことか

### Q4. サーバーはなぜ終了しないのか

**問い**

普通の Python スクリプトは最後の行まで実行すると終了する。`fastapi dev` はなぜ動き続けるのか。その間、プロセスは何を待っているのか。

**回答**

1. サーバーは内部で event loop が回っているから。
2. その間、リクエストを待ち、処理をする。

リクエストを待つ間 CPU を使い続けているか:

- 予想: 使い続けていない。ずっと待ち続けるのは非効率だと思うから。
- 実際: 自分の環境のタスクマネージャーから監視したが、常に振れているため分からない。

**フィードバック**

- サーバーが待つのは「レスポンス」ではなく「リクエスト(接続)」。最初の回答から修正できた。
- 予想は当たっている。ただ「非効率だから」は理由ではなく願望なので、仕組みで説明したい。
- Windows のタスクマネージャーで見えるのは WSL2 の VM 全体。Linux 内の1プロセスは見えない。

**残課題**

- [ ] Linux 側で、プロセス(PID)単位で CPU 使用率を見る
- [ ] 待っている間の uvicorn のプロセスの「状態(state)」を確認する。誰が、いつこのプロセスを起こすのか(Phase 1 につながる)

### Q5. `127.0.0.1:8000` の意味

**問い**

`127.0.0.1` と `8000` はそれぞれ何を指定しているか。別の PC(または Windows 側のブラウザ)から、WSL2 上のこのサーバーにアクセスできるか。

**回答**

1. `127.0.0.1`(`localhost`)は自分自身を指す特別な IP
2. `8000` はどのプロセスが通信をしているのか識別する番号
3. 他の PC からアクセスはできない。なぜなら PC 内で完結しており、外部に公開していないから。

実験結果: 他の端末では開けなかった。Windows のブラウザでは開けた。

**フィードバック**

- 1: 正解。`127.0.0.1` は loopback address で、パケットは NIC から外に出ずに kernel の中で折り返す。
- 2: OK。IP address で「どのマシンか」が決まり、port で「そのマシンのどのプロセス(socket)か」が決まる。
- Windows のブラウザで開けた理由: WSL2 は Windows とは IP の違う別の VM だが、Windows の `localhost` への接続を WSL 内に転送する仕組みを持っている。
- 他の端末で開けなかった理由: 壁が少なくとも2枚ある。
  1. **bind address**: uvicorn は `127.0.0.1` で listen している。loopback 宛ての接続しか受け付けない。
  2. **WSL2 のネットワーク構成**: 外から見えるのは Windows の IP だけ。WSL2 の VM はその奥(NAT の内側)にいる。

**残課題**

- [ ] `fastapi dev` で listen する address を変えるオプションを探し、変えると何が起きるか確かめる
- [ ] 2枚目の壁(WSL2 の NAT)は Phase 2 で掘る

### Q6. 同じ port で2つ起動するとどうなるか

**問い**

サーバーを起動したまま、別のターミナルでもう一度 `fastapi dev` を起動するとどうなるか。なぜそうなるのか。

**回答**

1. 起動しようとすると、使用済みの port で起動できない。
2. 同じ IP で同じ識別子で違うアプリを立ち上げたときに混ざってしまうから。

誰がエラーを返しているのか: OS のトランスポート層が投げてる

```bash
[Errno 98] Address already in use
```

メモ: `Isof` コマンドがある。

**フィードバック**

- 正解。「同じ IP と port に2つのアプリがいたら、どちらに渡せばいいか分からない」という理由付けが本質。
- 正確には、uvicorn が OS に `bind()` という system call で port を要求し、OS が `EADDRINUSE`(Errno 98)で断っている。
- `Errno` は、OS の system call が失敗したときに返すエラー番号。
- メモのコマンドは `lsof`(先頭は小文字の L、list open files の略)。どのプロセスが port を掴んでいるかを確認できる。

---

## C. 以降の学習の土台として

### Q7. 図の矢印の間に隠れている層

**問い**

ロードマップの図は `Browser → FastAPI → "Hello World"` と省略されている。矢印の間には実際にどんな層が隠れているか。Phase 1 以降の内容と対応づける。

**回答**

(未回答)

### Q8. `fastapi dev` を本番で使わない理由

**問い**

本番環境で `fastapi dev` を使わないほうが良い理由は何か。`dev` と `run` の違いを調べて確認しても OK。

**回答**

(未回答)
