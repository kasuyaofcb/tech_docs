# Azure DevOps 全体像

> 📝 [azure.md](azure.md) が **Azure クラウドの全体像**、[setup.md](setup.md) が **アプリリリースの実手順** だったのに対し、本資料は **Azure DevOps (旧 VSTS)** という「**Microsoft 版の GitHub**」の全体像をまとめる。Azure 本体 (`portal.azure.com`) とは別サイト (`dev.azure.com`) で動く、コード管理 + CI/CD + 課題管理の総合スイート。

🧠 先に押さえておきたい3つの言葉:

- **Azure DevOps**: Microsoft が提供する開発者向け総合プラットフォーム。GitHub 相当の機能を1つのスイートにまとめたもの。
- **Organization**: Azure DevOps の最上位のまとまり (会社・チーム単位)。URL: `dev.azure.com/{organization}`
- **Project**: Organization の中に作る、Repos / Pipelines / Boards 等を内包する作業の箱。GitHub の Repository より粒度が大きい (1 Project に複数の Repos を持てる)。

---

## 1. Azure DevOps が提供するもの

```mermaid
flowchart TB
    USER["👨‍💻<br/><b>開発者</b>"]

    subgraph AZD["☁️ Azure DevOps (dev.azure.com)"]
        direction LR
        REPOS["📦<br/><b>Repos</b><br/>Git ホスティング"]
        PIPE["🚀<br/><b>Pipelines</b><br/>CI/CD"]
        BOARDS["📋<br/><b>Boards</b><br/>課題/プロジェクト管理"]
        ARTI["🗄️<br/><b>Artifacts</b><br/>パッケージ管理<br/>(npm/NuGet等)"]
        TEST["🧪<br/><b>Test Plans</b><br/>テスト計画管理"]
        REPOS ~~~ PIPE ~~~ BOARDS ~~~ ARTI ~~~ TEST
    end

    AZURE["🔷<br/><b>Azure リソース</b><br/>(SWA / Functions /<br/>App Service 等)"]

    USER ==>|"git push / レビュー"| AZD
    AZD ==>|"自動デプロイ"| AZURE

    style USER fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style AZD fill:#E5F1FB,stroke:#0078D4,stroke-width:3px,color:#000
    style REPOS fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style PIPE fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style BOARDS fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style ARTI fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style TEST fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style AZURE fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:3px
```

`dev.azure.com` 配下に5つのサービスがまとまっており、**個別に取捨選択して使える**。一番よく使うのは Repos + Pipelines の組み合わせ。

> 📝 Azure DevOps は **Azure 本体 (`portal.azure.com`) とは別サイト**。よくある混同として「Azure に CI/CD を構築する」=「Portal で何かを設定する」と思いがちだが、実際は別ドメインの別サービスを開いて作業する。

---

## 2. よくある勘違い

```mermaid
flowchart TB
    subgraph WRONG["❌ 勘違い"]
        direction LR
        W1["Azure DevOps =<br/>Azure リソースの<br/>管理画面"] ==> W2["Portal の中に<br/>あるはず"]
    end

    subgraph RIGHT["⭕ 実際"]
        direction LR
        R1["Azure DevOps は<br/>独立した<br/>開発者向けサイト"] ==> R2["dev.azure.com<br/>(別ドメイン)"] ==> R3["GitHub と<br/>機能対応する"]
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
        WA["GitHubと<br/>Azure DevOps は<br/>どちらか一方"] ==> WB["乗り換えが必要"]
    end

    subgraph R2["⭕ 実際"]
        direction LR
        RA["両方の remote を<br/>同じローカルリポに<br/>持てる"] ==> RB["同じ Azure リソースに<br/>両方からデプロイ可"] ==> RC["並行運用 / 移行<br/>リハーサルが可能"]
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

## 3. 階層構造 — Organization / Project / Repos

Azure DevOps の階層は GitHub と微妙に違い、**Organization → Project → 各サービス** という3階層構造。

```mermaid
flowchart TB
    ORG["🏢 <b>Organization</b><br/>(会社・チーム単位)<br/>URL: dev.azure.com/{org}"]
    ORG ==> PRJ["📁 <b>Project</b><br/>(作業の箱)<br/>URL: dev.azure.com/{org}/{project}"]
    PRJ ==> R1["📦 Repos<br/>(複数のGitリポ)"]
    PRJ ==> R2["🚀 Pipelines"]
    PRJ ==> R3["📋 Boards"]
    PRJ ==> R4["🗄️ Artifacts"]
    PRJ ==> R5["🧪 Test Plans"]

    style ORG fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:3px
    style PRJ fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:3px
    style R1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R3 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R4 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R5 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
```

| 階層 | 役割 | GitHub の対応 |
| ---- | ---- | --- |
| 🏢 **Organization** | 全体の入れ物 | GitHub Organization (≒) |
| 📁 **Project** | 1つの製品・サービスごとに作る箱 | （GitHub には対応概念なし） |
| 📦 **Repos** | Git リポジトリ (Project に複数持てる) | Repository |

> 📝 GitHub では「Organization の直下に Repository」だが、Azure DevOps では **間に Project が1つ挟まる**。1 Project に複数 Repos を持てるため、関連する複数のリポを1箱にまとめられる。

---

## 4. GitHub ⇔ Azure DevOps 対応表

git そのもの (branch / commit / merge / pull / push) は**完全に同一**。違うのは「コードホスティングの UI」と「課題管理の概念」のみ。

```mermaid
flowchart LR
    subgraph G["🟪 GitHub"]
        direction TB
        G1["Repository"]
        G2["Issues / Projects"]
        G3["Actions"]
        G4["Packages"]
        G5["Branch protection<br/>rules"]
        G6["Pull Request"]
    end

    subgraph A["🟦 Azure DevOps"]
        direction TB
        A1["Repos"]
        A2["Boards"]
        A3["Pipelines"]
        A4["Artifacts"]
        A5["Branch policies"]
        A6["Pull Request"]
    end

    G1 -.-> A1
    G2 -.-> A2
    G3 -.-> A3
    G4 -.-> A4
    G5 -.-> A5
    G6 -.-> A6

    style G fill:#F0F0F0,stroke:#8E44AD,stroke-width:2px,color:#000
    style A fill:#E5F1FB,stroke:#0078D4,stroke-width:2px,color:#000
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

### 主要機能の対応表

| 概念 | GitHub | Azure DevOps | 違い |
| --- | --- | --- | --- |
| Git ホスティング | Repository | Repos | 構造的にはほぼ同じ |
| CI/CD | GitHub Actions | Pipelines | YAML 構造が似ている (詳細は [azure.md §11.2](azure.md)) |
| 課題管理 | Issues / Projects | Boards | Boards の方が WBS 寄りで重厚 |
| パッケージ | GitHub Packages | Artifacts | — |
| テスト計画 | (なし) | Test Plans | Azure 独自 |
| ブランチ保護 | Branch protection rules | Branch policies | 機能はほぼ同じ、用語違い |
| 必須レビュワー | Required reviewers | Required reviewers | 完全同一 |
| CI 必須化 | Required status checks | Build validation | 機能同じ |
| マージ後の自動削除 | Auto-delete branches | Delete source branch after merge | 機能同じ |
| Draft PR | Draft pull request | Draft pull request | 完全同一 |
| Issue 紐付け記法 | `Closes #123` | `AB#123` (Boards) | 記法違い、仕組み同じ |
| 認証トークン | Personal Access Token | Personal Access Token (PAT) | 用語同一 |

> 📝 **ローカル側の git コマンドは完全に何も変わらない** (`git checkout` / `commit` / `push` 等)。違うのは「PR を開く UI」と「ブランチ保護の設定画面」だけ。

---

## 5. はじめの一歩 (Organization 作成まで)

```mermaid
flowchart LR
    S1["①<br/>aex.dev.azure.com<br/>にアクセス"] ==> S2["②<br/>Microsoftアカウント<br/>でサインイン"]
    S2 ==> S3["③<br/>Organization 名<br/>+ リージョン<br/>+ サブスク選択"]
    S3 ==> S4["④<br/>Project 作成<br/>(Repos等が自動付帯)"]
    S4 ==> S5["✅<br/>準備完了"]

    style S1 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

> 🚨 **`dev.azure.com` が Azure Portal にリダイレクトされる罠**: 既存の Azure ログインセッションがあると、`dev.azure.com` を開いても `portal.azure.com` に飛ばされることがある。確実なのは **`https://aex.dev.azure.com/`** (Organization 作成専用ページ) を直接叩く方法。
>
> 🚨 **リージョンは地域単位**: Azure 本体は「Japan East」等の国単位だが、**Azure DevOps の選択肢は地域単位** (`Asia Pacific` / `East US` 等)。日本ラベルは出ない。日本向けは Asia Pacific を選ぶ (実体はシンガポール or 香港 DC)。

具体的な手順とハマりどころは [setup.md §2-B Step ①](setup.md) を参照。

---

## 6. CI/CD: Pipelines

Azure Pipelines は **GitHub Actions の Azure DevOps 版**。

```mermaid
flowchart LR
    PUSH["📤 git push<br/>(Azure Repos)"] ==>|"trigger"| P["🚀 Pipelines"]
    P ==> AGENT{"どのエージェント？"}
    AGENT -->|"標準"| MH["🟦 Microsoft-hosted<br/>(クラウド側VM)"]
    AGENT -->|"閉域要件"| SH["🟩 Self-hosted<br/>(組織内VM)"]
    MH ==> DEPLOY["🔷 Azureリソース<br/>(SWA等) へデプロイ"]
    SH ==> DEPLOY

    style PUSH fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
    style AGENT fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style MH fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style SH fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style DEPLOY fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| 観点 | GitHub Actions | Azure Pipelines |
| --- | --- | --- |
| 定義ファイル | `.github/workflows/*.yml` | `azure-pipelines.yml` (リポ直下) |
| シークレット | GitHub Secrets | Pipeline Variables / Variable Group |
| エージェント | GitHub-hosted (Microsoft管理) | Microsoft-hosted (Azure管理) / Self-hosted |
| Azure 公式タスク | `Azure/static-web-apps-deploy@v1` | `AzureStaticWebApp@0` |

> 📝 YAML の概念対応表は [azure.md §11.2](azure.md) に詳細あり。`trigger` / `jobs` / `steps` / `task` の構造はそっくりで翻訳が容易。

---

## 7. 並行運用パターン

「GitHub を本番のホストにしつつ、Azure DevOps を**学習・移行リハーサル・冗長化**のために並行で動かす」という構成が現実的に有用。

```mermaid
flowchart LR
    L["📁 ローカルリポ<br/>(remote 2つ持ち)"]
    L ==>|"git push origin"| GH["🟪 GitHub<br/>+ Actions"]
    L ==>|"git push azure"| AR["🟦 Azure DevOps<br/>+ Pipelines"]
    GH ==>|"AzureStaticWebApp"| SWA["🔷 同一の SWA"]
    AR ==>|"AzureStaticWebApp"| SWA

    style L fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style GH fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style AR fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style SWA fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

### 同一ローカルリポに 2 つの remote を持つ

| コマンド | 用途 |
| --- | --- |
| `git remote add azure <Azure Repos URL>` | Azure Repos を 2nd remote として追加 |
| `git push origin main` | GitHub 側に push → GitHub Actions が起動 |
| `git push azure main` | Azure Repos 側に push → Pipelines が起動 |
| `git push origin main && git push azure main` | 両方に push → 両方の CI/CD が起動 (同じ SWA にデプロイ) |

> 📝 **デプロイトークンは同じものを両方で使う**ため、SWA リソースは1つで済む。同じ SWA に対して GitHub Actions / Azure Pipelines のどちらが直近に push したかで内容が反映される動作になる。

### 並行運用が活きる場面

| 用途 | 詳細 |
| --- | --- |
| **移行リハーサル** | GitHub Actions で動くものを残しつつ Azure DevOps を試し、動作確認後に切替 |
| **学習・比較** | 同じ処理を2つの CI/CD で動かして UI と挙動の差分を体感 |
| **冗長化** | 片方の CI/CD が障害でも、もう片方からデプロイ継続可能 |

---

## 8. 認証 — PAT と Service Principal

```mermaid
flowchart TB
    Q["🔐 何の認証？"] ==> Q1{"対象"}

    Q1 -->|"開発者が手元から<br/>Azure Repos に push"| PAT["🔑 <b>Personal Access<br/>Token (PAT)</b><br/>個人発行・短期"]
    Q1 -->|"Pipelines から<br/>Azureリソース操作"| SP["🤖 <b>Service Principal</b><br/>Service Connection<br/>経由 (ロボットID)"]
    Q1 -->|"Pipelines から<br/>SWAデプロイ"| TOK["🎫 <b>Deployment Token</b><br/>SWAリソース固有"]

    style Q fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style Q1 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style PAT fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style SP fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style TOK fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| 認証種別 | 用途 | 取得場所 |
| --- | --- | --- |
| **Personal Access Token (PAT)** | 個人ユーザーが Azure Repos に git push する時の認証 | Azure DevOps → ユーザーアイコン → Personal access tokens |
| **Service Principal** | Pipelines が Azure 全般のリソースを操作する時の認証 (App Service デプロイ等) | Azure DevOps → Project Settings → Service connections |
| **Deployment Token** | Pipelines が **SWA に絞って**デプロイする時の認証 (Service Principal より軽量) | Azure Portal → SWA → デプロイトークンの管理 |

> 🚨 **PAT は表示直後にコピー必須**: 発行時の画面を閉じると**再表示できない**。コピーし忘れた場合は新しいトークンを作り直す。Scope は最小限 (`Code (Read & write)` 等) に絞るのが推奨。
>
> 📝 **SWA だけが目的なら Service Connection は不要**。Deployment Token を Variable に登録すれば動く。Service Connection が必要になるのは App Service / Container Apps / Key Vault 等の汎用リソース操作の時。

---

## 9. PR とブランチ運用

git そのものの概念は GitHub と完全同一 ([git.md §3〜§4](../02_git/git.md) と同じ)。違いは UI と用語のみ。

```mermaid
flowchart LR
    B1["🌿 feature ブランチで作業<br/>(git そのまま)"] ==>|"push"| AR["📦 Azure Repos"]
    AR ==> PR["📋 Pull Request 作成<br/>(Azure DevOps UI)"]
    PR ==>|"Build validation<br/>(Pipelines自動実行)"| CI["⚙️ CI チェック"]
    CI ==>|"成功"| REV["👁️ Required reviewers<br/>承認"]
    REV ==>|"承認後"| MERGE["🔀 main へ merge"]
    MERGE ==>|"trigger"| DEPLOY["🚀 本番デプロイ<br/>(Pipelines)"]

    style B1 fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style AR fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style PR fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style CI fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style REV fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style MERGE fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style DEPLOY fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

### Branch Policies で強制できるルール

GitHub の **Branch protection rules** に相当する機能。Project Settings → Repositories → 対象リポ → Policies で設定。

| ポリシー | 内容 |
| --- | --- |
| Require a minimum number of reviewers | レビュー必須人数 |
| Check for linked work items | Boards の作業項目との紐付け強制 |
| Build validation | 指定 Pipeline の成功を merge 条件にする |
| Limit merge types | Merge / Squash / Rebase のうち許可する方式を絞る |
| Automatically delete the source branch | merge 後にブランチ削除 |

> 📝 Azure DevOps は **GitHub よりやや「ガッチリ管理する文化」寄りの UI**。チーム規模や業務要件で細かいポリシーを設定したい場合に向いている。

---

## 10. 個人・小規模利用の無料枠

```mermaid
flowchart LR
    F["🆓 Azure DevOps<br/>Free tier"] ==> F1["📦 Private リポ<br/><b>無制限</b>"]
    F ==> F2["🚀 Microsoft-hosted<br/>Pipelines<br/><b>1,800分/月</b>"]
    F ==> F3["👥 ユーザー<br/><b>5人まで</b>"]
    F ==> F4["📋 Boards / Artifacts<br/>基本機能"]

    style F fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:3px
    style F1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 個人検証や小規模チームなら Free tier の範囲内で十分。Microsoft-hosted Agent の 1,800分/月は、軽量 SWA デプロイなら1回数分で済むため**月数百回のビルドが可能**。

---

## 11. 関連資料

- [azure.md](azure.md) — Azure クラウド全体像 (Compute / DB / Identity / DevOps)
  - §11 GitHub以外からのCI/CD — Azure Pipelines の詳細比較
- [setup.md](setup.md) — Azure 環境構築手順
  - §2-B Azure Pipelines版 — DevOps からの実デプロイ手順
- [git.md](../02_git/git.md) — Git 全体像 (Azure Repos でも完全に同じ)
- [vercel.md](../04a_vercel/vercel.md) — 比較対象 (Git連携CI/CDの別パターン)

> 📝 「**GitHub と同じことを Microsoft 配下で完結させたい時の選択肢**」というのが Azure DevOps の位置づけ。業務利用での要件 (Entra ID連携 / 閉域 / 監査) が出てきた時の選択肢としても有効。
