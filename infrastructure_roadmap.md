# Webサービスの土台を手を動かして学ぶロードマップ

## 方針

アプリケーションそのものを作り込むのではなく、**ほぼ空のWebアプリを土台に載せながら、Webサービスが動く仕組みを上から下まで理解する**。

最終的な目標は、単にAWSやDockerの使い方を覚えることではなく、

> 「ブラウザからリクエストが届いて、サーバー上のプログラムが動き、DBにアクセスし、レスポンスが返るまで何が起きているのか」

を自分で説明・構築・調査できるようになること。

---

# 全体ロードマップ

```text
Linux / OS
    ↓
ネットワーク
    ↓
HTTP / Web
    ↓
Webサーバー / Reverse Proxy
    ↓
Database
    ↓
Docker / Container
    ↓
Security
    ↓
Cloud
    ↓
CI/CD
    ↓
Infrastructure as Code
    ↓
Observability
    ↓
障害対応 / SRE
```

---

# Phase 0：学習用の空アプリを作る

## やること

- FastAPIなどで最小限のHTTPサーバーを作る
- `/` にアクセスすると `Hello World` が返るだけにする
- Gitで管理する
- READMEに構成を書いておく

アプリは極力作り込まない。

```text
Browser
   ↓
FastAPI
   ↓
"Hello World"
```

## 分かるようになること

- Webアプリとは何か
- サーバー上でプログラムが動くとはどういうことか
- 以降の学習で「何を土台にしているのか」が分かる

---

# Phase 1：Linux / OS

## やること

Linux VMを1台用意して、そこに空アプリを動かす。

- SSH
- ユーザー
- ファイル・ディレクトリ
- 権限
- プロセス
- PID
- CPU / メモリ
- ファイルディスクリプタ
- シグナル
- systemd
- journalctl
- 環境変数
- ログ
- cron / timer

## 実験

- アプリを手動起動する
- プロセスを探す
- プロセスをkillする
- systemdでサービス化する
- ログを見る
- 権限を変えて動かなくしてみる
- CPU / メモリを使わせる

## 分かるようになること

**「サーバー」とは特別な機械ではなく、プログラムが動いているLinuxマシンである**

という感覚。

---

# Phase 2：ネットワーク

## やること

- IPアドレス
- IPv4
- subnet / CIDR
- MACアドレス
- TCP / UDP
- ポート
- socket
- DNS
- routing
- NAT
- localhost
- firewall

## 実験

- `ping`
- `curl`
- `ss`
- `ip`
- `dig`
- `traceroute`
- `nc`

などを使って通信を観察する。

例えば、

```text
Browser
   ↓
IP
   ↓
TCP
   ↓
Port 80 / 443
   ↓
Server
```

を実際に確認する。

## 分かるようになること

- IPとは何か
- ポートとは何か
- TCP接続とは何か
- 「接続できない」がどの層の問題なのか

---

# Phase 3：HTTP / Web

## やること

- HTTP request / response
- GET / POST
- Header
- Body
- Status Code
- Cookie
- Keep-Alive
- HTTPS
- TLS
- Certificate

## 実験

`curl`を中心にHTTP通信を観察する。

```text
GET /
Host: example.com
User-Agent: ...
```

↓

```text
HTTP/1.1 200 OK
Content-Type: ...
...
```

のように、生の通信を意識する。

## 分かるようになること

ブラウザでURLを開くという行為が、実際には何をしているのか。

---

# Phase 4：Web Server / Reverse Proxy

## やること

Nginxなどを導入する。

```text
Internet
   ↓
Nginx
   ↓
FastAPI
```

- Web Server
- Reverse Proxy
- Static File
- Proxy
- Port forwarding
- HTTPS termination

## 実験

- Nginxを立てる
- FastAPIをlocalhostで動かす
- NginxからFastAPIへproxyする
- HTTP → HTTPSにする
- 設定を壊して原因を調査する

## 分かるようになること

「ブラウザ → アプリ」が直接繋がっているわけではなく、その間に様々な役割を持つソフトウェアが存在すること。

---

# Phase 5：Database

## やること

PostgreSQLなどを導入する。

- RDB
- Table
- SQL
- Connection
- User / Role
- Transaction
- Index
- Migration
- Backup
- Restore

## 構成

```text
Nginx
  ↓
FastAPI
  ↓
PostgreSQL
```

## 実験

- DBを作る
- Appから接続する
- データを保存する
- DBを止める
- 接続エラーを調査する
- Backupを取る
- Restoreする

## 分かるようになること

アプリケーションとデータベースの関係。

さらに、

> 「データを保存する」

だけではなく、

> 「壊れたらどう復旧する？」

まで考えられるようになる。

---

# Phase 6：Docker / Container

## やること

これまで作った環境をDocker化する。

```text
Docker Compose

┌─────────────┐
│    Nginx    │
└──────┬──────┘
       ↓
┌─────────────┐
│   FastAPI   │
└──────┬──────┘
       ↓
┌─────────────┐
│ PostgreSQL  │
└─────────────┘
```

- Image
- Container
- Dockerfile
- Volume
- Network
- Compose
- Environment Variable

## さらに知る

余裕が出たら、

- namespace
- cgroups
- overlay filesystem

など、Linux側の仕組みに戻って理解する。

## 分かるようになること

- コンテナとは何か
- VMとの違い
- コンテナ同士がどう通信するか
- なぜ「同じ環境」を作れるのか

---

# Phase 7：Security

## やること

これまで作った環境を安全にする。

- SSH key
- Firewall
- least privilege
- User / Group
- Secret
- HTTPS
- TLS
- Security Headers
- Network isolation

## 実験

- 不要なポートを閉じる
- rootで動かさない
- DBを外部公開しない
- SecretをGitに入れてしまう危険を確認する
- HTTPSを構築する

## 分かるようになること

「動けばOK」ではなく、

**「誰が、どこから、何にアクセスできるのか」**

という視点。

---

# Phase 8：Cloud

## やること

AWS / GCPなどに環境を移す。

まずはVMを使って、

```text
Local Linux
    ↓
Cloud VM
```

という対応関係を理解する。

その後、

- VPC
- Subnet
- Security Group
- Load Balancer
- Object Storage
- Managed Database
- DNS
- CDN

などを触る。

## 目標構成

```text
                Internet
                    ↓
                   DNS
                    ↓
              Load Balancer
                    ↓
              ┌─────┴─────┐
              ↓           ↓
            App 1       App 2
              │           │
              └─────┬─────┘
                    ↓
                 Database
                    ↓
                 Storage
```

## 分かるようになること

クラウドサービスの名前を覚えるのではなく、

**自分で構築したLinux / Network / Webの仕組みが、クラウド上ではどう抽象化されているのか**

が分かる。

---

# Phase 9：CI/CD

## やること

GitHub Actionsなどで、

```text
git push
   ↓
Test
   ↓
Build
   ↓
Deploy
```

を自動化する。

- CI
- CD
- Build
- Artifact
- Deployment
- Rollback
- Environment

## 実験

- pushすると自動test
- mainにmergeするとdeploy
- deploy失敗
- rollback

などを試す。

## 分かるようになること

「コードを書いた後、どうやって安全に本番へ届けるのか」。

---

# Phase 10：Infrastructure as Code

## やること

Terraformなどを使う。

例えば、

```text
terraform
    ↓
Network
    ↓
Server
    ↓
Database
    ↓
Load Balancer
```

をコードで構築する。

## 実験

- 手動で作る
- Terraformで作る
- destroyする
- 同じ環境を再構築する
- Gitで変更を管理する

## 分かるようになること

インフラもソースコードと同じように、

- version control
- review
- reproducibility
- automation

できること。

---

# Phase 11：Observability

## やること

サービスを「動かす」だけではなく、「状態を知る」。

### Logs

```text
何が起きた？
```

### Metrics

```text
どのくらい起きている？
```

### Traces

```text
どこで時間がかかっている？
```

- Logging
- Metrics
- Tracing
- Monitoring
- Alerting
- Dashboard

## 実験

わざと、

- Appを落とす
- DBを落とす
- CPUを使わせる
- メモリを使わせる
- レスポンスを遅くする

などを行う。

そして、

> どうやって異常を検知する？

を考える。

---

# Phase 12：SRE / Reliability

ここまで来て、ようやくSRE的な考え方を本格的に扱う。

## やること

- Availability
- Reliability
- SLI
- SLO
- SLA
- Incident Response
- Error Budget
- Capacity Planning
- Disaster Recovery
- Backup
- Rollback
- Postmortem

## 実験

自分のサービスに障害を起こして、

```text
障害発生
   ↓
検知
   ↓
原因調査
   ↓
影響範囲確認
   ↓
復旧
   ↓
再発防止
```

を実際にやる。

## 分かるようになること

「サーバーを立てられる」から、

**「サービスを安定して運用できる」**

へ進む。

---

# 最終的に身につくもの

このロードマップを一周すると、以下を横断して理解できる。

```text
                    Web Service
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
      Linux           Network          Database
        │                │                │
        └────────────────┼────────────────┘
                         ↓
                       Docker
                         ↓
                       Cloud
                         ↓
                    CI / CD / IaC
                         ↓
                 Observability
                         ↓
                       SRE
```

## 特に重要なのは「技術の地図」

例えば、

- Dockerを触った
- AWSを触った
- Nginxを触った
- PostgreSQLを触った

という個別の経験だけで終わらせない。

```text
「これは何を解決する技術なのか？」

「その下では何が動いているのか？」

「これが壊れたら何が起きるのか？」

「別の方法ではどう実現できるのか？」
```

まで考える。

---

# 学習時の基本ルール

## 1. アプリを作り込まない

アプリは、

```text
GET /
→ Hello World
```

くらいで十分。

学習対象はその周辺。

## 2. できるだけ自分で構築する

最初から全部マネージドサービスに任せない。

例えば、

> 「DBを作る」

だけではなく、

> 「DBはどこで動いていて、誰が接続できて、データはどこに保存されている？」

まで見る。

## 3. 壊す

正常動作だけではインフラは理解しにくい。

意図的に壊して、

```text
何が壊れた？
↓
どう気づく？
↓
どう調べる？
↓
どう直す？
```

を繰り返す。

## 4. 一つ前の層に戻る

例えばDockerでネットワークが分からなくなったら、

```text
Docker
 ↓
Network Namespace
 ↓
Linux Network
 ↓
TCP/IP
```

まで戻る。

これを繰り返すと知識が繋がっていく。

---

# ゴール

最終的に、次の構成を**自分で構築し、説明し、壊して、復旧できる状態**を目指す。

```text
                         Internet
                            │
                           DNS
                            │
                           HTTPS
                            │
                     Load Balancer
                            │
                    ┌───────┴───────┐
                    ↓               ↓
                 App 1           App 2
                    │               │
                    └───────┬───────┘
                            ↓
                         Database
                            │
                         Storage

          ┌─────────────────────────────────┐
          │ CI/CD                           │
          │ Infrastructure as Code          │
          │ Monitoring / Logging / Tracing  │
          │ Backup / Recovery               │
          └─────────────────────────────────┘
```

**Webサービスがコンピュータ・OS・ネットワーク・Web・DB・クラウド・自動化・運用によってどう成立しているかを一通り理解した**

と言える状態に近づく。
