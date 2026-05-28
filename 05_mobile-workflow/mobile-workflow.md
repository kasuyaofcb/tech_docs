# スマホ開発ワークフロー — AIアシスタント × クラウドGit

> 📝 「スマホで開発」は従来「コードエディタアプリで頑張る」という発想だった。**AIアシスタントの登場で構図が変わり**、スマホブラウザ → AIアシスタント → クラウドGit という流れで、**コードの修正・コミット・push までスマホで完結**できる。本資料はその全体像と境界を整理する。

🧠 先に押さえておきたい言葉:

- **AIアシスタント (Web版)**: ブラウザでアクセスできるAIサービス。Claude.ai / ChatGPT / Gemini など。
- **リポジトリ連携機能**: AIアシスタントが Git ホスト (GitHub等) のリポジトリを直接読み書きできる機能。サービス毎に名称が異なる (Claude Code on the web、ChatGPT Codex、GitHub Copilot Workspace など)。
- **クラウドGit**: GitHub / GitLab / Bitbucket / Azure Repos など、ブラウザからアクセス可能な Git ホスティングサービス。

---

## 1. 全体像

```mermaid
flowchart LR
    USER["📱<br/><b>利用者</b><br/>スマホブラウザ"]

    subgraph AI["☁️ AIアシスタント (Web版)"]
        direction TB
        CONV["💬 会話<br/>(壁打ち)"]
        CONN["🔗 リポジトリ<br/>連携機能"]
        CONV ~~~ CONN
    end

    GIT["📦<br/><b>クラウドGit</b><br/>GitHub/GitLab等"]

    USER ==>|"指示・対話"| AI
    AI ==>|"コード読み書き"| GIT
    GIT -.->|"差分を返す"| AI
    AI -.->|"結果を表示"| USER

    style USER fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style AI fill:#F0F0F0,stroke:#000,stroke-width:3px,color:#000
    style CONV fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style CONN fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style GIT fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

利用者は**スマホブラウザから AI アシスタントと対話**するだけ。実際の「リポジトリを読む」「ファイルを編集する」「コミット・push する」操作は AI が代行する。

> 📝 この構図の本質は、**「キーボード入力 + エディタ画面」というコーディングの前提を取り払った**こと。発想を言葉で渡せばコードに落ちる。スマホの狭い画面でもエディタを開く必要がなくなる。

---

## 2. よくある勘違い

```mermaid
flowchart TB
    subgraph WRONG["❌ 勘違い"]
        direction LR
        W1["スマホで開発<br/>=エディタアプリで<br/>頑張る"] ==> W2["画面狭くて<br/>実用にならない"]
    end

    subgraph RIGHT["⭕ 実際"]
        direction LR
        R1["AIに指示を<br/>言葉で渡す"] ==> R2["コード変更は<br/>AIが代行"] ==> R3["利用者は<br/>レビュー役"]
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
        WA["スマホで本格<br/>開発できる<br/>(なんでも可能)"] ==> WB["大規模変更も<br/>スマホで"]
    end

    subgraph R2["⭕ 実際"]
        direction LR
        RA["スマホで完結する<br/>のは規模が小さい<br/>範囲のみ"] ==> RB["大規模変更や<br/>デバッグはPC"] ==> RC["役割分担が重要"]
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

## 3. 必要な前提

本資料では **Claude Code on the web** + **GitHub** の組み合わせを代表例として説明する (他の組み合わせは §4 参照)。

```mermaid
flowchart LR
    P1["📱 スマホ<br/>(ブラウザが動けばOK)"] ~~~ P2["🌐 ブラウザ<br/>Chrome/Safari/<br/>Edge等"]
    P2 ~~~ P3["🤖 Anthropic<br/>アカウント<br/>(Claude)"]
    P3 ~~~ P4["📦 GitHub<br/>アカウント"]
    P4 ~~~ P5["📁 対象リポジトリ<br/>(Public/Privateとも可)"]

    style P1 fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P3 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P5 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 専用アプリのインストールは**不要**。スマホ標準のブラウザだけで完結する。

> 📝 Claude には無料プランと有料プランがあり、Claude Code on the web の利用範囲・トークン上限はプランによって異なる。継続的に使う場合は有料プランの利用が想定される。

---

## 4. Claude Code on the web の位置づけ

本資料は **Claude Code on the web** (URL: `claude.ai/code`) を代表例として説明する。Anthropic 公式のブラウザ版で、Claude とのチャット形式の対話だけで GitHub リポジトリを読み・書き・コミット・push まで実行できる。

```mermaid
flowchart LR
    USER["📱 利用者"] ==>|"対話"| CLAUDE["🤖 Claude<br/>(claude.ai/code)"]
    CLAUDE ==>|"リポ操作"| GH["📦 GitHub"]
    GH -.->|"差分・URL返却"| CLAUDE
    CLAUDE -.->|"結果表示"| USER

    style USER fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style CLAUDE fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:3px
    style GH fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| 項目 | 内容 |
| --- | --- |
| **入口** | `claude.ai/code` (ブラウザでアクセス) |
| **必要なもの** | Anthropic アカウント + GitHub アカウント連携 |
| **対象 Git ホスト** | GitHub (公開・非公開の両方) |
| **できること** | リポジトリ読み込み・コード変更・ブランチ作成・コミット・push |
| **PR / マージ** | GitHub 側で別途実行 (本資料の範囲外) |

> 📝 **同種の選択肢**として、ChatGPT の Codex、GitHub Copilot Workspace、Gemini Code Assist などもある。本資料は Claude Code on the web を中心に書くが、考え方や注意点は他のサービスでも概ね適用できる。具体的な機能差は各サービスの公式ドキュメントを参照。

---

## 5. 典型的なワークフロー

スマホブラウザから `claude.ai/code` にアクセスし、対象 GitHub リポジトリに対して**壁打ち → 変更 → push**まで実行する流れ。

```mermaid
sequenceDiagram
    participant U as 📱 利用者
    participant C as 🤖 Claude<br/>(claude.ai/code)
    participant G as 📦 GitHub

    U->>C: ① ブラウザで claude.ai/code を開く
    Note over U,C: 初回は Anthropic ログイン<br/>+ GitHub 連携の承認
    U->>C: ② 対象リポジトリを選択
    C->>G: リポジトリの構造を読み込む
    G->>C: ファイル一覧・コード内容
    Note over U,C: ③ 壁打ち / 改善案検討
    U->>C: ④ 修正内容を依頼<br/>(言葉ベースで)
    C->>C: コード変更を生成
    C->>U: 変更内容のプレビュー
    Note over U,C: ⑤ 利用者がレビュー
    U->>C: 確認OK → 反映依頼
    C->>G: ⑥ 新規ブランチ作成<br/>+ コミット + push
    G->>C: push完了
    C->>U: 完了通知 (ブランチURL)
    Note over U: ⑦ 必要に応じて<br/>PR作成は GitHub 側で別途
```

> 📝 **ブランチ運用**は [git.md §3〜§4](../02_git/git.md) と同じ考え方。スマホからは **featureブランチに push まで** をスコープにし、main へのマージは PR 経由で別途行うのが安全。Claude Code on the web は**作業ごとにブランチを切る**動作が基本で、同じ対話内で複数の修正を依頼する場合も Claude 側でブランチを分けて push してくれることが多い (詳細挙動は公式ドキュメント参照)。

---

## 6. スマホで完結する範囲 / 完結しない範囲

```mermaid
flowchart LR
    subgraph M["📱 スマホで完結する範囲"]
        direction TB
        Y1["✅ 壁打ち / 設計議論"]
        Y2["✅ コード読み込み・理解"]
        Y3["✅ 軽微な修正・typo・refactor"]
        Y4["✅ コミット + push"]
        Y5["✅ ドキュメント更新"]
        Y1 ~~~ Y2 ~~~ Y3 ~~~ Y4 ~~~ Y5
    end

    subgraph P["💻 PC で行うことが多い範囲"]
        direction TB
        N1["⚠️ 多ファイル横断の大規模変更"]
        N2["⚠️ ローカルでの動作確認・デバッグ"]
        N3["⚠️ 大量のテスト実行"]
        N4["⚠️ コードレビュー (差分が大きい場合)"]
        N5["⚠️ マージ判断 / PRレビュー (大規模)"]
        N1 ~~~ N2 ~~~ N3 ~~~ N4 ~~~ N5
    end

    style M fill:#E8F8E8,stroke:#27AE60,stroke-width:3px,color:#000
    style P fill:#E5F1FB,stroke:#3498DB,stroke-width:3px,color:#000
    style Y1 fill:#fff,color:#000,stroke:#27AE60,stroke-width:2px
    style Y2 fill:#fff,color:#000,stroke:#27AE60,stroke-width:2px
    style Y3 fill:#fff,color:#000,stroke:#27AE60,stroke-width:2px
    style Y4 fill:#fff,color:#000,stroke:#27AE60,stroke-width:2px
    style Y5 fill:#fff,color:#000,stroke:#27AE60,stroke-width:2px
    style N1 fill:#fff,color:#000,stroke:#3498DB,stroke-width:2px
    style N2 fill:#fff,color:#000,stroke:#3498DB,stroke-width:2px
    style N3 fill:#fff,color:#000,stroke:#3498DB,stroke-width:2px
    style N4 fill:#fff,color:#000,stroke:#3498DB,stroke-width:2px
    style N5 fill:#fff,color:#000,stroke:#3498DB,stroke-width:2px
```

> 📝 「**スマホで何でもできる**」と思い込まないことが重要。AIアシスタントは指示通りにコードを生成するが、**生成結果の正しさ・本番影響の判断は利用者の責任**。狭い画面で多ファイル差分を読むのは認知負荷が高く、見落としが発生しやすい。

---

## 7. PC との使い分け

```mermaid
flowchart LR
    Q["🤔 どちらでやる？"] ==> Q1{"作業の規模"}

    Q1 -->|"小規模<br/>(1-2ファイル<br/>数十行)"| M["📱 <b>スマホ</b><br/>AIアシスタント<br/>+ クラウドGit"]
    Q1 -->|"大規模<br/>(多ファイル<br/>機能追加)"| P["💻 <b>PC</b><br/>エディタ + ローカル<br/>環境で検証"]

    M -.->|"動作確認が<br/>必要なら"| P
    P -.->|"後日の<br/>微修正なら"| M

    style Q fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style Q1 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style M fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| 観点 | スマホで足りる | PC が必要 |
| --- | --- | --- |
| 修正範囲 | 1〜数ファイル、数十行 | 機能追加・多ファイル横断 |
| 検証 | コードレビューだけで判断可能 | ローカル起動・動作確認が必要 |
| 思考の種類 | 壁打ち・発想出し・小修正 | 集中して書く・デバッグ |
| 作業時間 | 隙間時間 (移動中・休憩) | まとまった時間 (デスクで) |

> 📝 **「スマホでアイデア出し → 軽微な修正は push まで → 大規模実装はPCで完成」**という流れが、両者の強みを活かす一つのパターン。

---

## 8. 注意点とコツ

```mermaid
flowchart TB
    R["⚠️ リスクと対策"] --> R1["🔐 <b>秘密情報を含むリポは<br/>連携時に要注意</b><br/>(APIキー・本番接続文字列等)"]
    R --> R2["👁️ <b>AI生成コードは必ずレビュー</b><br/>意図と違う書き換えが発生しうる"]
    R --> R3["🌿 <b>main直 push は避ける</b><br/>featureブランチ経由が原則"]
    R --> R4["📏 <b>大規模変更は分割</b><br/>スマホでレビューできる範囲に"]
    R --> R5["⏱️ <b>セッション/利用量の上限</b><br/>Claude プランの上限内で運用"]
    R --> R6["🔑 <b>GitHub 連携の権限スコープ</b><br/>連携時の権限範囲を必要最小限に"]

    style R fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style R1 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style R2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style R3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style R4 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style R5 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style R6 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
```

> 📝 Claude Code on the web は対話ごとにトークンを消費し、プランによって**1日 / 1ヶ月の利用量上限**が定められている。スマホでの長時間連続作業は上限に当たりやすいため、**1セッション=1まとまった作業**として区切ると効率的。
>
> 🚨 **GitHub 連携時の権限スコープ**: 初回連携時に「すべてのリポジトリへのアクセス」と「特定のリポジトリのみ」が選択できる。**特定のリポジトリのみ**を選び、対象を明示的に限定するのが推奨。後から GitHub 設定 → Applications で範囲を変更可能。

### 秘密情報の取り扱い

```mermaid
flowchart LR
    R["🔐 機密性が高いもの"] ==>|"含まれている場合"| C["⚠️ リポ連携前に<br/>除外を検討"]
    C ==> S["✅ <b>.gitignore で除外</b><br/>(.env等)<br/><a href='../02_git/git.md'>git.md §8</a>"]
    C ==> S2["✅ <b>Secret/Vault に分離</b><br/>(GitHub Secrets/Key Vault等)<br/><a href='../04b_azure/azure.md'>azure.md §10</a>"]

    style R fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 🚨 リポジトリ自体に `.env` 等のシークレットファイルがコミットされていれば、AIアシスタント側からも閲覧可能になる。スマホ運用かどうかに関わらず、**秘密情報は事前にリポから除外**するのが原則 ([git.md §8](../02_git/git.md))。

---

## 9. 全体像の位置づけ

```mermaid
flowchart TB
    DEV["👨‍💻 開発者の作業環境"]

    DEV --> P["💻 <b>PC (デスクトップ)</b><br/>本格的なエディタ + ローカル検証"]
    DEV --> S["📱 <b>スマホ (本資料)</b><br/>AI×クラウドGitで軽量作業"]
    DEV --> T["💻 <b>タブレット</b><br/>中間的な位置 (両方の特徴)"]

    P -.->|"連動"| S
    S -.->|"連動"| P

    style DEV fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style P fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:3px
    style T fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 スマホ単体での完結を目指すのではなく、**PCワークフローを補完する位置づけ**として捉えるのが現実的。「PCのときだけ開発」から「移動中も思考と軽作業を進める」への拡張、と考えるとイメージしやすい。

---

## 10. 関連資料

- [git.md](../02_git/git.md) — ブランチ・push・PR の基本概念 (§3〜§4)
- [git.md §8 .gitignore](../02_git/git.md) — 秘密情報をリポに残さないための除外設定
- [azure.md §10](../04b_azure/azure.md) — 環境変数とシークレット管理
- [app-release.md](../01_app-release/app-release.md) — 全体のリリースフローでの位置づけ
