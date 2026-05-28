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

```mermaid
flowchart LR
    P1["📱 スマホ<br/>(ブラウザが動けばOK)"] ~~~ P2["🌐 ブラウザ<br/>Chrome/Safari/<br/>Edge等"]
    P2 ~~~ P3["🤖 AIアシスタント<br/>のアカウント"]
    P3 ~~~ P4["📦 クラウドGit<br/>のアカウント"]
    P4 ~~~ P5["📁 対象リポジトリ<br/>(Public/Privateとも可)"]

    style P1 fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P3 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P5 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 専用アプリのインストールは**不要**。スマホ標準のブラウザだけで完結する。

---

## 4. AIアシスタント側の選択肢

リポジトリ連携機能を持つ Web 版 AI アシスタントは複数存在し、それぞれ名称や対応 Git ホストが異なる。

```mermaid
flowchart TB
    Q["🤖 どのAI<br/>アシスタント？"] ==> Q1{"優先したいもの"}

    Q1 --> C1["🟪 <b>Claude (Anthropic)</b><br/>claude.ai/code<br/>= Claude Code on the web"]
    Q1 --> C2["🟦 <b>ChatGPT (OpenAI)</b><br/>Codex<br/>(有料プラン対象)"]
    Q1 --> C3["🟪 <b>GitHub Copilot<br/>Workspace</b><br/>(GitHub標準連携)"]
    Q1 --> C4["🟧 <b>Gemini (Google)</b><br/>等の他選択肢"]

    style Q fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style Q1 fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style C1 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C2 fill:#0078D4,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C3 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C4 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
```

| サービス | URL / 入口 | 特徴 |
| --- | --- | --- |
| **Claude Code on the web** | `claude.ai/code` | Anthropic公式。GitHubリポを直接対象にブランチ作成・編集・push まで実行 |
| **ChatGPT Codex** | `chatgpt.com` の Codex 機能 | OpenAI公式。GitHub連携でリポジトリを sandbox にロード、編集・PR作成 |
| **GitHub Copilot Workspace** | GitHub 内蔵 | Issue/PR から起動する GitHub の AI 機能 |
| **その他 (Gemini Code Assist 等)** | 各社サービス | 機能仕様は各サービスのドキュメント参照 |

> 📝 サービスの**機能名や対応範囲は頻繁に更新**される。本資料では「**この種のサービスが選択肢として存在する**」という事実を示すに留め、具体的な機能比較は各サービスの公式ドキュメントを参照する。

---

## 5. 典型的なワークフロー

スマホブラウザから AI アシスタントを使い、ブランチ作成 → 編集 → push まで実行する流れ。

```mermaid
sequenceDiagram
    participant U as 📱 利用者
    participant A as 🤖 AIアシスタント
    participant G as 📦 クラウドGit

    U->>A: ① 対象リポジトリを指定
    A->>G: リポジトリの構造を読み込む
    G->>A: ファイル一覧・コード内容
    Note over U,A: ② 壁打ち / 改善案検討
    U->>A: ③ 修正内容を依頼<br/>(言葉ベースで)
    A->>A: コード変更を生成
    A->>U: 変更内容のプレビュー
    Note over U,A: ④ 利用者がレビュー
    U->>A: 確認OK → 反映依頼
    A->>G: ⑤ 新規ブランチ作成<br/>+ コミット + push
    G->>A: push完了
    A->>U: 完了通知 (ブランチURL)
    Note over U: ⑥ 必要に応じて<br/>PR作成は別途
```

> 📝 **ブランチ運用**は [git.md §3〜§4](../02_git/git.md) と同じ考え方。スマホからは **featureブランチに push まで** をスコープにし、main へのマージは PR 経由で別途行うのが安全。

---

## 6. スマホで完結する範囲 / 完結しない範囲

```mermaid
flowchart TB
    SCOPE["📱 スマホで完結する範囲"]

    SCOPE --> Y1["✅ 壁打ち / 設計議論"]
    SCOPE --> Y2["✅ コード読み込み・理解"]
    SCOPE --> Y3["✅ 軽微な修正・typo・refactor"]
    SCOPE --> Y4["✅ コミット + push"]
    SCOPE --> Y5["✅ ドキュメント更新"]

    OUT["💻 PC で行うことが多い範囲"]

    OUT --> N1["⚠️ 多ファイル横断の大規模変更"]
    OUT --> N2["⚠️ ローカルでの動作確認・デバッグ"]
    OUT --> N3["⚠️ 大量のテスト実行"]
    OUT --> N4["⚠️ コードレビュー (差分が大きい場合)"]
    OUT --> N5["⚠️ マージ判断 / PRレビュー (大規模)"]

    style SCOPE fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
    style OUT fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:3px
    style Y1 fill:#fff,color:#000,stroke:#27AE60
    style Y2 fill:#fff,color:#000,stroke:#27AE60
    style Y3 fill:#fff,color:#000,stroke:#27AE60
    style Y4 fill:#fff,color:#000,stroke:#27AE60
    style Y5 fill:#fff,color:#000,stroke:#27AE60
    style N1 fill:#fff,color:#000,stroke:#3498DB
    style N2 fill:#fff,color:#000,stroke:#3498DB
    style N3 fill:#fff,color:#000,stroke:#3498DB
    style N4 fill:#fff,color:#000,stroke:#3498DB
    style N5 fill:#fff,color:#000,stroke:#3498DB
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
    R --> R5["⏱️ <b>会話セッションの上限</b><br/>長時間連続作業はトークン消費が大きい"]

    style R fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style R1 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style R2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style R3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style R4 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style R5 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
```

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
