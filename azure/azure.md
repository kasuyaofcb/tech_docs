# Azure 全体像

> 📝 Azure は Microsoft の**総合クラウド**。「Webアプリの置き場」「DB」「認証」「AI」「ネットワーク」「仮想マシン」など、ITで使うものが**全部入り**で並んでいる巨大な道具箱。Vercel や Supabase が「特定の用途に特化した PaaS」なら、Azure は **「何でも揃うデパート」**。

🧠 先に押さえておきたい4つの言葉:

- **クラウド**: 自分のPCやサーバーではなく、**よその会社のデータセンター**を借りて動かす仕組み。Azure / AWS / Google Cloud の3社が代表。
- **PaaS / IaaS / SaaS**: クラウドの使い方の段階。**IaaS**=サーバーだけ借りる（OSは自分で） / **PaaS**=実行環境ごと借りる（コード置くだけ） / **SaaS**=完成品を使う。
- **リージョン**: データセンターの場所。「東日本リージョン」「米国東部」など世界中にある。近いほど速い。
- **テナント / サブスクリプション**: Azure特有の階層（後述§3）。最初に必ずつまずく場所。

---

## 1. Azureが提供するもの

```mermaid
flowchart TB
    APP["📱<br/><b>使う人</b><br/>開発者 / 会社"]

    subgraph AZ["☁️ Azure（クラウドのデパート）"]
        direction LR
        COMP["💻<br/><b>Compute</b><br/>アプリの置き場<br/>SWA / App Service /<br/>Functions / VM"]
        DB["🗄️<br/><b>Database</b><br/>SQL DB / Cosmos DB /<br/>PostgreSQL"]
        STG["📁<br/><b>Storage</b><br/>Blob / Files /<br/>Disks"]
        ID["🔐<br/><b>Identity</b><br/>Entra ID<br/>(旧 Azure AD)"]
        NET["🌐<br/><b>Network</b><br/>VNet / CDN /<br/>Front Door"]
        DEV["🚀<br/><b>DevOps</b><br/>GitHub Actions /<br/>Azure Pipelines"]
        AI["🤖<br/><b>AI</b><br/>Azure OpenAI /<br/>ML Studio"]
        COMP ~~~ DB ~~~ STG ~~~ ID ~~~ NET ~~~ DEV ~~~ AI
    end

    USER["📱<br/><b>エンドユーザー</b>"]

    APP ==>|"作る/運用"| AZ
    AZ ==>|"配信"| USER

    style APP fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style AZ fill:#E5F1FB,stroke:#0078D4,stroke-width:3px,color:#000
    style COMP fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style DB fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style STG fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style ID fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style NET fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style DEV fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style AI fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style USER fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
```

サービス数は **200種類以上**。最初は全部覚える必要はない。**「Webアプリを公開したい」というだけなら、出てくるのは Compute（特に Static Web Apps）+ Database + Identity の3つくらい**。

> 📝 本資料では「**個人開発から本格運用までの Web アプリ + CI/CD + DB**」という軸で、必要な部品だけを順に説明する。VMやAI周りなど他のジャンルは入口だけ触れる。

---

## 2. よくある勘違い

```mermaid
flowchart TB
    subgraph WRONG["❌ 勘違い"]
        direction LR
        W1["Azure は<br/>エンタープライズ専用"] ==> W2["個人開発には<br/>重すぎる / 高い"]
    end

    subgraph RIGHT["⭕ 実際"]
        direction LR
        R1["Static Web Apps は"] ==> R2["<b>無料枠で商用OK</b><br/>(Vercel Hobbyは商用NG)"] ==> R3["個人開発〜中規模に最適"]
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
        WA["サインインしたら<br/>すぐAzureが使える"] ==> WB["なぜか<br/>サービスが作れない"]
    end

    subgraph R2["⭕ 実際"]
        direction LR
        RA["Microsoftアカウントで<br/>サインインしただけでは<br/>「入口」だけ"] ==> RB["<b>サブスクリプション</b><br/>(=請求口座)を<br/>作って初めて使える"] ==> RC["無料試用版の<br/>サインアップ必須"]
    end

    style W2 fill:#FFE5E5,stroke:#C0392B,stroke-width:2px,color:#000
    style R2 fill:#E8F8E8,stroke:#27AE60,stroke-width:2px,color:#000
    style WA fill:#fff,color:#000,stroke:#C0392B
    style WB fill:#fff,color:#000,stroke:#C0392B
    style RA fill:#fff,color:#000,stroke:#27AE60
    style RB fill:#fff,color:#000,stroke:#27AE60
    style RC fill:#fff,color:#000,stroke:#27AE60
```

```mermaid
flowchart TB
    subgraph W3["❌ 勘違い"]
        direction LR
        WC["git push したら<br/>Azure が即本番反映"] ==> WD["怖くて使えない"]
    end

    subgraph R3["⭕ 実際"]
        direction LR
        RD["push 先の <b>ブランチ</b>で<br/>動作が違う"] ==> RE["main → 本番<br/>その他 → <b>Staging URL</b>"] ==> RF["PR毎に独立した<br/>検証環境(§6, §8)"]
    end

    style W3 fill:#FFE5E5,stroke:#C0392B,stroke-width:2px,color:#000
    style R3 fill:#E8F8E8,stroke:#27AE60,stroke-width:2px,color:#000
    style WC fill:#fff,color:#000,stroke:#C0392B
    style WD fill:#fff,color:#000,stroke:#C0392B
    style RD fill:#fff,color:#000,stroke:#27AE60
    style RE fill:#fff,color:#000,stroke:#27AE60
    style RF fill:#fff,color:#000,stroke:#27AE60
```

---

## 3. Azure特有の階層 — テナント / サブスクリプション / リソースグループ / リソース

Azureを使う上で**最初に必ずつまずく**のがこの4階層。Vercel や Supabase にはない独特の概念で、ここを理解すると一気にスッキリする。

```mermaid
flowchart TB
    T["🏢 <b>テナント</b><br/>(=組織の単位 / Entra IDで管理)<br/>例: Default Directory"]
    T ==> S["💳 <b>サブスクリプション</b><br/>(=請求口座 / 課金単位)<br/>例: Azure 無料試用版"]
    S ==> RG["📦 <b>リソースグループ</b><br/>(=フォルダ / 一括管理単位)<br/>例: rg-my-app"]
    RG ==> R1["🌐 <b>リソース</b><br/>Static Web Apps"]
    RG ==> R2["🗄️ <b>リソース</b><br/>Azure SQL DB"]
    RG ==> R3["🔑 <b>リソース</b><br/>Key Vault"]

    style T fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:3px
    style S fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:3px
    style RG fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style R1 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R3 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| 階層 | 役割 | 例えるなら |
| ---- | ---- | --------- |
| 🏢 **テナント** | ユーザー / グループの所属組織。AAD（Entra ID）が管理 | 会社そのもの |
| 💳 **サブスクリプション** | 請求のまとまり。「誰が払うか」を分ける | 経費の請求口座 |
| 📦 **リソースグループ** | 関連リソースのフォルダー。**一括で消せる**のが便利 | プロジェクト用の段ボール箱 |
| 🌐 **リソース** | 実体（Static Web Apps、DB、VMなど） | 段ボールの中身 |

> 📝 「**リソースグループごと削除**」で中身全部消せるので、お試し用に作って終わったら箱ごと捨てる、というのが定番。Free tier で課金されていなくても、**何を作ったか忘れがち**なので習慣化しておくと安全。

### 🚨 テナント切替の罠（最初の地雷）

```mermaid
flowchart LR
    L["🔐 サインイン"] ==> T1["✅ <b>Default Directory</b><br/>(自分専用テナント)"]
    L ==> T2["⚠️ <b>Microsoft Services</b><br/>その他、見覚えのない組織"]
    T2 ==>|"アクセスしようとすると"| E["🚫 'アカウントが<br/>テナントに存在しません'"]

    style L fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style T1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style T2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style E fill:#C0392B,color:#FFFFFF,stroke:#333,stroke-width:2px
```

ブラウザに古い Microsoft セッションが残っていると、勝手に**別組織のテナント**を見に行ってエラーになる。`portal.azure.com` 右上のディレクトリ表示が「Default Directory」になっていることを必ず確認する。直らない時は **別ブラウザ or シークレットウィンドウ**で `azure.microsoft.com/free` から入り直すのが確実。

---

## 4. はじめの一歩

Vercel や Supabase より一手間多い。**サインアップ → 無料試用版（=サブスクリプション作成）→ リソースグループ → 個別リソース**、の順で進む。

```mermaid
flowchart LR
    S1["① azure.microsoft.com/free<br/>「無料で始める」"] ==> S2["② Microsoftアカウントで<br/>サインイン"]
    S2 ==> S3["③ 電話 + クレカ<br/><b>本人確認</b><br/>(請求はされない)"]
    S3 ==> S4["④ サブスクリプション<br/>自動作成<br/>($200 / 30日)"]
    S4 ==> S5["⑤ portal.azure.com で<br/>リソースグループ作成"]
    S5 ==> S6["⑥ リソース<br/>(Static Web Apps等)<br/>を作成"]

    style S1 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S6 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 **クレジットカードは「本人確認」目的だけ**で、無料枠を超えなければ請求されない。デビットカード不可、JCB がたまに弾かれるので Visa/Master が無難。

### 🚨 「Pay-As-You-Go にアップグレード」を押さない

```mermaid
flowchart TB
    A["🆓 無料試用版"] ==>|"そのまま"| B["✅ 上限超過時は<br/><b>自動停止</b><br/>勝手な課金なし"]
    A ==>|"❌ ボタン押下"| C["⚠️ <b>Pay-As-You-Go</b><br/>従量課金モード<br/>上限なし"]

    style A fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C fill:#C0392B,color:#FFFFFF,stroke:#333,stroke-width:3px
```

無料試用版のままなら**枠を超えるとリソースが停止**するだけだが、Pay-As-You-Go に切り替えると**青天井で課金**される。Portal の各所に勧誘ボタンが出るが、個人開発・お試し段階では押さない。

---

## 5. アプリの置き場所 — 用途別の Compute 選び

Azureには「アプリを動かす場所」が複数ある。**個人開発→本格運用への成長段階**でこの選択が変わる。

```mermaid
flowchart TB
    Q["📱 アプリを<br/>置きたい"] ==> Q1{"どんな<br/>アプリ？"}

    Q1 -->|"フロントエンドメイン<br/>(SPA / Jamstack)"| SWA["🟢 <b>Static Web Apps</b><br/>個人〜中規模の本命<br/>無料 / GitHub連携"]
    Q1 -->|"普通のWebアプリ<br/>(SSR / API)"| APP["🔵 <b>App Service</b><br/>老舗のPaaS<br/>Slot機能が強力"]
    Q1 -->|"コンテナで<br/>動かしたい"| CA["🟣 <b>Container Apps</b><br/>Docker / KEDA<br/>マイクロサービス向き"]
    Q1 -->|"イベント駆動<br/>関数だけ"| FN["🟡 <b>Functions</b><br/>サーバーレス<br/>定期実行 / Webhook"]
    Q1 -->|"OSごと自分で<br/>使いたい"| VM["⚫ <b>Virtual Machines</b><br/>IaaS / 最重<br/>レガシー移行用"]
    Q1 -->|"本気のk8s"| AKS["🟠 <b>AKS</b><br/>Kubernetes<br/>大規模本格運用"]

    style Q fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style Q1 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style SWA fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style APP fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style CA fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style FN fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style VM fill:#34495E,color:#FFFFFF,stroke:#333,stroke-width:2px
    style AKS fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| サービス | 何向け | 料金感 | 個人開発 | 本格運用 |
| ---- | ---- | ---- | :---: | :---: |
| **Static Web Apps** | 静的サイト + API（Functions同梱） | 無料枠あり | ⭐⭐⭐ | ⭐⭐ |
| **App Service** | 一般的なWebアプリ（Node/.NET/Python） | $13/月〜 | ⭐⭐ | ⭐⭐⭐ |
| **Container Apps** | コンテナ型・マイクロサービス | 従量 | ⭐ | ⭐⭐⭐ |
| **Functions** | 単発処理・Webhook・定期実行 | 100万回/月無料 | ⭐⭐⭐ | ⭐⭐ |
| **VM** | 自由度最大・レガシー移行 | $7/月〜 | △ | ⭐⭐ |
| **AKS** | 本格的なKubernetes | 高い | × | ⭐⭐⭐ |

> 📝 まず **Static Web Apps から始め**、SSR や複雑なAPIが必要になったら **App Service** へ。本格的にコンテナ運用になったら **Container Apps** に進む、というのが定番の階段。

---

## 6. Azure Static Web Apps — Vercel に最も近い存在

本資料の中心。**「Vercelとほぼ同じ体験」を Azure で得られる**のがこのサービス。

```mermaid
flowchart TB
    DEV["💻<br/><b>開発者</b><br/>git push するだけ"]

    subgraph SWA["☁️ Static Web Apps"]
        direction LR
        BUILD["🏗️<br/><b>Build</b><br/>GitHub Actionsで<br/>自動ビルド"]
        CDN["🌐<br/><b>CDN</b><br/>世界配信"]
        SSL["🔒<br/><b>SSL</b><br/>HTTPS自動"]
        FN["⚡<br/><b>API</b><br/>Azure Functions<br/>同梱"]
        STG["🔗<br/><b>Staging</b><br/>PR毎に<br/>専用URL発行"]
        AUTH["🛂<br/><b>Auth</b><br/>GitHub/MS/X<br/>ログイン内蔵"]
        BUILD ~~~ CDN ~~~ SSL ~~~ FN ~~~ STG ~~~ AUTH
    end

    USER["📱<br/><b>エンドユーザー</b>"]

    DEV ==>|"git push"| SWA
    SWA ==>|"自動配信"| USER

    style DEV fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style SWA fill:#E5F1FB,stroke:#0078D4,stroke-width:3px,color:#000
    style BUILD fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style CDN fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style SSL fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style FN fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style STG fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style AUTH fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style USER fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
```

### Static Web Apps の作成フロー

```mermaid
flowchart LR
    S1["① portal.azure.com<br/>「Static Web Apps」検索"] ==> S2["② + 作成<br/>リソースグループ作成"]
    S2 ==> S3["③ プランは <b>Free</b><br/>リージョン: East Asia"]
    S3 ==> S4["④ GitHub認可<br/>(Only select repos推奨)"]
    S4 ==> S5["⑤ ビルドプリセット<br/>(Next.js/React/Vite)"]
    S5 ==> S6["⑥ 作成 → 2-3分<br/>URL発行"]

    style S1 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S4 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S6 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 作成時にAzureが**GitHub Actions のワークフローファイル**（`.github/workflows/azure-static-web-apps-xxxx.yml`）を**勝手にリポジトリへ commit** する。これが「git push したら自動デプロイ」の正体。Vercel が裏で持っている Webhook の代わりに、Azureは GitHub Actions に乗っかる方式。

### 💸 Static Web Apps 無料枠の主な制限

| 項目 | 上限 | 詰まったら |
| ---- | ---- | --------- |
| 帯域幅 | 100GB/月 | Standard($9/月)へ |
| ストレージ | 0.5GB | 同上 |
| カスタムドメイン | 2つまで | Standardで5つ |
| SSL証明書 | 自動・無料 | — |
| Staging環境 | PR毎に自動発行 | — |
| API実行(Functions同梱) | 無料枠内 | — |
| **商用利用** | ✅ **OK** | （Vercel Hobby より緩い） |

> 🚨 個人開発・社内ツール・小規模商用ならFree tierで十分。**Vercel Hobby と違い商用OK**なので、副業や小規模スタートアップにとっては実はAzureの方が条件が良い。

---

## 7. DB系の選び方

Static Web Apps はあくまでフロント+軽量APIなので、データ保存は別サービスが必要。Azure内で完結させるか、外部（Supabase等）と組み合わせるかが分かれ道。

```mermaid
flowchart TB
    Q["🗄️ データを<br/>保存したい"] ==> Q1{"DBの種類"}

    Q1 -->|"伝統的RDB"| AS["🔵 <b>Azure SQL<br/>Database</b><br/>SQL Server系<br/>エンプラの本命"]
    Q1 -->|"OSS Postgres"| PG["🟢 <b>Azure Database<br/>for PostgreSQL</b><br/>標準Postgres"]
    Q1 -->|"NoSQL / 多モデル"| CD["🟣 <b>Cosmos DB</b><br/>グローバル分散<br/>サーバーレス課金"]
    Q1 -->|"BaaS全部入り<br/>(認証+RT+RLS)"| SB["🟢 <b>Supabase</b><br/>(Azure外)<br/>(参考: <a href='../supabase/supabase.md'>supabase.md</a>)"]
    Q1 -->|"KV / キャッシュ"| RD["🔴 <b>Azure Cache<br/>for Redis</b><br/>セッション/キャッシュ"]

    style Q fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style Q1 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style AS fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style PG fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style CD fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style SB fill:#3ECF8E,color:#FFFFFF,stroke:#333,stroke-width:2px
    style RD fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| サービス | 強み | 個人開発 | 本格運用 |
| ---- | ---- | :---: | :---: |
| **Azure SQL Database** | エンプラ標準・MS整合性・最も無難 | △（高め） | ⭐⭐⭐ |
| **PostgreSQL (Flexible Server)** | OSS互換・移行性高 | ⭐⭐ | ⭐⭐⭐ |
| **Cosmos DB** | グローバル分散・無料枠1000RU/s | ⭐⭐ | ⭐⭐⭐ |
| **Supabase併用** | 認証+RLS+Realtime込み・手早い | ⭐⭐⭐ | ⭐⭐ |
| **Cache for Redis** | セッション/キャッシュ用の脇役 | △ | ⭐⭐ |

> 📝 個人開発の初手なら **Supabase + Azure Static Web Apps** の組み合わせが軽くて速い（実際の構成例: フロントを SWA、DB+認証は Supabase）。社内縛りで「Azure内で完結」が必要なら **Azure SQL or PostgreSQL** に切り替える。詳しいDBの概念は [supabase.md](../supabase/supabase.md) §4-5 を参照。

---

## 8. Git push = 即本番？ — 安全弁の4層

Vercelでも触れた話だが、Azure でも同じ仕組みが必要。「pushしたら即本番反映」は表面の話で、実際には**4層のゲート**が挟まる。

```mermaid
flowchart LR
    DEV["💻<br/>開発者"] ==>|"git push<br/>(featureブランチ)"| GH["☁️<br/>GitHub"]
    GH ==>|"PR作成"| G1{"🚪 ①<br/><b>CI</b><br/>テスト/Lint"}
    G1 ==>|"✅"| G2{"🚪 ②<br/><b>Staging URL</b><br/>でレビュー"}
    G2 ==>|"✅ Approve"| G3{"🚪 ③<br/><b>承認制マージ</b><br/>1+人レビュー"}
    G3 ==>|"main へ"| G4{"🚪 ④<br/><b>本番側保険</b><br/>Flag/Rollback"}
    G4 ==> PROD["🚀<br/>本番<br/>デプロイ"]

    style DEV fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style GH fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style G1 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style G2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style G3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style G4 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style PROD fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

| 層 | 仕組み | Azure 側の実現 |
| --- | --- | --- |
| ① CI（自動チェック） | テスト・型・Lintを走らせて落ちたら止める | GitHub Actions or Azure Pipelines |
| ② Staging環境 | PR毎に自動で別URLが立つ | SWA の **Staging environments** / App Service の **Deployment Slots** |
| ③ 承認制マージ | 1+人レビュー必須 | GitHub Branch protection / Azure DevOps Branch policies |
| ④ 本番側保険 | フラグ・ロールバック・カナリア | **App Configuration**（Feature Flag）/ Slot swap / Traffic routing |

> 📝 正確には「**main ブランチへのマージ＝本番**」であって、main に直接 push することは普通禁止する（Branch protection）。ブランチ運用の詳細は [git.md](../git/git.md) §4 を参照。

### 本番後でも戻せる: Rollback / Canary / Feature Flag

```mermaid
flowchart TB
    D["😱 本番でバグ発覚"] ==> Q{"対処"}
    Q -->|"全戻し"| R1["🔙 <b>Slot Swap</b><br/>前バージョンと<br/>瞬時に入れ替え"]
    Q -->|"一部だけ戻す"| R2["🎚️ <b>Traffic Routing</b><br/>新版10% / 旧版90%<br/>カナリア"]
    Q -->|"機能だけ閉じる"| R3["🚩 <b>Feature Flag</b><br/>App Configuration で<br/>ON/OFF切替"]

    style D fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Q fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style R1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R3 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 **Slot Swap** は実は Vercel より前から Azure（App Service）に存在する**老舗の仕組み**。「新バージョンを Staging スロットに置いて疎通試験 → swap ボタンで本番と入れ替え」を**ゼロダウンタイム**で実現する。エンプラ受けが良い理由のひとつ。

---

## 9. Vercel との対応表

両者の機能・概念を1対1で並べると、移行や比較の地図になる。

| Vercel | Azure 側の相当物 | 補足 |
| --- | --- | --- |
| **Preview Deployments** | **Static Web Apps Staging environments** / **App Service Deployment Slots** | PR毎に自動で別URL |
| **アトミックデプロイ** | **Slot Swap** | 新バージョンに瞬時切替 |
| **インスタントロールバック** | Slot swap で前スロットへ戻す / Pipelinesの過去ビルド再デプロイ | — |
| **CI Gate** | **GitHub Actions** or **Azure Pipelines** | SWAは前者がデフォルト |
| **環境別シークレット** | **Key Vault** + slot単位の **App Settings** | §10で詳述 |
| **Canary Deployment** | **App Service Traffic Routing**（slot間で%分配） | Vercel Pro相当 |
| **Feature Flag** | **App Configuration の Feature Manager** | — |
| **CDN** | **Front Door** or **Azure CDN**（SWAは内蔵） | — |
| **Edge Functions** | **Functions** + Front Door / **Container Apps** | — |
| **Custom Domain + 自動SSL** | SWA / App Service 標準機能 | Let's Encrypt 相当 |

```mermaid
flowchart LR
    V["⚡<br/><b>Vercel</b><br/>洗練された一体型"] ~~~ A["🔷<br/><b>Azure</b><br/>部品の組み合わせ"]
    V -.->|"Preview URL"| A1["SWA Staging<br/>App Service Slot"]
    V -.->|"Rollback"| A2["Slot Swap"]
    V -.->|"Canary"| A3["Traffic Routing"]
    V -.->|"Feature Flag"| A4["App Configuration"]
    V -.->|"Secrets"| A5["Key Vault"]

    style V fill:#000,color:#FFFFFF,stroke:#333,stroke-width:3px
    style A fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:3px
    style A1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A5 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 Vercel は **「全部1サービスに統合」**、Azure は **「部品を組み合わせて作る」**。柔軟だが、最初は「**どの部品を選べばいいの？**」で迷う。SWAから入って、必要に応じて Key Vault や App Configuration を足していくのが順当。

---

## 10. 環境変数とシークレット — 置き場所が3つある

ここがVercelより**ややこしい**ポイント。Azureでは「**いつ使うか**」で置き場所が変わる。

```mermaid
flowchart TB
    E["🔐 環境変数 / シークレット"] ==> Q{"いつ使う？"}

    Q -->|"ビルド時に<br/>埋め込みたい<br/>(例: NEXT_PUBLIC_*)"| B["📦 <b>GitHub Secrets</b><br/>Actions ワークフローで<br/>env: として渡す"]
    Q -->|"実行時に<br/>サーバー側で読む<br/>(API_KEY等)"| R["⚙️ <b>SWA Configuration</b><br/>or App Service<br/>Application Settings"]
    Q -->|"高セキュアに<br/>一元管理"| K["🔑 <b>Key Vault</b><br/>参照 (@Microsoft.KeyVault)"]

    style E fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style Q fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style B fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style K fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| 置き場 | いつ参照される | 用途例 |
| --- | --- | --- |
| **GitHub Secrets** | ビルド時 | `NEXT_PUBLIC_*`、デプロイトークン、テスト用APIキー |
| **SWA Configuration** | 実行時（サーバー側） | バックエンドAPIキー、DB接続文字列 |
| **Key Vault** | 実行時（高セキュア） | 本番DBパスワード、証明書、テナント横断のシークレット |

> 🚨 **`NEXT_PUBLIC_*` の罠**: Next.js の `NEXT_PUBLIC_*` 系は**ビルド時にコードへ埋め込まれる**。**SWA Configuration に入れても意味がない**（実行時にしか参照されないため）。必ず **GitHub Secrets** に登録し、ワークフローの `env:` で渡す。

> 📝 環境変数の基本概念は [git.md](../git/git.md) §8 / [vercel.md](../vercel/vercel.md) §7 を参照。Vercel が「Production / Preview / Development」の3環境で**自動的に分けて持てる**のに対し、Azureでは **GitHub Environments**（Actionsの環境別シークレット）や **deployment slot 別の Configuration** を**自分で組む**必要がある。ここはVercelの方が一歩楽。

### Key Vault を挟むメリット

```mermaid
flowchart LR
    APP["🟢 Static Web Apps"] ==>|"@Microsoft.KeyVault(...)"| KV["🔑 Key Vault<br/>シークレット保管庫"]
    KV ==>|"監査ログ"| LOG["📜 Azure Monitor"]
    KV ==>|"アクセス制御"| RBAC["🛂 RBAC / Managed Identity"]

    style APP fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style KV fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:3px
    style LOG fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style RBAC fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

エンプラ要件で「誰がいつシークレットを参照したか監査したい」「**人間にはキーを見せない**（Managed Identity 経由で取得）」が必要になったら Key Vault。個人開発段階では SWA Configuration で十分。

---

## 11. GitHub以外からのCI/CD — 社内縛り対策

Azure SWA は GitHub 連携が基本だが、**GitHub が使えない環境**（社内ネットワーク・コンプライアンス制約など）でも以下の選択肢がある。

```mermaid
flowchart TB
    Q["🏢 GitHubが<br/>社内で使えない"] ==> Q1{"どうする？"}

    Q1 -->|"Microsoft純正"| P1["🔷 <b>Azure DevOps<br/>Pipelines</b><br/>AAD/閉域ネットワーク◎"]
    Q1 -->|"任意のCIから"| P2["🟢 <b>SWA CLI</b><br/>deployment tokenで<br/>どこからでも push可"]
    Q1 -->|"GitLab/Bitbucket"| P3["🟠 各CIから<br/>SWA CLI経由<br/>(SWA作成は「No Source」)"]
    Q1 -->|"手動デプロイ"| P4["🟣 ローカルから<br/>swa deploy"]

    style Q fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style Q1 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style P1 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P3 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P4 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| 選択肢 | 特徴 | 社内環境で響く点 |
| --- | --- | --- |
| **Azure DevOps Pipelines** | MS純正・SWAタスク標準提供 | AAD連携◎ / セルフホストAgent / Azure Monitor統合 |
| **SWA CLI + deployment token** | 任意のCIから `swa deploy` | GitLab/Jenkins/CircleCI等 |
| **GitLab/Bitbucket** | SWA作成時は「No Source」で作り、上記CLI経由 | — |
| **手動デプロイ** | ローカルから同じCLI | お試し/緊急用 |

### 11.1 Azure DevOps の中身 — Pipelines は5部品の1つ

「Azure DevOps」は単一サービスではなく、**5つのサービスをまとめたスイート**（`dev.azure.com` 配下）。CI/CD 文脈で主役なのは Pipelines だが、社内縛り環境では **Repos も合わせて選ぶ**ことが多い（=「GitHub と GitHub Actions を Azure 内で両方代替」）。

```mermaid
flowchart TB
    AZD["🏢 <b>Azure DevOps</b><br/>(dev.azure.com)"]

    AZD --> P1["🚀 <b>Pipelines</b><br/>CI/CD<br/>(GH Actions相当)"]
    AZD --> P2["📦 <b>Repos</b><br/>Gitホスティング<br/>(GitHub相当)"]
    AZD --> P3["📋 <b>Boards</b><br/>課題管理<br/>(GH Issues相当)"]
    AZD --> P4["🗄️ <b>Artifacts</b><br/>パッケージ管理<br/>(npm/NuGet等)"]
    AZD --> P5["🧪 <b>Test Plans</b><br/>テスト計画管理<br/>(手動QA寄り)"]

    style AZD fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:3px
    style P1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P2 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P3 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P4 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P5 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| サービス | GitHub対応物 | 個人開発 | エンプラ |
| --- | --- | :---: | :---: |
| **Pipelines** | GitHub Actions | ⭐⭐ | ⭐⭐⭐ |
| **Repos** | GitHub本体 | △ | ⭐⭐⭐ |
| **Boards** | GitHub Issues / Projects | △ | ⭐⭐⭐ |
| **Artifacts** | GitHub Packages | △ | ⭐⭐ |
| **Test Plans** | （独自・GitHubにない） | × | ⭐⭐ |

> 📝 「GitHubを丸ごとAzure側で代替」したいなら **Repos + Pipelines** をセットで使う。両方とも **Entra ID (AAD)** で社員アカウント連携でき、Microsoft 365 と権限管理を一元化できる。

### 11.2 GitHub Actions ⇄ Azure Pipelines マッピング

YAML の作りはそっくりで、概念単位で**素直に翻訳できる**。既存の GH Actions ワークフローを書き換える際のチートシート。

```mermaid
flowchart LR
    subgraph GHA["🟪 GitHub Actions"]
        direction TB
        G1["📄 .github/workflows/<br/>*.yml"]
        G2["on: push"]
        G3["jobs:"]
        G4["runs-on: ubuntu-latest"]
        G5["steps:<br/>- uses: actions/..."]
        G6["secrets.MY_KEY"]
        G1 --> G2 --> G3 --> G4 --> G5 --> G6
    end

    subgraph ADP["🟦 Azure Pipelines"]
        direction TB
        A1["📄 azure-pipelines.yml"]
        A2["trigger: / pr:"]
        A3["stages: / jobs:"]
        A4["pool: vmImage:<br/>'ubuntu-latest'"]
        A5["steps:<br/>- task: ...@1"]
        A6["$(MY_KEY)<br/>(Variable group)"]
        A1 --> A2 --> A3 --> A4 --> A5 --> A6
    end

    GHA -.->|"翻訳"| ADP

    style GHA fill:#F0F0F0,stroke:#8E44AD,stroke-width:2px,color:#000
    style ADP fill:#E5F1FB,stroke:#0078D4,stroke-width:2px,color:#000
    style G1 fill:#fff,color:#000,stroke:#8E44AD
    style G2 fill:#fff,color:#000,stroke:#8E44AD
    style G3 fill:#fff,color:#000,stroke:#8E44AD
    style G4 fill:#fff,color:#000,stroke:#8E44AD
    style G5 fill:#fff,color:#000,stroke:#8E44AD
    style G6 fill:#fff,color:#000,stroke:#8E44AD
    style A1 fill:#fff,color:#000,stroke:#0078D4
    style A2 fill:#fff,color:#000,stroke:#0078D4
    style A3 fill:#fff,color:#000,stroke:#0078D4
    style A4 fill:#fff,color:#000,stroke:#0078D4
    style A5 fill:#fff,color:#000,stroke:#0078D4
    style A6 fill:#fff,color:#000,stroke:#0078D4
```

| GitHub Actions | Azure Pipelines | 役割 |
| --- | --- | --- |
| `.github/workflows/*.yml` | `azure-pipelines.yml` | パイプライン定義ファイル |
| `on:` | `trigger:` / `pr:` | トリガー（push/PR） |
| `jobs:` | `jobs:` / `stages:` | 並列実行単位 |
| `runs-on:` | `pool:` | 実行マシン指定 |
| `steps:` + `uses:` | `steps:` + `task:` | 再利用部品 |
| `secrets.*` | `$(name)` + Variable Group | シークレット参照 |
| `env:` | `variables:` | 環境変数 |
| `environment:` | `environment:` + Approval | 環境ゲート・承認 |

> 📝 SWA 専用タスク **`AzureStaticWebApp@0`** が Microsoft 公式に用意されているので、GH Actions の `Azure/static-web-apps-deploy@v1` と1対1で置き換えられる。NEXT_PUBLIC_* 系のビルド時埋め込み（§10）も `env:` で渡せばよく、考え方は同じ。

### 11.3 エンプラで響く3つの理由 — 閉域 / AAD / 監査

社内でGitHubが使いにくい状況の本質は「**外部にコード/資格情報を出したくない**」「**社員アカウントで一元管理したい**」「**監査ログがほしい**」の3点。Azure DevOps Pipelines はここに**標準対応**する。

```mermaid
flowchart TB
    R["🏢 エンプラ要件"] --> R1{"必要なもの"}

    R1 --> N1["🛡️ <b>閉域ネットワーク</b><br/>ビルドマシンを<br/>社内に置きたい"]
    R1 --> N2["🛂 <b>SSO / ID統合</b><br/>社員アカウントで<br/>そのまま使う"]
    R1 --> N3["📜 <b>監査</b><br/>誰がいつ何をしたか<br/>記録/出力"]

    N1 --> S1["✅ <b>Self-hosted Agent</b><br/>社内サーバー/VMに<br/>エージェント常駐"]
    N2 --> S2["✅ <b>Entra ID (AAD)</b><br/>連携<br/>Service Principal<br/>でAzure操作"]
    N3 --> S3["✅ <b>Azure Monitor</b><br/>連携<br/>監査ログ集約"]

    style R fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style R1 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style N1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style N2 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style N3 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

#### Self-hosted Agent — 閉域ネットワークの肝

```mermaid
flowchart LR
    subgraph CLOUD["☁️ Microsoft-hosted (デフォルト)"]
        MH["🟦 Microsoft<br/>データセンター<br/>のVM"]
    end

    subgraph INTRA["🏢 社内ネットワーク (閉域)"]
        SH["🟩 <b>Self-hosted</b><br/><b>Agent</b><br/>社内サーバー"]
        SC["🗄️ 社内Git/<br/>社内DB/<br/>社内API"]
        SH ---|"閉域内で完結"| SC
    end

    P["🟦 Azure Pipelines<br/>(クラウド側のオーケストレータ)"] -.->|"ジョブ指示"| MH
    P -.->|"ジョブ指示 (アウトバウンドのみ)"| SH

    style CLOUD fill:#E5F1FB,stroke:#0078D4,stroke-width:2px,color:#000
    style INTRA fill:#E8F8E8,stroke:#27AE60,stroke-width:2px,color:#000
    style P fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:3px
    style MH fill:#fff,color:#000,stroke:#0078D4
    style SH fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style SC fill:#fff,color:#000,stroke:#27AE60
```

| 項目 | Microsoft-hosted | Self-hosted |
| --- | --- | --- |
| 動く場所 | Microsoft のクラウド | 社内サーバー / VM |
| インターネット必須 | ✅ 必要 | ❌ アウトバウンドのみで可 |
| 無料枠 | 1,800分/月（Private） | 自前マシン |
| 社内DB/社内APIアクセス | ❌ 不可（外から見えない） | ✅ 可 |
| 構築の手間 | ゼロ | エージェントインストール必要 |

> 📝 Agent は**アウトバウンドの長時間ポーリング**で Pipelines クラウドから「次のジョブ」を取りに行く方式。**社内ファイアウォール越しでも疎通可能**で、社内側でポートを開ける必要がない。これが「閉域でも回せる」最大の理由。

#### AAD連携 + Service Principal — 人ではなくロボットIDで認証

```mermaid
flowchart LR
    DEV["👤 社員<br/>(Entra ID)"] ==>|"SSO"| ADO["🟦 Azure DevOps"]
    ADO ==>|"Service Connection"| SP["🤖 <b>Service Principal</b><br/>(ロボット ID)"]
    SP ==>|"RBAC で必要権限のみ"| AR["🔷 Azureリソース<br/>(SWA / Key Vault等)"]

    style DEV fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style ADO fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style SP fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style AR fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 **Service Principal** = Pipelines が Azure リソースを操作するための**専用ロボットアカウント**。「人間のパスワードをパイプラインに埋め込む」のではなく、ロボットIDに最小権限だけ与える。退職者対応・権限剥奪が AAD で一元化できるのがエンプラで効く。

### 11.4 エンプラ移行の現実的なレシピ

```mermaid
flowchart LR
    Step1["① GitHub Actions で<br/>動く構成を作る<br/>(個人開発)"] ==> Step2["② Azure Repos に<br/>コードを移送"]
    Step2 ==> Step3["③ workflow.yml を<br/>azure-pipelines.yml に翻訳<br/>(§11.2 マッピング)"]
    Step3 ==> Step4["④ Self-hosted Agent<br/>を社内に立てる"]
    Step4 ==> Step5["⑤ Service Connection<br/>(Service Principal)<br/>でAzure接続"]
    Step5 ==> Step6["✅ 閉域 + AAD +<br/>監査 全部満たす"]

    style Step1 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Step2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Step3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style Step4 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Step5 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Step6 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

> 📝 個人開発段階で GitHub Actions に慣れておけば、エンプラ段階でも **「同じYAMLを翻訳するだけ」** に持ち込める。最初から Pipelines で組む必要はなく、**①〜⑤を順番に置き換える**のが現実的。

---

## 12. 無料枠と課金体系

Azure はサービスごとに無料枠の有無・形式がバラバラ。「**Always Free（常時無料）**」と「**12ヶ月無料**」と「**$200クレジット**」が混在している。

```mermaid
flowchart TB
    F["🆓 Azure Free"] ==> F1["🟢 <b>Always Free</b><br/>ずっと無料<br/>(枠内なら永続)"]
    F ==> F2["🟡 <b>12ヶ月無料</b><br/>サービス毎に<br/>無料期間あり"]
    F ==> F3["🟣 <b>$200クレジット</b><br/>30日有効<br/>有料サービス試用用"]

    F1 ==> A1["Static Web Apps"]
    F1 ==> A2["Functions (100万実行/月)"]
    F1 ==> A3["Cosmos DB (1000RU/s)"]
    F2 ==> B1["App Service (B1)"]
    F2 ==> B2["VM (B1S 750h/月)"]
    F3 ==> C1["VM / SQL DB等<br/>のお試し"]

    style F fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:3px
    style F1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style F3 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A1 fill:#fff,color:#000,stroke:#27AE60
    style A2 fill:#fff,color:#000,stroke:#27AE60
    style A3 fill:#fff,color:#000,stroke:#27AE60
    style B1 fill:#fff,color:#000,stroke:#F4C430
    style B2 fill:#fff,color:#000,stroke:#F4C430
    style C1 fill:#fff,color:#000,stroke:#8E44AD
```

| 区分 | サービス例 | 注意点 |
| --- | --- | --- |
| **Always Free** | Static Web Apps、Functions、Cosmos DB、Entra ID無料層 | 個人開発の基盤になる |
| **12ヶ月無料** | App Service B1、VM B1S、SQL DB 250GB等 | 期限後は課金開始 |
| **$200クレジット** | 全サービスに使える試用枠 | 30日で消滅 |

### 💰 GitHub Actions の無料枠（Privateリポ）

| 内容 | 無料枠 |
| --- | --- |
| Public リポ | **無制限** |
| Private リポ | **2,000分/月** |
| SWA の1ビルド | 30秒〜2分 |

> 📝 個人開発で月1,000ビルドはまず使い切らない。チームで毎日大量に PR を回す段階で初めて意識する。

### 🚨 課金の見張り方

```mermaid
flowchart LR
    P["💰 課金が怖い"] ==> B["📊 <b>コスト管理 + 課金</b><br/>(Cost Management)"]
    B ==> B1["📈 予測コスト確認"]
    B ==> B2["🔔 <b>予算アラート</b>設定<br/>(例: $5超で通知)"]
    B ==> B3["🔍 リソース別内訳"]

    style P fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style B1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

最初に「**予算アラート**」を $1 や $5 で仕掛けておくと、知らないうちに課金が走るのを防げる。

---

## 13. 認証 — 2つの「private」を整理する

「private にしたい」と言ったとき、**意味が2つ**ある。混同しがちなので最初に切り分ける。

```mermaid
flowchart LR
    Q["🤔 「private にしたい」"] ==> A{"どっちの<br/>private？"}
    A -->|"ソースコードを隠す"| P1["📦 <b>リポジトリを Private</b><br/>GitHub側の設定<br/>✅ Free tierでOK"]
    A -->|"サイト自体に<br/>ログインを要求"| P2["🔐 <b>サイトに認証</b><br/>Azure側の機能<br/>(SWA組込 or Entra ID)"]

    style Q fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style A fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style P1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P2 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

### 13.1 リポジトリを Private にする

```mermaid
flowchart LR
    G["GitHub<br/>リポジトリ作成"] ==>|"Private 選択"| R["📦 Private リポ"]
    R ==>|"Azure 連携時"| AC["⚙️ Azure 認可で<br/>「Only select repositories」<br/>を選ぶ"]
    AC ==> S["✅ そのリポだけ<br/>Azureに見える"]

    style G fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style AC fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

Privateリポでも Free tier の機能はすべて変わらず使える。**Azureに渡す権限の範囲を最小化**するのがコツ。

### 13.2 サイト自体にログインをかける

```mermaid
flowchart TB
    Q["🔐 サイトを<br/>ログイン必須にしたい"] ==> A{"どの認証？"}
    A -->|"無料 / 簡単"| B1["✅ <b>SWA組込み認証</b><br/>GitHub / MS / X<br/>(staticwebapp.config.json)"]
    A -->|"会社のAD連携"| B2["🏢 <b>Entra ID</b><br/>(旧 Azure AD)<br/>シングルサインオン"]
    A -->|"独自B2C"| B3["💰 <b>Entra External ID</b><br/>(旧 Azure AD B2C)<br/>カスタムドメイン認証"]

    style Q fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:3px
    style A fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style B1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B2 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B3 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 Free tier でも **`staticwebapp.config.json`** を置けば「全ページ Microsoftアカウントログイン必須」のような認証は無料でかかる。「会社アカウントで社内向けポータル」レベルなら **Entra ID** との連携、「外部ユーザー向けの SaaS 認証」が必要になったら **Entra External ID** に進む。

---

## 14. 個人開発 → 本格運用への階段

「最初は Free tier、でもいずれ本番に育てたい」という流れで、**増やしていくサービス**と**置き換えていく構成**を一望する。

```mermaid
flowchart LR
    L1["🌱 <b>Lv.1 個人開発</b><br/>(無料)"] ==> L2["🌿 <b>Lv.2 小規模本番</b><br/>($10-30/月)"]
    L2 ==> L3["🌳 <b>Lv.3 本格運用</b><br/>($100+/月)"]
    L3 ==> L4["🌲 <b>Lv.4 エンプラ</b><br/>(従量)"]

    style L1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
    style L2 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:3px
    style L3 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:3px
    style L4 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:3px
```

| Lv | フロント | API | DB | 認証 | シークレット | CI/CD | 監視 |
| -- | -- | -- | -- | -- | -- | -- | -- |
| **Lv.1 個人** | SWA Free | SWA同梱Functions | Supabase Free or Cosmos DB Free | SWA組込 | SWA Configuration | GitHub Actions | Portalで目視 |
| **Lv.2 小規模本番** | SWA Standard | 同上 or 単独Functions | Azure SQL Basic / PostgreSQL Burstable | Entra ID Free | Key Vault | + Branch protection | Application Insights |
| **Lv.3 本格運用** | SWA Standard / App Service | App Service or Container Apps | Azure SQL / Cosmos / PostgreSQL（HA構成） | Entra ID P1 | Key Vault + Managed Identity | + Staging slot + 承認ゲート | App Insights + Azure Monitor |
| **Lv.4 エンプラ** | Front Door + App Service / AKS | AKS or Container Apps | 上記 + Read Replica + Geo-Redundant | Entra ID P2 + Conditional Access | Key Vault + Private Endpoint | Azure Pipelines + Bicep/Terraform | Monitor + Sentinel |

```mermaid
flowchart TB
    subgraph SCALE["📈 育てる方向の典型パス"]
        direction TB
        S1["① <b>SWA Free + Supabase</b><br/>最速で公開"]
        S2["② <b>SWA Standard + Azure SQL</b><br/>独自ドメイン・SLA重視"]
        S3["③ <b>App Service + Slot Swap</b><br/>SSR/重いAPI対応"]
        S4["④ <b>Container Apps / AKS</b><br/>マイクロサービス化"]
        S1 ==> S2 ==> S3 ==> S4
    end

    style SCALE fill:#F0F0F0,stroke:#0078D4,stroke-width:2px,color:#000
    style S1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S4 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 **Lv.1 → Lv.2 の段差が一番大きい**（Supabaseから Azure SQL への移行、シークレット管理を Key Vault に集約、独自ドメイン取得 等）。Lv.2 を超えると Lv.3/4 は「同じ仕組みを規模拡大していくだけ」になるため、Lv.2 を1度経験しておくのが目安。

### 各レベルで足す/置き換える代表的なもの

```mermaid
flowchart LR
    L1A["🌱 Lv.1"] -->|"独自ドメイン + SLA"| L2A["🌿 Lv.2"]
    L2A -->|"SSR / 重い処理"| L3A["🌳 Lv.3"]
    L3A -->|"複数サービス連携"| L4A["🌲 Lv.4"]

    L1A -.->|"足すもの"| F1["💳 課金プラン化<br/>🌐 独自ドメイン<br/>🔑 Key Vault"]
    L2A -.->|"足すもの"| F2["🟦 App Service<br/>🎚️ Deployment Slots<br/>📊 App Insights"]
    L3A -.->|"足すもの"| F3["🐳 Container Apps<br/>🚪 Front Door<br/>🚩 App Configuration"]
    L4A -.->|"足すもの"| F4["☸ AKS<br/>🛡 Private Endpoint<br/>🔍 Sentinel"]

    style L1A fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style L2A fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style L3A fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style L4A fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F1 fill:#fff,color:#000,stroke:#27AE60
    style F2 fill:#fff,color:#000,stroke:#16A085
    style F3 fill:#fff,color:#000,stroke:#0078D4
    style F4 fill:#fff,color:#000,stroke:#8E44AD
```

---

## 15. 迷ったらどこを見る？

```mermaid
flowchart LR
    Q["❓ 困った"] ==> P["<b>Portal → リソース概要</b><br/>🏠 状態とURL"]
    Q ==> A["<b>GitHub Actions タブ</b><br/>🏗️ ビルド/デプロイ失敗"]
    Q ==> L["<b>Portal → ログストリーム / Insights</b><br/>📜 実行時エラー"]
    Q ==> C["<b>Portal → 構成 (Configuration)</b><br/>🔐 環境変数の確認"]
    Q ==> B["<b>Portal → コスト管理</b><br/>💰 課金状況"]
    Q ==> M["<b>Portal → アクティビティログ</b><br/>📋 誰が何を変更したか"]

    style Q fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style P fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style L fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style M fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| 症状 | 見る場所 |
| --- | --- |
| ビルドが失敗 | **GitHub → Actions タブ**のログ |
| URLが404 | **Portal → SWA → 構成 / ルーティング設定** |
| Preview URLが出ない | PRの **Checks** タブ（コメントではなく） |
| 環境変数が反映されない | `NEXT_PUBLIC_*` ならGitHub Secrets、それ以外なら Configuration |
| 「テナントエラー」 | 右上ディレクトリを **Default Directory** へ切替 |
| 何か変 | **Portal → 概要** の状態欄 |

「デプロイは成功してるのに動かない」のほとんどは **環境変数の置き場違い** か **NEXT_PUBLIC_系のビルド時埋め込み忘れ**。

---

## 16. 補足: Vercel / Supabase との立ち位置の違い

```mermaid
flowchart TB
    subgraph VC["⚡ Vercel"]
        direction TB
        V1["フロント特化"]
        V2["1つで完結"]
        V3["DX最高 / Next最適化"]
    end

    subgraph SP["🟢 Supabase"]
        direction TB
        SP1["バックエンド特化"]
        SP2["Postgresに全部生やす"]
        SP3["OSS / 移行性高"]
    end

    subgraph AZ["🔷 Azure"]
        direction TB
        A1["総合クラウド"]
        A2["部品の組み合わせ"]
        A3["エンプラ受け◎"]
    end

    style VC fill:#F0F0F0,stroke:#000,stroke-width:2px,color:#000
    style SP fill:#E8F8E8,stroke:#3ECF8E,stroke-width:2px,color:#000
    style AZ fill:#E5F1FB,stroke:#0078D4,stroke-width:2px,color:#000
    style V1 fill:#fff,color:#000,stroke:#000
    style V2 fill:#fff,color:#000,stroke:#000
    style V3 fill:#fff,color:#000,stroke:#000
    style SP1 fill:#fff,color:#000,stroke:#3ECF8E
    style SP2 fill:#fff,color:#000,stroke:#3ECF8E
    style SP3 fill:#fff,color:#000,stroke:#3ECF8E
    style A1 fill:#fff,color:#000,stroke:#0078D4
    style A2 fill:#fff,color:#000,stroke:#0078D4
    style A3 fill:#fff,color:#000,stroke:#0078D4
```

**結論: 3者は競合ではなく層が違う。**

- **Vercel**: フロントエンドのDXが最強。Next.js を作っている会社が運営。「すぐ公開・運用も楽」が最大の価値
- **Supabase**: バックエンド（DB+認証+ファイル）を1個にまとめた BaaS。Postgresが軸（[supabase.md](../supabase/supabase.md) §14参照）
- **Azure**: クラウドの**総合デパート**。何でも揃うが組み合わせ力が要る。**エンプラ・社内・規模拡大**に強い

実務的な使い分け:

```mermaid
flowchart TB
    Q["🎯 何を作る？"] ==> Q1{"規模 / 制約"}
    Q1 -->|"個人 / 趣味"| C1["⚡ Vercel + Supabase"]
    Q1 -->|"社内ツール<br/>小規模商用"| C2["🔷 SWA + Supabase or Azure SQL"]
    Q1 -->|"エンプラ縛り<br/>(社内 / 監査)"| C3["🔷 SWA/App Service +<br/>Azure SQL + Key Vault +<br/>Entra ID + Pipelines"]

    style Q fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style Q1 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style C1 fill:#000,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C2 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C3 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 「**クラウドのデパート1個に、Compute / DB / Auth / DevOps を全部並べたもの**」というのが Azure のしっくりくる説明。Supabase が「Postgres全部入り」、Vercel が「CDN全部入り」だったのに対し、Azure は「全部入りの全部入り」。だからこそ**入り口の Static Web Apps から始めて、必要になったら隣の棚を覗く**、という育て方が向いている（[vercel.md](../vercel/vercel.md) §13 / [supabase.md](../supabase/supabase.md) §14 と対になる構造）。
