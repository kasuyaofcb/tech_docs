# アプリリリース全体像

> 📝 「Webアプリを作って世に出す」までに登場する**全部の登場人物**と、それらが**どう繋がっているか**を1枚で見渡すための資料。各サービスの詳細は個別資料に任せ、ここでは **「結局、何と何を組み合わせれば1個のアプリが世に出せるのか」** の地図を提供する。

---

## 1. 全体像 — 登場人物マップ

```mermaid
flowchart TB
    subgraph LOCAL["🏠 開発者の手元（ローカル環境）"]
        DEV["👨‍💻<br/><b>開発者</b><br/>エディタ + Node.js"]
    end

    subgraph SCM["🔀 バージョン管理"]
        GH["☁️<br/><b>GitHub</b><br/>ソースコード保管<br/>PR / レビュー"]
    end

    subgraph BACK["🗄️ バックエンド（BaaS）"]
        SB["🟢<br/><b>Supabase</b><br/>DB / Auth / Storage"]
    end

    subgraph FRONT["⚡ フロントエンド配信"]
        VC["▲<br/><b>Vercel</b>"]
        AZ["🔷<br/><b>Azure Static<br/>Web Apps</b>"]
    end

    USER["📱<br/><b>エンドユーザー</b>"]

    DEV ==>|"git push"| GH
    GH ==>|"Webhook"| FRONT
    FRONT ==>|"ビルド済み<br/>HTML/JS配信"| USER
    USER ==>|"DB読み書き<br/>(SDK経由)"| SB
    DEV -.->|"ローカルでも<br/>接続して開発"| SB

    style LOCAL fill:#FFF4E5,stroke:#E67E22,stroke-width:2px,color:#000
    style SCM fill:#F4ECF7,stroke:#8E44AD,stroke-width:2px,color:#000
    style BACK fill:#E8F8E8,stroke:#3ECF8E,stroke-width:2px,color:#000
    style FRONT fill:#F0F0F0,stroke:#000,stroke-width:2px,color:#000

    style DEV fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style GH fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style SB fill:#3ECF8E,color:#FFFFFF,stroke:#333,stroke-width:2px
    style VC fill:#000,color:#FFFFFF,stroke:#333,stroke-width:2px
    style AZ fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style USER fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 **Vercel と Azure Static Web Apps は二者択一**。どちらか1つを選ぶ（両方同時には使わない）。配信先以外の登場人物（ローカル / GitHub / Supabase）は共通。

---

## 2. 推奨学習順

各資料をどの順で読めば理解が積み上がるかの地図。

```mermaid
flowchart LR
    S0["📖 <b>app-release</b><br/>(この資料)<br/>全体像をつかむ"] ==> S1["📖 <b>git</b><br/>バージョン管理の<br/>基本動作"]
    S1 ==> S2["📖 <b>supabase</b><br/>バックエンド<br/>(DB/Auth)"]
    S2 ==> S3{"配信先を<br/>選ぶ"}
    S3 ==>|"海外SaaS派<br/>(個人開発)"| S4A["📖 <b>vercel</b>"]
    S3 ==>|"MS縛り派<br/>(企業案件)"| S4B["📖 <b>azure</b><br/>↓<br/>📖 <b>azure/setup</b>"]
    S4A ==> S5["🚀 自分のアプリを<br/>世に出す"]
    S4B ==> S5

    style S0 fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style S1 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3ECF8E,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style S4A fill:#000,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S4B fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

> 📝 Git → Supabase → 配信先 の順で読むと、「ローカルで動くものをどう外の世界に出すか」が段階的に理解できる。Vercel と Azure は**どちらか片方だけでOK**。

---

## 3. 役割マップ — 誰が何を担当するか

「自分でやらないこと」を把握するのが大事。**サービスに任せる部分**と**自分で書く部分**の境界線を見る。

| 仕事 | 担当 | 自分で書くか |
| ---- | ---- | ----------- |
| アプリのソースコード | 自分 | ✅ |
| バージョン管理 | **Git/GitHub** | ❌（コマンドだけ覚える）|
| DBの構築・運用 | **Supabase** | ❌（テーブル設計だけ）|
| ユーザー認証（ログイン機能） | **Supabase Auth** | ❌（呼び出すだけ）|
| ファイル保管（画像など） | **Supabase Storage** | ❌（呼び出すだけ）|
| ビルド（ソース→公開用変換） | **Vercel / Azure** | ❌ |
| HTTPS化・SSL証明書 | **Vercel / Azure** | ❌（自動）|
| 世界中への配信（CDN） | **Vercel / Azure** | ❌（自動）|
| PR毎の検証環境 | **Vercel / Azure** | ❌（自動）|
| 本番への反映 | **Vercel / Azure** | ❌（mainマージで自動）|
| アプリのUI/UXを考える | 自分 | ✅ |
| データの構造を設計する | 自分 | ✅ |

> 📝 **「自分で書くのはコード本体とDB設計だけ」**。それ以外のインフラ仕事はサービスが肩代わりしてくれる、というのが現代のアプリ開発スタイル。

---

## 4. データの流れ — ユーザー操作からDB保存まで

例: ユーザーが「投稿ボタン」を押した時に何が起きるか。

```mermaid
sequenceDiagram
    participant U as 📱 ユーザー
    participant FR as ⚡ Vercel/Azure<br/>(フロント配信)
    participant SB as 🟢 Supabase<br/>(DB)

    U->>FR: アクセス（最初の表示）
    FR-->>U: HTML/JS を返す
    Note over U: ブラウザでアプリ起動
    U->>SB: 投稿データを送信<br/>(SDK経由・JWT付き)
    Note over SB: RLSで権限チェック
    SB-->>U: 保存成功
    Note over U: 画面に反映
```

ポイント:

- 最初の **HTML/JSの配信**は Vercel/Azure から
- それ以降の **データ通信**は ユーザーのブラウザ ↔ Supabase が直接やり取り
- **Vercel/Azureは中継しない**（フロント配信専門）

> 📝 つまりVercel/Azureが落ちてもDBは生きている、Supabaseが落ちても画面は表示できる、と**障害ドメインが分かれている**。これは利点でもあり、トラブルシュート時に「どっち側の問題か」を見分ける勘所でもある。

---

## 5. 環境変数の流れ — 秘密情報の旅

APIキーやDB接続先などの**秘密情報がどこから来てどこで使われるか**。3つの場所に同じ値を入れることになる。

```mermaid
flowchart LR
    subgraph L["🏠 ローカル開発"]
        ENV[".env<br/>(Gitに含めない)"]
    end

    subgraph C["☁️ GitHub"]
        GHS["Actions Secrets<br/>(CI/CD用)"]
    end

    subgraph P["🚀 本番（Vercel/Azure）"]
        VENV["Environment Variables<br/>管理画面で設定"]
    end

    SB1["🟢 Supabase Dashboard<br/>(APIキー発行元)"]

    SB1 ==>|"コピペ"| ENV
    SB1 ==>|"コピペ"| GHS
    SB1 ==>|"コピペ"| VENV

    ENV -.->|"ローカル実行時"| APP1["💻 ローカルアプリ"]
    GHS -.->|"テスト/ビルド時"| APP2["🧪 CI"]
    VENV -.->|"本番実行時"| APP3["🌐 本番アプリ"]

    style L fill:#FFF4E5,stroke:#E67E22,stroke-width:2px,color:#000
    style C fill:#F4ECF7,stroke:#8E44AD,stroke-width:2px,color:#000
    style P fill:#E8F8E8,stroke:#27AE60,stroke-width:2px,color:#000
    style SB1 fill:#3ECF8E,color:#FFFFFF,stroke:#333,stroke-width:3px
    style ENV fill:#fff,color:#000,stroke:#E67E22
    style GHS fill:#fff,color:#000,stroke:#8E44AD
    style VENV fill:#fff,color:#000,stroke:#27AE60
```

### 鉄則

```mermaid
flowchart TB
    R1["🔐 秘密情報は<br/><b>3つの場所に分けて持つ</b>"] ==> R2["✅ .env<br/>(.gitignoreで除外)"]
    R1 ==> R3["✅ GitHub Secrets<br/>(CI用)"]
    R1 ==> R4["✅ Vercel/Azure Env Vars<br/>(本番用)"]
    R1 ==> R5["❌ <b>絶対にコードに直書きしない</b>"]
    R1 ==> R6["❌ <b>絶対にGitにcommitしない</b>"]

    style R1 fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style R2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R5 fill:#C0392B,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R6 fill:#C0392B,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 🚨 一度Gitにcommitした秘密情報は、**過去のコミット履歴に永遠に残る**。即座にSupabaseダッシュボードで再発行 → 古いキーは無効化、が正しい対処。詳細は [git.md](../02_git/git.md) §8、[supabase.md](../03_supabase/supabase.md) §12 参照。

---

## 6. 開発からリリースまで — 1日の作業フロー

「あるユーザーが投稿削除機能を追加する」という典型シナリオを通して、各サービスがどのタイミングで登場するかを見る。

```mermaid
flowchart TB
    Start["🌅 朝、出社"] ==> S1["① <b>git checkout -b feature/delete</b><br/>新しいブランチ作成"]
    S1 ==> S2["② エディタでコード編集<br/>必要ならローカルでsupabase起動"]
    S2 ==> S3["③ <b>git add → commit → push</b><br/>変更をGitHubに送る"]
    S3 ==> S4["④ <b>GitHubでPR作成</b>"]
    S4 ==> S5["⑤ ⚡ <b>Vercel/Azureが自動で<br/>Preview URLを発行</b>"]
    S5 ==> S6["⑥ チームメンバーが<br/>Preview URLで動作確認<br/>+ コードレビュー"]
    S6 ==> S7{"OK？"}
    S7 ==>|"修正依頼"| S2
    S7 ==>|"Approve"| S8["⑦ <b>mainにマージ</b>"]
    S8 ==> S9["⑧ 🚀 <b>本番に自動デプロイ</b>"]
    S9 ==> End["🎉 機能リリース完了"]

    style Start fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S1 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S4 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#000,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S6 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S7 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style S8 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S9 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style End fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

> 📝 ⑤と⑨は**人間が何もしなくても勝手に起きる**。これが「現代のCI/CD」の正体。詳細は [git.md](../02_git/git.md) §4・[vercel.md](../04a_vercel/vercel.md) §5・[azure/azure.md](../04b_azure/azure.md) で。

---

## 7. ローカル / Preview / 本番 — 3つの環境

開発中に意識する「**3つの場所**」。それぞれ役割が違う。

```mermaid
flowchart LR
    L["🏠 <b>ローカル</b><br/>自分のPC<br/>localhost:3000"] ==>|"git push"| PR["🔗 <b>Preview</b><br/>PRごとに自動発行<br/>xxx-pr-123.vercel.app"]
    PR ==>|"mainマージ"| PD["🌐 <b>本番</b><br/>みんなが見るURL<br/>myapp.com"]

    L -.->|"開発用DB"| DBL["🗄️ ローカルDB<br/>or Supabase開発プロジェクト"]
    PR -.->|"テスト用DB"| DBP["🗄️ Supabase開発プロジェクト"]
    PD -.->|"本番DB"| DBR["🗄️ Supabase本番プロジェクト"]

    style L fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style PR fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style PD fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style DBL fill:#fff,color:#000,stroke:#FF8C42
    style DBP fill:#fff,color:#000,stroke:#8E44AD
    style DBR fill:#fff,color:#000,stroke:#27AE60
```

| 環境 | 誰が触る | DBは | 壊れたら | 環境変数の置き場 |
| ---- | -------- | ---- | -------- | --------------- |
| ローカル | 自分だけ | ローカルDB or 開発用Supabase | 自分のPCを直すだけ | `.env` |
| Preview | レビュアー / QA | 開発用Supabase | 該当PRを直す | Vercel/Azure管理画面 |
| 本番 | エンドユーザー | 本番Supabase | 即ロールバック → 修正PR | Vercel/Azure管理画面 |

> 🚨 **本番DBを開発で触らない**こと。PreviewやローカルからはSupabaseの**別プロジェクト**を見るように環境変数で切り替える。「Previewで誤って本番DBを破壊」は最悪のシナリオ。

---

## 8. 配信先の選び方 — Vercel か Azure か

最初に「どっちで配信するか」を決める必要がある。判断軸は3つ。

```mermaid
flowchart TB
    Q["❓ どちらを選ぶ？"] ==> A{"判断"}
    A ==>|"個人開発・趣味<br/>or 商用OKな個人案件"| V["▲ <b>Vercel</b><br/>セットアップ最速<br/>体験が滑らか<br/>(商用はPro $20/月)"]
    A ==>|"会社がMS縛り<br/>or 商用Free必須<br/>or AzureのAD連携"| AZ["🔷 <b>Azure Static Web Apps</b><br/>商用もFree tier<br/>Microsoft連携が濃い"]
    A ==>|"安さ最優先・<br/>帯域多め"| CF["🟧 Cloudflare Pages<br/>(別資料予定)"]

    style Q fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style A fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style V fill:#000,color:#FFFFFF,stroke:#333,stroke-width:2px
    style AZ fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style CF fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| 観点 | Vercel | Azure Static Web Apps |
| --- | --- | --- |
| セットアップの楽さ | ⭐⭐⭐ | ⭐⭐ |
| 体感の速さ・UI | ⭐⭐⭐ | ⭐⭐ |
| Free tierでの商用利用 | ❌ Pro必須 | ✅ OK |
| 大企業案件との親和性 | △ | ⭐⭐⭐ |
| ドキュメント量 | ⭐⭐⭐ | ⭐⭐ |
| Next.jsとの相性 | ⭐⭐⭐（本家） | ⭐⭐ |

> 📝 迷ったら **個人で試すならVercel、仕事ならAzure** くらいの大雑把な分け方でOK。アプリの中身（コード）は両者で**ほぼ書き換え不要**なので、後から乗り換えも比較的容易。

---

## 9. 困ったらどこを見る？

トラブル種別 → どの資料の何節を読むか の早見表。

| 症状 | 見る資料 |
| ---- | -------- |
| Gitの操作で混乱 | [git.md](../02_git/git.md) §1（全体像）/ §7（やらかし時） |
| commitしてはいけないものをcommitした | [git.md](../02_git/git.md) §8 |
| RLSが効いてデータが取れない | [supabase.md](../03_supabase/supabase.md) §5・§13 |
| ログイン状態がアプリで取れない | [supabase.md](../03_supabase/supabase.md) §6 |
| usernameをどこに保存する？ | [supabase.md](../03_supabase/supabase.md) §7 |
| Vercelでビルド失敗 | [vercel.md](../04a_vercel/vercel.md) §11 |
| 本番でバグ→急いで戻したい | [vercel.md](../04a_vercel/vercel.md) §10（ロールバック）|
| 環境変数の設定が分からない | [vercel.md](../04a_vercel/vercel.md) §7 / [git.md](../02_git/git.md) §8 |
| Azureでサインアップから始めたい | [azure/setup.md](../04b_azure/setup.md) |
| Azureサービスの選び方 | [azure/azure.md](../04b_azure/azure.md) |

---

## 10. まとめ — 全体像の覚え方

```mermaid
flowchart LR
    A["💻<br/>コードを書く<br/>(自分)"] ==> B["🔀<br/>Gitで管理<br/>(GitHub)"]
    B ==> C["⚡<br/>配信は任せる<br/>(Vercel/Azure)"]
    C ==> D["🗄️<br/>データは任せる<br/>(Supabase)"]
    D ==> E["📱<br/>ユーザーに届く"]

    style A fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C fill:#000,color:#FFFFFF,stroke:#333,stroke-width:2px
    style D fill:#3ECF8E,color:#FFFFFF,stroke:#333,stroke-width:2px
    style E fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
```

**現代のアプリ開発は「自分で全部やる」のではなく、「コード以外は外部サービスに任せる」スタイル**。各サービスの境界線を理解すれば、トラブルが起きた時にも「どこを見ればいいか」が即座に分かる。

> 📝 全部を完璧に理解しなくても、**「自分で書くのはコード本体だけ。インフラはサービスがやってくれる」**という大枠さえ掴めれば、まずはアプリを世に出せる。詳細は走りながら学べばOK。
