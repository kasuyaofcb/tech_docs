# Supabase 全体像

> 📝 Supabase は「**バックエンドを丸ごと肩代わりしてくれるサービス**」。データベース・認証・ファイル保管・リアルタイム通信などを、サーバーを建てずに使えるようにする仕組み（BaaS = Backend as a Service）。中身は **PostgreSQL** という伝統的なDB。

---

## 1. Supabaseが提供するもの

```mermaid
flowchart TB
    APP["📱<br/><b>アプリ（フロント）</b><br/>React / Next.js / Flutter など"]

    subgraph SB["☁️ Supabase（バックエンド一式）"]
        direction LR
        DB["🗄️<br/><b>Database</b><br/>Postgres"]
        AUTH["🔐<br/><b>Auth</b><br/>ログイン管理"]
        ST["📁<br/><b>Storage</b><br/>ファイル保管"]
        RT["📡<br/><b>Realtime</b><br/>変更を即配信"]
        EF["⚡<br/><b>Edge Functions</b><br/>サーバー処理"]
        DB ~~~ AUTH ~~~ ST ~~~ RT ~~~ EF
    end

    APP ==>|"SDK / REST / GraphQL"| SB

    style APP fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style SB fill:#E8F8E8,stroke:#3ECF8E,stroke-width:3px,color:#000
    style DB fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style AUTH fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style ST fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style RT fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style EF fill:#F4C430,color:#000000,stroke:#333,stroke-width:2px
```

アプリはSupabaseが用意したSDKを呼ぶだけで、サーバーコードを書かなくても「DB読み書き／ログイン／ファイル保存」が動く。

> 📝 中心にあるのは **Postgres**。AuthもStorageもRealtimeも、内部的にはPostgresのテーブルに紐づいている。「全部Postgresの上に乗っている」と理解すると一気に見通しがよくなる。

---

## 2. よくある勘違い

```mermaid
flowchart TB
    subgraph WRONG["❌ 勘違い"]
        direction LR
        W1["Supabase は<br/>Firebase の別名"] ==> W2["NoSQL でしょ？"]
    end

    subgraph RIGHT["⭕ 実際"]
        direction LR
        R1["Supabase は"] ==> R2["中身は <b>Postgres</b><br/>普通のSQL"] ==> R3["テーブル / SQL / 外部キーが使える"]
    end

    style WRONG fill:#FFE5E5,stroke:#C0392B,stroke-width:2px,color:#000
    style RIGHT fill:#E8F8E8,stroke:#27AE60,stroke-width:2px,color:#000
    style W1 fill:#fff,color:#000,stroke:#C0392B
    style W2 fill:#fff,color:#000,stroke:#C0392B
    style R1 fill:#fff,color:#000,stroke:#27AE60
    style R2 fill:#fff,color:#000,stroke:#27AE60
    style R3 fill:#fff,color:#000,stroke:#27AE60
```

```mermaid
flowchart TB
    subgraph W2["❌ 勘違い"]
        direction LR
        WA["SDKで直接DBを叩く＝危険"] ==> WB["サーバー自作必須？"]
    end

    subgraph R2["⭕ 実際"]
        direction LR
        RA["フロントから直接叩いてOK"] ==> RB["<b>RLS（行レベルセキュリティ）</b>"] ==> RC["DB側でアクセス制御"]
    end

    style W2 fill:#FFE5E5,stroke:#C0392B,stroke-width:2px,color:#000
    style R2 fill:#E8F8E8,stroke:#27AE60,stroke-width:2px,color:#000
    style WA fill:#fff,color:#000,stroke:#C0392B
    style WB fill:#fff,color:#000,stroke:#C0392B
    style RA fill:#fff,color:#000,stroke:#27AE60
    style RB fill:#fff,color:#000,stroke:#27AE60
    style RC fill:#fff,color:#000,stroke:#27AE60
```

---

## 3. はじめの一歩

何から始めるのか、迷わないように流れだけ押さえる。

```mermaid
flowchart LR
    S1["① supabase.com<br/>サインアップ"] ==> S2["② New project<br/>リージョン・<br/>DBパスワード設定"]
    S2 ==> S3["③ Settings → API<br/><b>URL と anon key</b> をコピー"]
    S3 ==> S4["④ SDKをインストール"]
    S4 ==> S5["⑤ createClient<br/>で接続"]
    S5 ==> S6["⑥ テーブル操作開始"]

    style S1 fill:#3ECF8E,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3ECF8E,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S6 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 `createClient(URL, anon_key)` を呼ぶだけで接続完了。あとはテーブル名を指定してSDKを叩く、というのが基本動作。

### 💸 無料枠（Free tier）の主な制限

| 項目 | 上限 | 詰まったら |
| ---- | ---- | --------- |
| DB容量 | 500MB | 不要データ削除 or Pro($25/月) |
| ストレージ | 1GB | 同上 |
| 認証ユーザー数 | 5万 MAU | 余裕、まず当たらない |
| Edge Functions実行 | 50万回/月 | 大量呼び出しに注意 |
| 非アクティブ | 1週間使わないと一時停止 | ログインで復活 |

> 🚨 個人開発なら無料枠で十分。本番サービスに乗せるならPro以上推奨（バックアップ7日 → 7日以上に伸びる、自動停止なし）。

---

## 4. Database — Postgres と自動API

Supabaseで「テーブルを作る」と、**REST APIとGraphQL APIが自動で生える**。サーバーコードを書かずに、フロントから読み書きできる。

```mermaid
flowchart LR
    T["📋 テーブル作成<br/>(GUI or SQL)"] ==>|"自動生成"| API["🌐 REST / GraphQL API"]
    API ==> SDK["📦 supabase-js<br/>SDKから呼ぶ"]
    SDK ==> APP["📱 アプリで利用"]

    style T fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style API fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style SDK fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style APP fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
```

### SDKでできる4つの基本操作

```mermaid
flowchart TB
    Q["❓ やりたいこと"] ==> R["📥 <b>select</b><br/>読み出す"]
    Q ==> I["📤 <b>insert</b><br/>追加する"]
    Q ==> U["✏️ <b>update</b><br/>更新する"]
    Q ==> D["🗑️ <b>delete</b><br/>削除する"]

    style Q fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style R fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style I fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style U fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style D fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 内部では [PostgREST](https://postgrest.org/) がテーブル定義からAPIを動的に生成している。**「DBスキーマがそのままAPI仕様になる」**のがSupabaseの肝。

---

## 5. Row Level Security（RLS）— セキュリティの要

フロントから直接DBを叩けるなら、**他人のデータを盗み放題なのでは？** という疑問が出る。これを防ぐのが **RLS**。

```mermaid
flowchart LR
    R["🙋 リクエスト<br/>「自分のpostsをくれ」"] ==> RLS{"🔒 RLSポリシー<br/>auth.uid() = posts.user_id ?"}
    RLS ==>|"✅ 一致"| OK["📥 自分の行だけ返す"]
    RLS ==>|"❌ 不一致"| NG["🚫 返さない"]

    style R fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style RLS fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style OK fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style NG fill:#C0392B,color:#FFFFFF,stroke:#333,stroke-width:2px
```

RLSは「**この行を見せていいか／変えていいか**」を**ルールとしてDBに登録**しておく仕組み。Postgres本体の機能で、Supabaseが独自に実装したものではない。

### RLSのイメージ（postsテーブル）

```mermaid
flowchart TB
    subgraph T["📋 posts テーブル"]
        direction TB
        R1["id=1  user_id=A  title=日記1"]
        R2["id=2  user_id=A  title=日記2"]
        R3["id=3  user_id=B  title=日記3"]
        R4["id=4  user_id=C  title=日記4"]
    end

    UA["👤 ユーザーA<br/>でログイン"] ==> F{"RLS フィルタ<br/>user_id = A"}
    F ==>|"通過"| V1["✅ id=1, id=2 が見える"]
    F -.->|"はじく"| V2["❌ id=3, id=4 は存在しないように見える"]

    style T fill:#ECF0F1,stroke:#333,stroke-width:2px,color:#000
    style UA fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style V1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style V2 fill:#C0392B,color:#FFFFFF,stroke:#333,stroke-width:2px
```

### ポリシーは3つの要素でできている

```mermaid
flowchart LR
    P["📜 1つのポリシー"] ==> C1["🎯 <b>対象テーブル</b><br/>例: posts"]
    P ==> C2["🔧 <b>操作の種類</b><br/>SELECT / INSERT<br/>UPDATE / DELETE"]
    P ==> C3["✅ <b>許可する条件</b><br/>例: auth.uid() = user_id"]

    style P fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:3px
    style C1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C2 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 「**postsテーブルのSELECTは、`auth.uid() = user_id` の行だけ許可**」というルールを4種類（読む・作る・更新・削除）書けば、自分のデータだけ触れる状態が完成。

### ⚠️ 鉄則: RLSは「デフォルトで有効化」

```mermaid
flowchart LR
    S1["😱 RLSをオフのまま<br/>公開"] ==> S2["❌ 誰でも全行<br/>読み書き可能"]
    S2 ==> S3["✅ テーブルごとに<br/>RLSを有効化"]
    S3 ==> S4["✅ ポリシーを<br/>1つ以上登録"]

    style S1 fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 🚨 RLSを有効化しただけだと「**何も許可されていない＝全部拒否**」。**ポリシーを書いて初めて読み書きできる**。Supabaseダッシュボードは「RLS未有効テーブル」を警告してくれる。

---

## 6. Auth — 認証

Supabase Authは、メール／パスワード、Google、GitHub、マジックリンクなど多様なログインを**最初から用意**してくれる。ログインしたユーザーは `auth.users` テーブルに自動で記録される。

```mermaid
flowchart LR
    U["👤 ユーザー"] ==>|"signUp / signIn"| A["🔐 Supabase Auth"]
    A ==> JWT["🎫 JWT トークン発行"]
    JWT ==> SDK["📦 SDKに自動保持"]
    SDK ==>|"以降のリクエストに添付"| DB["🗄️ DB（RLSがJWT中のuser_idを参照）"]

    style U fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style JWT fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style SDK fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style DB fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
```

### Auth × RLS の連動

```mermaid
flowchart TB
    L["① ログイン操作"] ==> J["② JWT発行<br/>中身: { sub: ユーザーID, ... }"]
    J ==> Q["③ DBアクセス時、JWTが自動添付"]
    Q ==> P["④ RLS が <b>auth.uid()</b> で<br/>そのIDを参照"]
    P ==> R["⑤ 「自分の行だけ」が返る"]

    style L fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style J fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style Q fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 RLSポリシーの中で `auth.uid()` と書くと「**今アクセスしてきたユーザーのID**」を取れる。これがAuthとDBをつなぐ橋。

---

## 7. `auth.users` と `public.profiles` — 定番パターン

Supabaseで**みんな最初にハマる**のがこれ。「ユーザー名やアバターをどこに置く？」問題。

### ❌ よくある勘違い

```mermaid
flowchart LR
    W["😱 auth.users に<br/>username を追加すれば<br/>いいんじゃない？"] ==> X["❌ auth.users は<br/>Supabase管理スキーマ<br/>勝手にカラム足せない"]

    style W fill:#FFE5E5,color:#000,stroke:#C0392B,stroke-width:2px
    style X fill:#C0392B,color:#FFFFFF,stroke:#333,stroke-width:2px
```

### ⭕ 定番パターン: `public.profiles` を作って外部キーで紐づける

```mermaid
flowchart LR
    subgraph A["🔐 auth スキーマ（Supabase管理）"]
        AU["auth.users<br/>id / email / 認証情報"]
    end

    subgraph P["📋 public スキーマ（自分の領域）"]
        PR["public.profiles<br/>id (FK) / username / avatar_url / bio"]
    end

    AU ==>|"1対1<br/>id で連結"| PR

    style A fill:#F4ECF7,stroke:#8E44AD,stroke-width:2px,color:#000
    style P fill:#E8F4FD,stroke:#3498DB,stroke-width:2px,color:#000
    style AU fill:#fff,color:#000,stroke:#8E44AD
    style PR fill:#fff,color:#000,stroke:#3498DB
```

### サインアップ時に自動でprofilesを作る

「`auth.users` に行が追加されたら、`public.profiles` にも同じIDで行を自動作成する」をDB側で仕込んでおく（**トリガー**という仕組み）。これで抜け漏れがなくなる。

```mermaid
sequenceDiagram
    participant U as 👤 ユーザー
    participant A as 🔐 auth.users
    participant T as ⚡ トリガー
    participant P as 📋 public.profiles

    U->>A: サインアップ
    A->>T: 行が追加されたぞ！
    T->>P: 同じidで行を自動作成
    Note over P: ✨ 抜け漏れゼロ
```

### 何が嬉しいの？

```mermaid
flowchart TB
    Q["✨ メリット"] ==> M1["✅ <b>サインアップ＝profiles自動生成</b><br/>抜け漏れゼロ"]
    Q ==> M2["✅ <b>退会＝profiles自動削除</b><br/>(外部キーのcascade)"]
    Q ==> M3["✅ <b>username / bio など<br/>自由にカラム追加</b>"]
    Q ==> M4["✅ <b>他テーブルからは<br/>profiles を参照</b>"]

    style Q fill:#3ECF8E,color:#FFFFFF,stroke:#333,stroke-width:3px
    style M1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style M2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style M3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style M4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 公式テンプレートやチュートリアルもほぼ全部このパターン。**「Supabaseの作法」**として覚えてしまうのが早い。

---

## 8. Storage — ファイル保管

画像・動画・PDFなど、DBに入れたくない大きなファイルを置く場所。**バケット（bucket）** 単位で管理する。

```mermaid
flowchart TB
    F["📷 画像ファイル"] ==>|"upload"| B1["🪣 バケット: avatars"]
    F2["📄 PDF"] ==>|"upload"| B2["🪣 バケット: documents"]

    B1 ==> P["🔒 RLS的なポリシーで<br/>誰が読めるか制御"]
    B2 ==> P

    P ==>|"public bucket"| U1["🌐 URLで誰でも閲覧"]
    P ==>|"private bucket"| U2["🔑 署名付きURLが必要"]

    style F fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F2 fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B1 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B2 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style U1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style U2 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 Storage も内部はPostgresのテーブルでメタデータを持っている。**バケット／オブジェクトに対してRLS的なポリシーが書ける**ので、「自分のアバターは自分しか上書きできない」みたいな制御が同じ発想でできる。

### publicバケットとprivateバケットの使い分け

```mermaid
flowchart TB
    Q["❓ ファイルをどう公開する？"] ==> A{"判断"}
    A ==>|"誰でも見ていいもの<br/>(プロフ画像など)"| B1["🌐 <b>public バケット</b><br/>固定URLで配信"]
    A ==>|"本人/権限者だけ<br/>(個人書類など)"| B2["🔑 <b>private バケット</b><br/>短時間有効な署名URL"]

    style Q fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style A fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style B1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B2 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

---

## 9. Realtime — 変更を即配信

DBの変更（INSERT/UPDATE/DELETE）を**WebSocketで購読**できる。チャット、共同編集、ダッシュボードなど「即時反映」が必要な用途で使う。

```mermaid
sequenceDiagram
    participant A as ユーザーA
    participant DB as Postgres
    participant RT as Realtime
    participant B as ユーザーB

    B->>RT: subscribe('messages')
    A->>DB: insert('messages', { text: 'Hi' })
    DB-->>RT: 変更を検知
    RT-->>B: 📡 新しい行を即配信
    Note over B: 画面に「Hi」が自動表示
```

```mermaid
flowchart LR
    DB["🗄️ Postgres<br/>WAL（変更ログ）"] ==>|"監視"| RT["📡 Realtime サーバー"]
    RT ==>|"WebSocket"| C1["📱 クライアント1"]
    RT ==>|"WebSocket"| C2["📱 クライアント2"]
    RT ==>|"WebSocket"| C3["📱 クライアント3"]

    style DB fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style RT fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C1 fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C2 fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C3 fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 仕組み的にはPostgresの**論理レプリケーション（WAL）**を読んで配信している。テーブルごとに「Realtime対象にするか」をDashboardで有効化して使う。

---

## 10. Edge Functions — サーバー処理を書きたいとき

「外部APIを叩く」「決済の確定処理」「Webhookを受ける」など、**フロントに置きたくないロジック**はEdge Functions（Deno製）で書ける。

```mermaid
flowchart LR
    APP["📱 アプリ"] ==>|"functions.invoke('xxx')"| EF["⚡ Edge Function<br/>(Deno / TS)"]
    EF ==>|"必要なら"| DB["🗄️ DB"]
    EF ==>|"外部API"| EXT["🌐 Stripe / OpenAI など"]
    EF ==>|"レスポンス"| APP

    style APP fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style EF fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style DB fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style EXT fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

### いつ Edge Functions を使う？

```mermaid
flowchart TB
    Q["❓ サーバー処理が必要？"] ==> A{"判断"}
    A ==>|"DB読み書きだけ"| N1["❌ 不要<br/>SDKで直接でOK"]
    A ==>|"APIキーを秘匿したい"| Y1["✅ Edge Function"]
    A ==>|"Webhook受信"| Y2["✅ Edge Function"]
    A ==>|"重い集計／メール送信"| Y3["✅ Edge Function"]

    style Q fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style A fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style N1 fill:#7F8C8D,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Y1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Y2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Y3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

---

## 11. ローカル開発と Migration（CLI）

Supabaseは**ローカルで丸ごと動かせる**（Docker）。本番DBと同じ構造を手元で再現し、スキーマ変更をマイグレーションファイルとしてGitで管理する。

```mermaid
flowchart LR
    L["💻 ローカル<br/>(supabase start)"] ==>|"開発・実験"| C["✏️ スキーマ変更"]
    C ==>|"supabase db diff"| M["📜 マイグレーションSQL生成"]
    M ==>|"git commit / push"| GH["☁️ GitHub"]
    GH ==>|"supabase db push"| P["🚀 本番Supabase"]

    style L fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style M fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style GH fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

### よく使うCLIコマンド

| コマンド | 説明 |
| -------- | ---- |
| `supabase init` | プロジェクトに `supabase/` フォルダーを作成 |
| `supabase start` | ローカルでDB・Auth・StorageをDocker起動 |
| `supabase stop` | ローカル環境を停止 |
| `supabase db diff -f 名前` | 現在のスキーマ差分をマイグレーションSQLに書き出す |
| `supabase db reset` | ローカルDBを初期化し、マイグレーションを再適用 |
| `supabase link --project-ref xxx` | リモートプロジェクトと紐付け |
| `supabase db push` | ローカルのマイグレーションを本番に適用 |
| `supabase gen types typescript` | DBスキーマからTypeScript型を生成 |

> 📝 イメージは [git.md](../git/git.md) と同じ。**ローカルで作って → 差分を記録 → リモートに反映**。コードだけでなく**DBスキーマもGit管理**するのがモダンな運用。

---

## 12. APIキーの種類 — anon と service_role

Supabaseは2種類のAPIキーを発行する。**取り違えると事故になる**ので最重要ポイント。

```mermaid
flowchart TB
    K1["🔑 <b>anon key</b><br/>公開キー"] ==> U1["📱 フロントエンドに置いてOK<br/>(RLSで守られているため)"]
    K2["🔑 <b>service_role key</b><br/>管理者キー"] ==> U2["🖥️ サーバー側のみ<br/><b>RLSを完全に無視</b>する"]

    U2 ==> W["⚠️ フロントに置いたら<br/>全データ漏洩"]

    style K1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style K2 fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
    style U1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style U2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style W fill:#C0392B,color:#FFFFFF,stroke:#333,stroke-width:3px
```

### 鉄則

```mermaid
flowchart LR
    R1["✅ anon key"] ==> R1A["フロント / public OK"]
    R2["❌ service_role key"] ==> R2A["絶対に .env で<br/>サーバー側だけ"]

    style R1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R1A fill:#fff,color:#000,stroke:#27AE60,stroke-width:2px
    style R2 fill:#C0392B,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R2A fill:#fff,color:#000,stroke:#C0392B,stroke-width:2px
```

> 🚨 `service_role` をGitにcommitしてしまったら、**そのキーは即座にSupabaseダッシュボードで再発行**。漏れたキーは取り戻せない。[git.md](../git/git.md) §8の `.gitignore` の話とセットで覚える。

---

## 13. 迷ったらどこを見る？

```mermaid
flowchart LR
    Q["❓ 困った"] ==> S["<b>Dashboard → Table Editor</b><br/>📋 データを直接確認"]
    Q ==> D["<b>Dashboard → SQL Editor</b><br/>🔍 SQLを叩いて検証"]
    Q ==> L["<b>Dashboard → Logs</b><br/>📜 API / Auth / Postgresのログ"]
    Q ==> A["<b>Dashboard → Authentication → Policies</b><br/>🔐 RLSポリシーを確認"]

    style Q fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style S fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style D fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style L fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

「データが取れない」のほとんどは**RLSポリシー不足**。まずPoliciesを疑う。

---

## 14. 補足: Firebase との違い

```mermaid
flowchart TB
    subgraph FB["🔥 Firebase"]
        direction TB
        FB1["Firestore（NoSQL）"]
        FB2["独自のクエリ言語"]
        FB3["ベンダーロック強め"]
    end

    subgraph SP["⚡ Supabase"]
        direction TB
        SP1["Postgres（SQL）"]
        SP2["標準SQL / SQLが書ける"]
        SP3["OSS / セルフホスト可"]
    end

    style FB fill:#FFF4E5,stroke:#E67E22,stroke-width:2px,color:#000
    style SP fill:#E8F8E8,stroke:#3ECF8E,stroke-width:2px,color:#000
    style FB1 fill:#fff,color:#000,stroke:#E67E22
    style FB2 fill:#fff,color:#000,stroke:#E67E22
    style FB3 fill:#fff,color:#000,stroke:#E67E22
    style SP1 fill:#fff,color:#000,stroke:#3ECF8E
    style SP2 fill:#fff,color:#000,stroke:#3ECF8E
    style SP3 fill:#fff,color:#000,stroke:#3ECF8E
```

**結論: 「Firebaseの代替」と言われがちだが、思想は別物。**

- Firebase: NoSQLとリアルタイムが主役、独自エコシステム
- Supabase: 伝統的なSQL（Postgres）にBaaSの皮を被せたもの
- 「将来データをエクスポートして他のPostgresに引っ越し」も普通にできる
- 既存のSQL知識・PostgresのナレッジがそのままSupabaseに通用する

> 📝 「**Postgres 1個に、Auth/Storage/Realtime/Functions を全部生やしたもの**」というのが一番しっくりくる説明。
