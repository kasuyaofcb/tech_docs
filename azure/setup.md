# Azure 環境構築手順 — アカウント開設からアプリ公開まで

> 📝 [azure.md](azure.md) が**概念の地図**なら、このドキュメントは**実際に手を動かす順番**。**Azure アカウント開設 → リソース準備 → CI/CD → DB → デプロイ確認**までを通しで案内する。CI/CD と DB は途中で分岐するので、自分の状況に合わせて選ぶ。

🧠 想定する到達点:

- ブラウザから `https://xxxxx.azurestaticapps.net` にアクセスしてアプリが見える
- GitHub（or Azure Repos）のmainブランチに push → 数分後に本番反映される
- featureブランチで PR を作る → Preview URL が立つ
- DB（Supabase or Azure内DB）から読み書きできる

---

## 0. 全体マップ

```mermaid
flowchart TB
    P1["🟢 <b>Part 1</b><br/>共通ベース<br/>(必須)"]

    P1 ==> S1["①<br/>Azureアカウント<br/>開設"]
    S1 ==> S2["②<br/>リソースグループ<br/>作成"]
    S2 ==> S3["③<br/>アプリのGit準備"]

    S3 ==> P2["🟡 <b>Part 2</b><br/>CI/CD構築<br/>(2択)"]
    P2 --> A["🅰️ GitHub Actions版<br/>(個人〜小規模)"]
    P2 --> B["🅱️ Azure Pipelines版<br/>(社内縛り)"]

    A ==> P3["🟣 <b>Part 3</b><br/>DB選択<br/>(3択)"]
    B ==> P3
    P3 --> X["Ⓧ Supabase併用<br/>(最速)"]
    P3 --> Y["Ⓨ Azure SQL Database<br/>(エンプラ)"]
    P3 --> Z["Ⓩ Azure PostgreSQL<br/>(OSS互換)"]

    X ==> P4["🟢 <b>Part 4</b><br/>つなぎ込み + デプロイ確認"]
    Y ==> P4
    Z ==> P4

    P4 ==> END["🎉 公開完了"]

    style P1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
    style P2 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style P3 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:3px
    style P4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
    style A fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style X fill:#3ECF8E,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Y fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Z fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style END fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
```

| Part | 内容 | 所要時間 |
| --- | --- | --- |
| 1 | 共通ベース (アカウント・リソースグループ・リポジトリ) | 30〜45分 |
| 2 | CI/CD構築 (GH Actions版 or Pipelines版) | 15〜30分 |
| 3 | DB準備 (Supabase / Azure SQL / PostgreSQL) | 15〜30分 |
| 4 | つなぎ込み + デプロイ確認 | 15分 |

> 📝 **おすすめの初手**: 「Part 1 → 2-A (GH Actions) → 3-X (Supabase) → Part 4」が最短で公開まで行ける。エンプラ要件が出てきたら 2-B / 3-Y に置き換える。

---

# Part 1: 共通ベース

CI/CDやDBの選択肢に関わらず、必ず通る土台。

## 1.1 Azureアカウント開設

```mermaid
flowchart LR
    S1["①<br/>シークレットウィンドウ<br/>を開く"] ==> S2["②<br/>azure.microsoft.com/free<br/>「無料で始める」"]
    S2 ==> S3["③<br/>Microsoftアカウント<br/>でサインイン"]
    S3 ==> S4["④<br/>📞 電話認証<br/>+ 💳 クレカ認証"]
    S4 ==> S5["⑤<br/>サブスクリプション<br/>自動作成"]
    S5 ==> S6["⑥<br/>Portal にログイン<br/>確認"]

    style S1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S4 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S5 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S6 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

**操作の要点:**

| ステップ | やること | 注意 |
| --- | --- | --- |
| ① | シークレット/プライベートウィンドウを開く | 既存Microsoftセッションの干渉を防ぐ |
| ② | `azure.microsoft.com/free` で「無料で始める」 | `portal.azure.com` から行くと詰まりやすい |
| ③ | Microsoftアカウントでサインイン | 個人用アカウントを選ぶ（職場用ではない）|
| ④ | 電話 (SMS or 音声) + クレカで本人確認 | クレカは確認用、課金されない |
| ⑤ | サブスクリプションが自動で作られる | `$200 / 30日` のクレジット付与 |
| ⑥ | Portalで右上ディレクトリ表示を確認 | **Default Directory** になっていればOK |

> 🚨 **テナント切替の罠**: ⑥で「Microsoft Services」などになっていたら、別Microsoftアカウントのセッションが残っている。新しいシークレットウィンドウで②からやり直す。詳しくは [azure.md §3](azure.md) の「テナント切替の罠」参照。

> 🚨 **「Pay-As-You-Goにアップグレード」を押さない**: Portalの随所に出るが、押すと従量課金モードに切替わる。無料試用版のまま使うこと（上限超過時は自動停止）。

## 1.2 リソースグループ作成

```mermaid
flowchart LR
    S1["①<br/>Portal上部の<br/>検索バー"] ==>|"「リソースグループ」"| S2["②<br/>+ 作成"]
    S2 ==> S3["③<br/>サブスクリプション<br/>リージョン<br/>名前 を入力"]
    S3 ==> S4["④<br/>確認と作成 → 作成"]
    S4 ==> S5["⑤<br/>'デプロイが完了しました'"]

    style S1 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

**入力内容:**

| 項目 | 入れる値 |
| --- | --- |
| サブスクリプション | Azure サブスクリプション 1 (自動表示) |
| リソースグループ名 | `rg-my-app` など（自由・英数字とハイフン） |
| リージョン | East Asia (東アジア) または Japan East |

> 📝 リソースグループは**フォルダ**の役割。SWA / DB / Key Vault など、これから作るリソースはすべてこのグループに入れる。**お試しが終わったら「リソースグループ削除」で中身ごと一掃**できる（[azure.md §3](azure.md) 参照）。

## 1.3 アプリの Git 準備

CI/CD の入力になるアプリのコードを Git に置く。

```mermaid
flowchart LR
    S1["①<br/>GitHubで<br/>新規リポジトリ"] ==> S2["②<br/>Private 選択<br/>(推奨)"]
    S2 ==> S3["③<br/>ローカルで<br/>アプリ準備"]
    S3 ==> S4["④<br/>git init / add / commit<br/>初期コミット"]
    S4 ==> S5["⑤<br/>git push -u origin main<br/>リモートに上げる"]

    style S1 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S4 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S5 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

**アプリの選択肢（小さく始める）:**

| 種類 | 中身 | おすすめ |
| --- | --- | --- |
| 静的HTML | `index.html` 1枚 | いちばん躓きにくい |
| Vite + React | `npm create vite@latest` | 5分で雛形が立つ |
| Next.js | `npx create-next-app@latest` | 本格運用に近い |

> 🚨 **`.gitignore` を忘れない**: `.env*` `node_modules/` `.next/` `dist/` などを除外。[git.md §8](../git/git.md) 参照。Supabaseキーなどの秘密情報がリポジトリに乗らないようにする。

> 📝 リポジトリは**Private**で問題なし。Azure SWA は Private リポにも対応していて Free tier の制限も変わらない。詳しくは [azure.md §13](azure.md) 参照。

---

# Part 2: CI/CD構築

GitHub Actions版 (2-A) と Azure Pipelines版 (2-B) のどちらかを選ぶ。

```mermaid
flowchart TB
    Q["🤔 どっち？"] ==> Q1{"環境の制約"}

    Q1 -->|"GitHub使える<br/>(個人/小規模)"| A["🅰️ <b>GitHub Actions版</b><br/>(2-A)<br/>所要15分"]
    Q1 -->|"社内縛り<br/>GitHub使えない"| B["🅱️ <b>Azure Pipelines版</b><br/>(2-B)<br/>所要30分"]

    style Q fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style Q1 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style A fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
```

---

## 2-A. GitHub Actions版 (個人開発〜小規模)

Azure SWA を作る際に GitHub 連携を指定すれば、Azure が**勝手にワークフローファイルを commit** してくれる。実質「Portal で数項目埋めるだけ」。

```mermaid
flowchart LR
    S1["①<br/>Portalで<br/>'Static Web Apps'検索"] ==> S2["②<br/>+ 作成"]
    S2 ==> S3["③<br/>基本タブ<br/>名前/プラン/リージョン"]
    S3 ==> S4["④<br/>GitHubでサインイン<br/>+ リポジトリ認可"]
    S4 ==> S5["⑤<br/>ビルドプリセット選択<br/>(Next.js / Vite / 静的)"]
    S5 ==> S6["⑥<br/>確認と作成 → 作成<br/>(2〜3分待つ)"]
    S6 ==> S7["⑦<br/>GitHub Actions が<br/>自動起動・初回デプロイ"]
    S7 ==> S8["✅<br/>URL発行<br/>xxxxx.azurestaticapps.net"]

    style S1 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S6 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S7 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S8 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

**基本タブの入力内容:**

| 項目 | 入れる値 |
| --- | --- |
| サブスクリプション | Azure サブスクリプション 1 (自動) |
| リソースグループ | `rg-my-app` (Part 1で作ったもの) |
| 名前 | `my-app-swa` など |
| プランの種類 | **Free** ⚠️ Standardを選ばない |
| Azure Functions のリージョン | East Asia |

**GitHub 認可の選び方:**

- 「Authorize Azure-App-Service-Static-Web-Apps」をクリック
- 権限スコープは **「Only select repositories」** を選び、対象リポジトリだけチェック
- 戻ってきたら、組織 / リポジトリ / ブランチ を選ぶ（ブランチは `main`）

**ビルドの詳細:**

| 項目 | 入れる値 |
| --- | --- |
| ビルドのプリセット | `Next.js` / `React` / `Vite` / `Custom` から選ぶ |
| アプリの場所 | `/` (ルート) |
| API の場所 | 空欄 |
| 出力先 | プリセットが自動判定（HTML1枚なら空欄でOK） |

> 🚨 **プリセット誤検出に注意**: Next.js を使っているのに「React」と誤検出されることがある。必ず**ドロップダウンを開いて正しいフレームワークを選ぶ**。React のままだと `build` フォルダを探しに行ってデプロイ失敗する。

**作成後に起きること:**

```mermaid
flowchart LR
    A["✅<br/>Azure側で<br/>リソース作成完了"] ==> B["📄<br/>Azureが<br/>workflow.ymlを<br/>リポジトリにcommit"]
    B ==> C["⚡<br/>GitHub Actions<br/>初回ビルド起動"]
    C ==> D["🚀<br/>初回デプロイ<br/>完了 → URL有効化"]

    style A fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style D fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
```

完了後にローカルで `git pull` すると、`.github/workflows/azure-static-web-apps-xxxx.yml` が降ってくる。これが今後の自動ビルド設定。

> 📝 環境変数を渡す必要がある場合（NEXT_PUBLIC_* 等）は、Part 4 でこの YAML を編集する。

---

## 2-B. Azure Pipelines版 (社内縛り向け)

GitHub が使えない / Azure 内で完結させたい場合の手順。**Azure DevOps 組織の作成 → Azure Repos にコード移送 → Pipelines 構築**の3段階。

```mermaid
flowchart LR
    S1["①<br/>dev.azure.com<br/>Organization作成"] ==> S2["②<br/>Project作成<br/>(Reposも自動付帯)"]
    S2 ==> S3["③<br/>ローカルリポを<br/>Azure Reposへpush"]
    S3 ==> S4["④<br/>Portalで<br/>SWA作成<br/>'デプロイのソース: その他'"]
    S4 ==> S5["⑤<br/>deployment token<br/>取得"]
    S5 ==> S6["⑥<br/>azure-pipelines.yml<br/>作成"]
    S6 ==> S7["⑦<br/>Service Connection<br/>+ Variable Group登録"]
    S7 ==> S8["✅<br/>Pipelines実行<br/>→ デプロイ完了"]

    style S1 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S6 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S7 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S8 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

### Step ① Organization と Project 作成

- `dev.azure.com` にアクセス（Portalと同じMicrosoftアカウントでサインイン）
- 初回は「Create new organization」→ 組織名（例: `my-company`）と地域を選ぶ
- 続けて「New Project」→ プロジェクト名（例: `my-app`）→ Visibility: Private → Create

### Step ② Azure Repos にコード移送

| 状況 | 操作 |
| --- | --- |
| まだ何もリポジトリがない | プロジェクト → Repos → ローカルから `git remote add origin ...` で push |
| GitHub からコピーしたい | プロジェクト → Repos → Import → GitHub URL を貼る |

> 📝 Azure Repos の URL は `https://dev.azure.com/{org}/{project}/_git/{repo}` の形。Personal Access Token (PAT) を発行して認証する。

### Step ③ SWA を「ソースなし」で作る

```mermaid
flowchart LR
    A["Portalで<br/>SWA作成画面"] ==>|"基本タブ"| B["プラン: Free<br/>リージョン: East Asia"]
    B ==>|"デプロイの詳細"| C["⚠️ ソースを<br/>'<b>その他</b>' に"]
    C ==> D["確認と作成 → 作成<br/>(GitHubAS連携なし)"]

    style A fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style C fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style D fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

ここで GitHub と紐づけず、**自前の Pipelines からデプロイ**する状態にする。

### Step ④ Deployment Token を取得

- Portal → 作った SWA → 概要 → 右上「**デプロイトークンの管理**」
- 表示されたトークンをコピー（一度しか出ない）

### Step ⑤ azure-pipelines.yml を作る

リポジトリのルートに `azure-pipelines.yml` を作成。最低限の構造:

| 要素 | 役割 |
| --- | --- |
| `trigger:` | どのブランチへの push でビルドするか |
| `pool: vmImage:` | Microsoft-hosted Agent 指定（例: `ubuntu-latest`）|
| `steps:` | npm install → build → デプロイの順 |
| `task: AzureStaticWebApp@0` | Microsoft 公式の SWA デプロイタスク |

> 📝 Microsoft Learn の「Azure Pipelines を使用してデプロイする」ページに公式テンプレートあり。GitHub Actions の `azure-static-web-apps-xxxx.yml` と構造は1対1対応（[azure.md §11.2](azure.md) 参照）。

### Step ⑥ Variable Group に Deployment Token を登録

- Azure DevOps → Pipelines → Library → + Variable group
- 名前: `swa-secrets` など
- 変数追加: `AZURE_STATIC_WEB_APPS_API_TOKEN` = 先ほどのトークン
- 🔒 マークをONにして秘匿化

### Step ⑦ Pipeline 作成と初回実行

- Pipelines → Create Pipeline → Azure Repos Git → 対象リポジトリ
- 既存 YAML を使う → ブランチ `main`、パス `/azure-pipelines.yml`
- Variable group を紐付け
- Run pipeline → ビルド成功 → SWA に反映

> 📝 **社内ネットワーク縛り**で Microsoft-hosted Agent が使えない場合は、Step ⑦ の `pool:` を `pool: name: <自前エージェントプール名>` に変えて、別途 Self-hosted Agent を社内VMにインストールする。Agent はアウトバウンドのみで Pipelines と通信するのでファイアウォールを開ける必要なし（[azure.md §11.3](azure.md) 参照）。

---

# Part 3: DB選択

```mermaid
flowchart TB
    Q["🤔 どのDB？"] ==> Q1{"優先したいもの"}

    Q1 -->|"最速で公開<br/>認証+RLS込み"| X["Ⓧ <b>Supabase併用</b><br/>(3-X)<br/>所要15分"]
    Q1 -->|"エンプラ整合性<br/>MS純正"| Y["Ⓨ <b>Azure SQL Database</b><br/>(3-Y)<br/>所要20分"]
    Q1 -->|"OSS Postgres<br/>移行性重視"| Z["Ⓩ <b>Azure PostgreSQL</b><br/>(3-Z)<br/>所要20分"]

    style Q fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style Q1 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style X fill:#3ECF8E,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Y fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Z fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

---

## 3-X. Supabase 併用 (最速ルート)

DB + 認証 + Realtime + RLS が一式揃った BaaS。Azure 側に DB を作らずに済む。**個人開発で最も速い**。

```mermaid
flowchart LR
    S1["①<br/>supabase.com<br/>サインアップ"] ==> S2["②<br/>New project<br/>リージョン/DBパスワード"]
    S2 ==> S3["③<br/>Settings → API<br/>URL + anon key コピー"]
    S3 ==> S4["④<br/>SDKをアプリに追加<br/>(npm install)"]
    S4 ==> S5["⑤<br/>ローカル .env.local に<br/>キーを書く"]
    S5 ==> S6["✅<br/>ローカルで動作確認<br/>(SWA連携は Part 4)"]

    style S1 fill:#3ECF8E,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S6 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

**取得する2つの値:**

| キー | 役割 | 置き場所 |
| --- | --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | プロジェクトのURL | フロントOK |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` (旧 `anon_key`) | 公開可能なAPIキー | フロントOK |

> 🚨 `service_role` キーは**フロントに絶対置かない**。サーバー側専用。[supabase.md §12](../supabase/supabase.md) 参照。

> 📝 テーブル作成・RLS設定など Supabase 側の設計は [supabase.md §4-7](../supabase/supabase.md) 参照。本資料の射程外。

---

## 3-Y. Azure SQL Database (エンプラ本命)

Microsoft 純正の RDB。SQL Server 系で、エンプラ環境との整合性が高い。

```mermaid
flowchart LR
    S1["①<br/>Portalで<br/>'SQL databases'検索"] ==> S2["②<br/>+ 作成"]
    S2 ==> S3["③<br/>サーバー新規作成<br/>(なければ)"]
    S3 ==> S4["④<br/>認証方法選択<br/>(SQL or Entra)"]
    S4 ==> S5["⑤<br/>サイズ: Basic<br/>(個人開発)"]
    S5 ==> S6["⑥<br/>ファイアウォール<br/>'Azureサービスを許可'"]
    S6 ==> S7["✅<br/>接続文字列<br/>コピー"]

    style S1 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S6 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S7 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

**作成時の入力:**

| 項目 | 入れる値 |
| --- | --- |
| リソースグループ | `rg-my-app` |
| データベース名 | `db-my-app` |
| サーバー | 新規作成 → 名前 / 場所 / 管理者ログイン+パスワード |
| 認証方法 | SQL認証 (個人開発) / Entra ID (エンプラ) |
| 価格レベル | **Basic** ($5/月程度) または **無料サービス** (12ヶ月) |
| バックアップ冗長性 | ローカル冗長 (個人開発) |

**接続文字列の取得:**

- 作成完了後、DB リソースを開く
- 左メニュー「接続文字列」→ ADO.NET / JDBC / ODBC など言語別タブ
- パスワード部分は `{your_password}` プレースホルダなので、自分の値に置換してメモ

> 🚨 **ファイアウォール設定が必須**: 作成直後はインターネットから繋がらない。サーバーリソース → ネットワーク → 「Azure サービスとリソースにこのサーバーへのアクセスを許可する」 を **オン** にする。これで同じAzure内のSWA / Functionsから接続できる。

> 🚨 **接続文字列に管理者パスワードを直書きしない**: Part 4 で **Key Vault** に保管するのが本筋。

---

## 3-Z. Azure Database for PostgreSQL (OSS互換)

OSS Postgres そのもの。**Supabaseから移行しやすい**のが強み（SQL方言が同じ）。

```mermaid
flowchart LR
    S1["①<br/>Portalで<br/>'PostgreSQL'検索"] ==> S2["②<br/>+ 作成<br/>'Flexible Server'選択"]
    S2 ==> S3["③<br/>基本情報<br/>(サーバー名/管理者)"]
    S3 ==> S4["④<br/>サイズ: Burstable<br/>(B1ms = 一番小さい)"]
    S4 ==> S5["⑤<br/>ネットワーク<br/>'Azureサービスを許可'"]
    S5 ==> S6["⑥<br/>作成 (5-10分)"]
    S6 ==> S7["✅<br/>接続文字列<br/>コピー"]

    style S1 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S6 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S7 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

**作成時の入力:**

| 項目 | 入れる値 |
| --- | --- |
| リソースグループ | `rg-my-app` |
| サーバー名 | `pg-my-app` |
| デプロイの種類 | **Flexible server** (推奨) |
| バージョン | 16 (新しい安定版) |
| サイズ | Burstable B1ms (個人開発の最小) |
| 認証方法 | PostgreSQL 認証のみ / Entra ID 併用 |
| 管理者ユーザー / パスワード | 自分で決める |

> 📝 **「Single Server」は選ばない**: 旧世代で廃止予定。必ず **Flexible Server** を選ぶ。

> 📝 **Supabase からの移行**: Supabase の Dashboard → Database → Backups から dump を取り、Azure PostgreSQL に `pg_restore` で投入できる。スキーマはそのまま使える（RLSポリシーも標準Postgresの機能なので移植可能）。

---

# Part 4: つなぎ込み + デプロイ確認

ここまでで SWA・CI/CD・DB が揃った。**環境変数で繋ぎ込み**、**実際に変更がデプロイされる**ことを確認する。

## 4.1 環境変数の置き場所マトリクス

```mermaid
flowchart TB
    E["🔐 渡したい値"] ==> Q{"どんな値？"}

    Q -->|"ビルド時に必要<br/>(例: NEXT_PUBLIC_*)"| B["📦 <b>GitHub Secrets</b><br/>or Variable Group<br/>(Pipelines)"]
    Q -->|"実行時にサーバー読む<br/>(API_KEY, 接続文字列)"| R["⚙️ <b>SWA Configuration</b><br/>(Portal)"]
    Q -->|"本番DBパスワード等<br/>高セキュア"| K["🔑 <b>Key Vault</b><br/>参照"]

    style E fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style Q fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style B fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style K fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| 値 | 置き場 | 反映タイミング |
| --- | --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | GitHub Secrets (or Variable Group) | ビルド時 |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | GitHub Secrets (or Variable Group) | ビルド時 |
| `DATABASE_URL` (Azure SQL/PG接続文字列) | SWA Configuration | 実行時 |
| 本番DBパスワード | Key Vault + SWA Configuration から参照 | 実行時 |

詳しい仕組みは [azure.md §10](azure.md) 参照。

## 4.2 GitHub Actions 版でビルド時 env を渡す

Azureが自動生成した `.github/workflows/azure-static-web-apps-xxxx.yml` を編集する。

```mermaid
flowchart LR
    S1["①<br/>GitHubリポ →<br/>Settings →<br/>Secrets and variables"] ==> S2["②<br/>Actions タブ →<br/>New repository secret"]
    S2 ==> S3["③<br/>NEXT_PUBLIC_SUPABASE_URL<br/>等を追加"]
    S3 ==> S4["④<br/>git pull で<br/>workflow.yml取得"]
    S4 ==> S5["⑤<br/>'Build And Deploy'<br/>ステップに env: を追加"]
    S5 ==> S6["⑥<br/>commit & push<br/>→ 再ビルド"]

    style S1 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S6 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

> 🚨 **NEXT_PUBLIC_* の罠（再掲）**: Next.js の `NEXT_PUBLIC_*` 系は**ビルド時にコードへ埋め込まれる**ため、Azure Portal の Configuration（実行時）に入れても効かない。必ず **GitHub Secrets** に登録し、`env:` でビルドステップに渡すこと。

## 4.3 Azure Pipelines 版でビルド時 env を渡す

Pipelines は **Variable Group** を `azure-pipelines.yml` に紐づけて参照する。

```mermaid
flowchart LR
    S1["①<br/>Azure DevOps →<br/>Pipelines → Library"] ==> S2["②<br/>Variable group<br/>'app-env' 作成"]
    S2 ==> S3["③<br/>NEXT_PUBLIC_*<br/>を変数追加"]
    S3 ==> S4["④<br/>azure-pipelines.yml で<br/>variables: で参照"]
    S4 ==> S5["⑤<br/>npm run build ステップに<br/>env: で渡す"]
    S5 ==> S6["✅<br/>再実行 → 反映"]

    style S1 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S6 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

## 4.4 PR で Preview URL を体感する

ここが Vercel ライクな体験を初めて味わうハイライト。

```mermaid
sequenceDiagram
    participant U as 👨‍💻 開発者
    participant G as ☁️ Git (GH/Azure Repos)
    participant CI as ⚙️ CI/CD
    participant SWA as 🔷 Static Web Apps

    U->>G: ① ブランチ作成<br/>git checkout -b change-title
    U->>G: ② ファイル編集してpush
    U->>G: ③ PR作成
    G->>CI: Webhook通知
    CI->>CI: ④ Previewビルド
    CI->>SWA: ⑤ Staging環境にデプロイ
    SWA->>G: ⑥ PRに Staging URL通知<br/>(Checks欄に表示)
    Note over U: ⑦ Staging URL で<br/>変更を確認 ✨
    U->>G: ⑧ merge
    G->>CI: 本番デプロイ起動
    CI->>SWA: ⑨ 本番URL更新
```

**確認するポイント:**

| 確認 | どこで |
| --- | --- |
| ビルド成功/失敗 | GitHub Actions タブ / Azure DevOps Pipelines |
| Preview/Staging URL | PR の Checks タブ（コメントではなく） |
| デプロイ完了 | Portal → SWA → 環境 (Environments) タブ |
| 本番反映 | 本番URLで F5（キャッシュに注意） |

---

# Part 5: 後片付け / 撤収

検証が終わったら、リソースグループごと削除すると一発で全部消える。

```mermaid
flowchart LR
    S1["①<br/>Portal →<br/>リソースグループ"] ==> S2["②<br/>rg-my-app を開く"]
    S2 ==> S3["③<br/>'リソースグループの削除'"]
    S3 ==> S4["④<br/>名前を入力して確認"]
    S4 ==> S5["✅<br/>中身全部削除<br/>(課金もゼロに)"]

    style S1 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style S4 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S5 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 **Free tier なら放置しても課金されない**が、何を作ったか忘れがちなので、検証が一段落したら片付ける習慣を。Supabase プロジェクトも同様に Dashboard から削除可。

> 🚨 **GitHub 側のリポジトリ・ワークフローは残る**ので、本番運用しない場合は workflow.yml を消すか、リポジトリ自体をアーカイブする。

---

# 困ったときの初動

```mermaid
flowchart TB
    P["❓ 動かない"] ==> Q{"症状"}

    Q -->|"そもそも作れない"| A1["📍 サブスクリプション有無<br/>+ テナント確認"]
    Q -->|"ビルドが赤い"| A2["📍 GitHub Actions ログ<br/>or Pipelines ログ"]
    Q -->|"URL開いたら404"| A3["📍 SWA → 構成 →<br/>出力先設定"]
    Q -->|"DB繋がらない"| A4["📍 ファイアウォール<br/>+ 接続文字列"]
    Q -->|"環境変数効かない"| A5["📍 ビルド時=Secrets<br/>実行時=Configuration"]
    Q -->|"課金が怖い"| A6["📍 コスト管理 →<br/>予算アラート設定"]

    style P fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style Q fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style A1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A5 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A6 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

詳しい切り分け表は [azure.md §15](azure.md) 参照。

---

# 関連資料

- [azure.md](azure.md) — Azure 全体像 (本資料の概念編)
- [vercel.md](../vercel/vercel.md) — 比較対象の PaaS（CI/CD思想が同じ）
- [supabase.md](../supabase/supabase.md) — DB側に Supabase を選ぶ場合の概念
- [git.md](../git/git.md) — ブランチ運用・`.gitignore`・PR フロー
