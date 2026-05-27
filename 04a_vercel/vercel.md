# Vercel 全体像

> 📝 Vercel は「**作ったWebアプリを、ネット上に公開する作業をまるごと肩代わりしてくれるサービス**」。GitHubにpushすると、自動でビルド（ソースを公開用のファイルに変換）してネットに配信し、HTTPS化（鍵マーク🔒）まで全部やってくれる。サーバーを自分で用意しなくていい。

🧠 先に押さえておきたい2つの言葉:

- **CDN**: 世界中にサーバーのコピーを置いて、ユーザーの近くから速く配信する仕組み。日本から見ると東京、アメリカから見るとアメリカのサーバーが返事してくれる。
- **サーバーレス**: サーバーを常時起動せず、**呼ばれたときだけ瞬間起動する**処理の置き場。使った分だけしか課金されない。

---

## 1. Vercelが提供するもの

```mermaid
flowchart TB
    DEV["👨‍💻<br/><b>開発者</b><br/>git push するだけ"]

    subgraph VC["☁️ Vercel（フロントエンド一式）"]
        direction LR
        BUILD["🏗️<br/><b>Build</b><br/>ソースを<br/>公開用に変換"]
        CDN["🌐<br/><b>CDN</b><br/>世界中に置いて<br/>速く届ける"]
        SSL["🔒<br/><b>SSL</b><br/>HTTPS化を<br/>自動でやる"]
        FN["⚡<br/><b>Functions</b><br/>サーバー処理を<br/>呼ばれた時だけ実行"]
        PV["🔗<br/><b>Preview</b><br/>PRごとに<br/>専用URL発行"]
        BUILD ~~~ CDN ~~~ SSL ~~~ FN ~~~ PV
    end

    USER["📱<br/><b>エンドユーザー</b>"]

    DEV ==>|"git push"| VC
    VC ==>|"自動配信"| USER

    style DEV fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style VC fill:#F0F0F0,stroke:#000,stroke-width:3px,color:#000
    style BUILD fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style CDN fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style SSL fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style FN fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style PV fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style USER fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
```

開発者はGitHubに変更を**push**するだけ。Vercelが裏で「ビルド → CDNに配信 → HTTPS化」を全部やって、ユーザーがアクセスできるURLを発行する。サーバーの面倒は一切見なくていい。

> 📝 中心にあるのは **CDN（コンテンツ配信網）**。HTMLや画像のような「変わらないもの」は世界中のサーバーにコピーが置かれ、API（プログラムから呼び出すための窓口）のような「動的処理」はサーバーレス関数で動く。「全部CDNとサーバーレスに乗っている」と理解すると見通しがよくなる。

---

## 2. よくある勘違い

```mermaid
flowchart TB
    subgraph WRONG["❌ 勘違い"]
        direction LR
        W1["Vercel は<br/>Next.js 専用"] ==> W2["他のフレームワークは<br/>動かない？"]
    end

    subgraph RIGHT["⭕ 実際"]
        direction LR
        R1["Vercel は"] ==> R2["React / Vue / Svelte /<br/>Astro / Nuxt / 静的HTML<br/>なんでも動く"] ==> R3["Next.js は<br/>「最も最適化されてる」だけ"]
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
        WA["push したら<br/>すぐ本番反映"] ==> WB["怖くて使えない"]
    end

    subgraph R2["⭕ 実際"]
        direction LR
        RA["push 先のブランチで<br/>動作が違う"] ==> RB["main → 本番<br/>その他 → <b>Preview URL</b>"] ==> RC["PRごとに<br/>独立した検証環境"]
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
    S1["① vercel.com<br/>GitHubでサインアップ"] ==> S2["② New Project<br/>リポジトリを選ぶ"]
    S2 ==> S3["③ フレームワーク<br/>自動検出"]
    S3 ==> S4["④ 環境変数を設定<br/>（必要なら）"]
    S4 ==> S5["⑤ Deploy<br/>クリック"]
    S5 ==> S6["⑥ ✨ URL発行<br/>xxx.vercel.app"]

    style S1 fill:#000,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S4 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S5 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S6 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 GitHub連携してリポジトリを選ぶと、Vercelが自動で「これはNext.js／Reactね」と検出し、適切なビルド設定をかけてくれる。**設定ファイルを書かなくても動く**のがVercelの売り。

各ステップを補足すると:

| ステップ | 何をする | 補足 |
| -------- | -------- | ---- |
| ① サインアップ | GitHubアカウントでログイン | メアド登録だけでもOK |
| ② New Project | 公開したいリポジトリを選ぶ | Vercelに権限を渡す |
| ③ 自動検出 | Next.js / React等を判定 | 後述 §6 |
| ④ 環境変数 | APIキーなどの設定値を入力 | 後述 §7。空でもまず動く |
| ⑤ Deploy | ボタンを押すだけ | 数十秒〜数分待つ |
| ⑥ URL発行 | `xxx.vercel.app` が即発行 | 完了！ |

### 💸 無料枠（Hobby plan）の主な制限

| 項目 | 上限 | 詰まったら |
| ---- | ---- | --------- |
| 帯域幅 | 100GB/月 | Pro($20/月)へ |
| ビルド時間 | 6,000分/月 | Pro($20/月)へ |
| Functions実行 | 100,000回/日 | 大量呼び出しに注意 |
| Function実行時間 | 10秒/回 | Edge Functionsへ移行 |
| 商用利用 | ❌ 不可 | Pro必須 |
| チーム共有 | 個人専用 | Proでチーム機能 |

> 🚨 個人開発や趣味なら無料枠で十分。**商用サービスはPro必須**（ライセンス的に）。スタートアップが本番運用するなら最低Pro。

---

## 4. Git連携とデプロイの流れ

Vercelの核は**Gitとの連携**。pushがそのままデプロイ（=公開作業）のトリガーになる。

```mermaid
flowchart LR
    DEV["💻<br/>開発者"] ==>|"git push"| GH["☁️<br/>GitHub"]
    GH ==>|"Webhook通知"| VC["⚡<br/>Vercel"]
    VC ==>|"ビルド<br/>(npm run build)"| VC
    VC ==>|"成果物を<br/>CDNにアップ"| CDN["🌐<br/>CDN（世界中）"]
    CDN ==>|"高速配信"| USER["📱<br/>ユーザー"]

    style DEV fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style GH fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style VC fill:#000,color:#FFFFFF,stroke:#333,stroke-width:2px
    style CDN fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style USER fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 **Webhook（ウェブフック）** = 「何かが起きたら、別のサービスに自動で電話する」仕組み。ここではGitHubが「新しいpushが来たよ！」とVercelに知らせている。これがあるからVercelは**push直後に動き始められる**。

### ブランチによって行き先が変わる

```mermaid
flowchart TB
    P["📤 git push"] ==> Q{"どのブランチ？"}
    Q ==>|"main（master）"| PROD["🚀 <b>Production Deployment</b><br/>本番URL（例: myapp.com）に反映"]
    Q ==>|"その他ブランチ"| PRV["🔗 <b>Preview Deployment</b><br/>専用URL発行<br/>例: myapp-git-feature-xxx.vercel.app"]

    style P fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style Q fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style PROD fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style PRV fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 「**Production = mainブランチ**」が初期設定。設定で別ブランチを本番にすることもできるが、まずはこの形で覚えればOK。詳細は [git.md](../02_git/git.md) §4 のPRフローと組み合わせて読むと理解が深まる。

---

## 5. Preview Deployment — PRごとの検証URL

VercelのいちばんユニークなところがこのPreview。**プルリクエストごとに独立したURL**が自動で発行される。

```mermaid
sequenceDiagram
    participant D as 👨‍💻 開発者
    participant G as ☁️ GitHub
    participant V as ⚡ Vercel
    participant R as 👀 レビュアー

    D->>G: git push feature/login
    D->>G: PRを作成
    G->>V: Webhook（新しいブランチ）
    V->>V: ビルド
    V->>G: PRに Preview URL をコメント<br/>例: myapp-git-feature-login.vercel.app
    R->>V: URL を開く
    Note over R: ✨ 実際に動かして確認できる
```

### 何が嬉しいの？

```mermaid
flowchart TB
    Q["✨ Preview のメリット"] ==> M1["✅ <b>レビュアーが実物を触れる</b><br/>コードだけでなくUIで確認"]
    Q ==> M2["✅ <b>ステージング環境不要</b><br/>PRごとに勝手に立つ"]
    Q ==> M3["✅ <b>本番と隔離</b><br/>壊しても影響ゼロ"]
    Q ==> M4["✅ <b>非エンジニアも確認可能</b><br/>URLをSlackに貼るだけ"]

    style Q fill:#000,color:#FFFFFF,stroke:#333,stroke-width:3px
    style M1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style M2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style M3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style M4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 PRをマージするまでPreview URLは生きている。マージ後は自動的に**Production**へ反映され、Preview URLは履歴として残る。

---

## 6. プロジェクトとフレームワーク自動検出

Vercelは`package.json`を見て、フレームワークを推測する。**設定ファイルを書かなくても動く**のが基本。

> 📝 **package.json** = JavaScript/TypeScriptプロジェクトの「設計書」。「このアプリは何を使ってるか（React、Next.jsなど）」「ビルドするときのコマンドは何か」が書かれている。Vercelはこれを覗いて判断する。

```mermaid
flowchart LR
    PJ["📦 package.json"] ==>|"依存を解析"| D{"検出"}
    D ==>|"next がある"| F1["⚛️ Next.js"]
    D ==>|"react-scripts"| F2["⚛️ Create React App"]
    D ==>|"vite"| F3["⚡ Vite"]
    D ==>|"nuxt"| F4["💚 Nuxt"]
    D ==>|"astro"| F5["🚀 Astro"]
    D ==>|"何もなし"| F6["📄 静的HTML"]

    F1 ==> B["🏗️ 適切な<br/>ビルドコマンド自動設定"]
    F2 ==> B
    F3 ==> B
    F4 ==> B
    F5 ==> B
    F6 ==> B

    style PJ fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style D fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style B fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

### 主なビルド設定の項目

| 項目 | 何を指定する | デフォルト |
| ---- | ----------- | --------- |
| Framework Preset | フレームワーク名 | 自動検出 |
| Build Command | ビルド時に実行 | `npm run build` |
| Output Directory | 成果物の場所 | フレームワーク準拠 |
| Install Command | 依存インストール | `npm install` |
| Root Directory | プロジェクトのルート | リポジトリ直下 |

> 📝 モノレポ（1つのリポジトリに複数アプリ）の場合、**Root Directory**で「このアプリは`apps/web/`を見て」と指定するのが定番。

---

## 7. 環境変数 — 3つの環境を使い分ける

> 📝 **環境変数** = APIキー・パスワード・DBの接続先など、「**コードに直接書き込みたくない設定値**」を外から渡す仕組み。たとえば`API_KEY=abc123`のように「名前と値」のペアで持つ。コードからは「名前」で呼び出す。Git管理から外したい秘密情報の置き場として標準的（[git.md](../02_git/git.md) §8 の `.env` の話と地続き）。

VercelはAPIキーなどを**3つの環境（Production / Preview / Development）ごとに分けて持てる**のが特徴。

```mermaid
flowchart TB
    ENV["🔐 環境変数<br/>(API_KEY など)"] ==> E1["🚀 <b>Production</b><br/>本番デプロイ"]
    ENV ==> E2["🔗 <b>Preview</b><br/>PR / その他ブランチ"]
    ENV ==> E3["💻 <b>Development</b><br/>ローカル（vercel dev）"]

    E1 ==>|"本番DB"| DB1["🗄️ 本番DB / 本番Stripe"]
    E2 ==>|"テストDB"| DB2["🗄️ 開発DB / テストStripe"]
    E3 ==>|"ローカルDB"| DB3["🗄️ ローカル / モック"]

    style ENV fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style E1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style E2 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style E3 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style DB1 fill:#fff,color:#000,stroke:#27AE60
    style DB2 fill:#fff,color:#000,stroke:#000
    style DB3 fill:#fff,color:#000,stroke:#3498DB
```

> 📝 「Previewに本番DBのキーを入れない」が鉄則。PreviewはPRごとに自動で立つので、**間違ったマイグレーションを誤って本番に流す事故**を防ぐ。

### 環境変数を入れる場所

```mermaid
flowchart LR
    S["📍 Dashboard<br/>Settings → Environment Variables"] ==> A["Name と Value<br/>を入力"]
    A ==> B["✅ 適用先を選ぶ<br/>(Production / Preview / Development)"]
    B ==> C["Save"]
    C ==> D["⚡ 次のデプロイから反映"]

    style S fill:#000,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style C fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style D fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 🚨 環境変数を変えても、**既にデプロイ済みのバージョンには反映されない**。再デプロイ（Redeploy）が必要。「変えたのに動かない」のほぼ全部がこれ。

---

## 8. ドメインとSSL

Vercelでデプロイすると、最初から **`xxx.vercel.app`** という無料のサブドメインがもらえる。独自ドメインも数分で接続できる。

> 📝 **ドメイン**は「`example.com` のような住所」。**DNS**は「住所からサーバーの実体（IPアドレス）に変換する仕組み」、いわば**インターネットの住所録**。独自ドメインを使うときは、ドメイン業者の管理画面で「このドメインはVercelを指してください」とDNSに書き込む。

```mermaid
flowchart LR
    P["📦 プロジェクト"] ==>|"デフォルト"| V["🌐 xxx.vercel.app<br/>(無料・即発行)"]
    P ==>|"独自ドメイン"| C["🌐 myapp.com<br/>(Settings → Domains)"]

    C ==>|"DNS設定"| DNS["📡 ドメイン業者で<br/>Aレコード or CNAME"]
    DNS ==> SSL["🔒 Vercelが<br/>SSL証明書を自動発行<br/>(Let's Encrypt)"]
    SSL ==> READY["✅ HTTPSで公開"]

    style P fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style V fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style DNS fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style SSL fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style READY fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

> 📝 **SSL / HTTPS** = ブラウザとサーバー間の通信を暗号化する仕組み。鍵マーク🔒の正体。**Let's Encrypt** という無料で証明書を発行してくれる団体があり、Vercelは内部でこれを使う。**証明書の更新も含めて完全自動**。昔は手動でLet's Encryptを叩いていた作業が、Vercelでは「ドメイン繋ぐ」だけで終わる。

---

## 9. Functions — サーバー処理を書きたいとき

「APIを置きたい」「DBに書き込みたい」「外部サービスを叩きたい」など**サーバー処理**が必要なら、Vercelでは**関数（function）を1ファイル置くだけで動く**。

```mermaid
flowchart LR
    APP["📱 アプリ"] ==>|"呼び出す"| FN["⚡ Function<br/>(1ファイル置くだけ)"]
    FN ==>|"必要なら"| DB["🗄️ DB"]
    FN ==>|"外部API"| EXT["🌐 Stripe / OpenAI など"]
    FN ==>|"レスポンス"| APP

    style APP fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style FN fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style DB fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style EXT fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 サーバーを自分で立てる必要はない。**呼ばれたときだけ瞬間起動して、返事したら消える**（=サーバーレス）。常時稼働しないので、安く・スケールしやすい。

### 実は2種類ある: Serverless と Edge

最初はあまり気にしなくていいが、Vercelの関数には2タイプある。**「速さ重視か、何でもできる重視か」の違い**と思えばOK。

| | 🖥️ **Serverless Functions** | 🌐 **Edge Functions** |
| --- | --- | --- |
| 実行場所 | 固定リージョン（例: 東京） | 世界中（ユーザー近く） |
| 速さ | 普通 | 超速い |
| できること | **なんでもできる**（Node.jsの全機能、npmパッケージ自由） | **軽い処理だけ**（認証チェック・リダイレクトなど） |
| 実行時間 | 長め（数秒〜数十秒OK） | 短い（数百ミリ秒推奨） |
| 迷ったら | **こちらが基本** | レイテンシ最優先のときに |

> 📝 Next.jsの場合、`app/api/xxx/route.ts` を置けば自動的にServerless Functionになる。「Edge化」は1行追加するだけで切り替え可能。**まずはServerlessから始めて、必要に応じてEdgeにする**のが順当。

---

## 10. ロールバック — 過去のデプロイに戻す

本番に問題が出たら、**過去の任意のデプロイに1クリックで戻せる**。これがVercel運用の安心感の正体。

```mermaid
flowchart LR
    D1["😱 本番でバグ発覚"] ==> D2["📜 Deployments一覧<br/>過去のデプロイを選択"]
    D2 ==> D3["⚡ Promote to Production"]
    D3 ==> D4["✅ 即座に切り替わる<br/>(ビルド不要)"]

    style D1 fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
    style D2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style D3 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style D4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 過去のビルド成果物はそのまま残っているので、**「もう一度ビルド」が要らない**のがポイント。再デプロイではなく、配信先を切り替えるだけ。

注意点として、DBのスキーマ（テーブル構造）が変わっている場合は古いコードが新しいDBで動かないこともある。コードだけで完結する変更でないなら、慎重にロールバック。

> 🚨 [supabase.md](../03_supabase/supabase.md) §11のマイグレーション運用と合わせて考える。

---

## 11. ビルドキャッシュとビルドログ

毎回フルビルドだと時間がかかるので、Vercelは**ビルド成果物をキャッシュ**する（再利用のために取っておく）。

```mermaid
flowchart LR
    P1["📤 git push"] ==> B{"ビルド"}
    B ==>|"前回と同じ依存"| C["♻️ <b>キャッシュ利用</b><br/>node_modules を再利用"]
    B ==>|"依存が変わった"| F["🏗️ <b>フルビルド</b><br/>npm install から"]
    C ==> D["⚡ 高速デプロイ"]
    F ==> D2["⏱️ 通常デプロイ"]

    style P1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style C fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style D fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style D2 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
```

### ビルドが失敗したら？

```mermaid
flowchart LR
    F["❌ ビルド失敗"] ==> L["📜 <b>Deployments → 該当デプロイ</b><br/>Build Logs を開く"]
    L ==> E["🔍 エラーログ確認"]
    E ==>|"依存不足"| F1["package.json を修正"]
    E ==>|"環境変数"| F2["Settings → Env Variables を確認"]
    E ==>|"型エラー"| F3["ローカルで npm run build を試す"]

    style F fill:#C0392B,color:#FFFFFF,stroke:#333,stroke-width:2px
    style L fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style E fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 「ローカルで動くのにVercelで動かない」のほとんどは**環境変数の設定漏れ**か**Node.jsバージョンの差**。Settings → General で Node version も指定できる。

---

## 12. 迷ったらどこを見る？

```mermaid
flowchart LR
    Q["❓ 困った"] ==> D["<b>Dashboard → Deployments</b><br/>📦 デプロイ履歴と状態"]
    Q ==> L["<b>該当デプロイ → Build Logs</b><br/>🏗️ ビルド失敗の原因"]
    Q ==> F["<b>該当デプロイ → Functions</b><br/>⚡ APIの実行ログ"]
    Q ==> E["<b>Settings → Environment Variables</b><br/>🔐 環境変数の確認"]
    Q ==> A["<b>Settings → Domains</b><br/>🌐 ドメイン / DNS設定"]

    style Q fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style D fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style L fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style E fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

「デプロイは成功してるのに動かない」のほとんどは**環境変数の未設定**か**反映漏れ**。まずSettings → Env Variablesを疑う。

---

## 13. 補足: Netlify / Cloudflare Pages / Azure SWA との違い

```mermaid
flowchart TB
    subgraph VC["⚡ Vercel"]
        direction TB
        V1["Next.js に最適化"]
        V2["Serverless / Edge<br/>両対応"]
        V3["Preview URL が秀逸"]
    end

    subgraph NL["💚 Netlify"]
        direction TB
        N1["フレームワーク中立"]
        N2["Forms / Identity など<br/>独自機能多め"]
        N3["老舗・安定"]
    end

    subgraph CF["🟧 Cloudflare Pages"]
        direction TB
        C1["CDN最強・無料枠が広い"]
        C2["Workers と統合"]
        C3["ネットワーク層が強力"]
    end

    subgraph AZ["🔷 Azure Static Web Apps"]
        direction TB
        A1["総合クラウドの一部"]
        A2["Free tierでも商用OK"]
        A3["エンプラ/社内縛りに強い"]
    end

    style VC fill:#F0F0F0,stroke:#000,stroke-width:2px,color:#000
    style NL fill:#E8F8E8,stroke:#27AE60,stroke-width:2px,color:#000
    style CF fill:#FFF4E5,stroke:#E67E22,stroke-width:2px,color:#000
    style AZ fill:#E5F1FB,stroke:#0078D4,stroke-width:2px,color:#000
    style V1 fill:#fff,color:#000,stroke:#000
    style V2 fill:#fff,color:#000,stroke:#000
    style V3 fill:#fff,color:#000,stroke:#000
    style N1 fill:#fff,color:#000,stroke:#27AE60
    style N2 fill:#fff,color:#000,stroke:#27AE60
    style N3 fill:#fff,color:#000,stroke:#27AE60
    style C1 fill:#fff,color:#000,stroke:#E67E22
    style C2 fill:#fff,color:#000,stroke:#E67E22
    style C3 fill:#fff,color:#000,stroke:#E67E22
    style A1 fill:#fff,color:#000,stroke:#0078D4
    style A2 fill:#fff,color:#000,stroke:#0078D4
    style A3 fill:#fff,color:#000,stroke:#0078D4
```

**結論: 4者とも「Gitに繋いでpushすればデプロイ」という体験は同じ。差は周辺機能と最適化の方向性。**

- **Vercel**: Next.jsを作っている会社が運営。Next.jsを使うなら最短ルート
- **Netlify**: 一番老舗。Vercelより前からこのモデルを作った
- **Cloudflare Pages**: 帯域無料・ネットワーク強力。コストとパフォーマンス重視ならアリ
- **Azure Static Web Apps**: Microsoftの総合クラウドの一部。**Free tierで商用OK**（Vercel Hobbyより緩い）、エンプラ/社内環境縛りなら本命（[azure.md](../04b_azure/azure.md) §6参照）

> 📝 「**CDN 1個に、Build / Functions / Preview URL を全部生やしたもの**」というのが一番しっくりくる説明。Supabaseが「**Postgresに全部生やしたもの**」、Azureが「**クラウドのデパートに全部並べたもの**」だったのと対になる構造（[supabase.md](../03_supabase/supabase.md) §14 / [azure.md](../04b_azure/azure.md) §16 参照）。
