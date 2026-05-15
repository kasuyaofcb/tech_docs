# Git 全体像

> 📝 本資料の`main`ブランチは、古い呼称では`master`と同じものです。以降`main（master）`と表記します。

---

## 1. 全体の流れ

```mermaid
flowchart LR
    WD["💻<br/><b>作業ディレクトリ</b><br/>エディタで編集"]
    SA["📋<br/><b>ステージング</b><br/>コミット候補"]
    LR["📦<br/><b>ローカル履歴</b><br/>自分のPC内"]
    RR["☁️<br/><b>リモート</b><br/>GitHub"]

    WD ==>|"<b>git add</b><br/>候補に入れる"| SA
    SA ==>|"<b>git commit</b><br/>履歴に刻む"| LR
    LR ==>|"<b>git push</b><br/>みんなに共有"| RR
    RR -.->|"<b>git fetch</b><br/>リモート履歴を取得"| LR
    LR -.->|"<b>git merge</b><br/>自分の作業に統合"| WD
    RR ==>|"<b>git clone</b><br/>(初回のみ)"| WD

    style WD fill:#FF8C42,color:#FFFFFF,stroke:#333,stroke-width:3px
    style SA fill:#F4C430,color:#000000,stroke:#333,stroke-width:3px
    style LR fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:3px
    style RR fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:3px
```

左から右が基本の流れ。commitまでは自分のPCだけ。pushしてはじめて共有される。

> 📝 **`git pull` = `git fetch` + `git merge` のショートカット**
> 普段使うのは `git pull` ひとつでOK。「リモートから最新を取ってきて自分の作業に統合する」動作を一発でやってくれる。

---

## 2. よくある勘違い

```mermaid
flowchart TB
    subgraph WRONG["❌ 勘違い"]
        direction LR
        W1["commit した"] ==> W2["チームに共有された🎉"]
    end

    subgraph RIGHT["⭕ 実際"]
        direction LR
        R1["commit した"] ==> R2["自分のPCに記録<br/>📦"] ==> R3["push で初めて共有<br/>☁️"]
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
        WA["add した"] ==> WB["保存完了✅"]
    end

    subgraph R2["⭕ 実際"]
        direction LR
        RA["add した"] ==> RB["候補に入れただけ<br/>📋"] ==> RC["commit で履歴確定<br/>📦"]
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

## 3. ブランチとマージ

```mermaid
%%{init: { 'theme': 'base', 'themeVariables': {
  'git0': '#3498DB', 'git1': '#E67E22',
  'commitLabelColor': '#000', 'commitLabelBackground': '#fff',
  'tagLabelColor': '#000'
}}}%%
gitGraph
    commit id: "初期"
    commit id: "README"
    branch feature/login
    checkout feature/login
    commit id: "ログイン画面"
    commit id: "バリデーション"
    checkout main
    commit id: "誤字修正"
    merge feature/login id: "マージ"
    commit id: "リリース準備"
```

main（master）に影響を与えずに作業するため、機能ごとに枝分かれ（ブランチ）を作る。完成したらmain（master）に合流させる（merge）。

### 実際のコマンド順序

```mermaid
flowchart LR
    C1["<b>git checkout -b feature/login</b><br/>①ブランチ作成+移動"] ==> C2["編集 → add → commit<br/>②feature/loginで作業"]
    C2 ==> C3["<b>git checkout main</b><br/>③mainに戻る"]
    C3 ==> C4["<b>git merge feature/login</b><br/>④mainに合流"]
    C4 ==> C5["<b>git branch -d feature/login</b><br/>⑤用済みを削除"]

    style C1 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C3 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C4 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C5 fill:#7F8C8D,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 §1で見せた「全体の流れ」は4つの**場所**をまたぐコマンド。こちらは`Local Repository`の**内部**でのブランチ操作。

---

## 4. Pull Request フロー

```mermaid
flowchart TB
    A["① ブランチ作成<br/>git checkout -b"] ==> B["② 編集 → add → commit"]
    B ==> C["③ git push"]
    C ==> D["④ GitHub で<br/>PR を作成"]
    D ==> E{"⑤ レビュー"}
    E ==>|"修正依頼"| F["⑥ 追加 commit → push"]
    F ==> E
    E ==>|"Approve"| G["⑦ main（master）に Merge"]
    G ==> H["⑧ ブランチ削除"]

    style A fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style C fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style D fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style E fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style G fill:#2980B9,color:#FFFFFF,stroke:#333,stroke-width:2px
    style H fill:#7F8C8D,color:#FFFFFF,stroke:#333,stroke-width:2px
```

main（master）に直接commitしない。必ずブランチ→PR→レビュー→mergeの流れ。

---

## 5. 迷ったら叩くコマンド

```mermaid
flowchart LR
    Q["❓ 今どこ？<br/>何が起きてる？"] ==> S["<b>git status</b><br/>📋 変更の一覧"]
    Q ==> D["<b>git diff</b><br/>🔍 中身の差分"]
    Q ==> B["<b>git branch</b><br/>🌿 今のブランチ"]
    Q ==> L["<b>git log --oneline</b><br/>📜 直近の履歴"]

    style Q fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style S fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style D fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style L fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

---

## 6. コンフリクト（衝突）の解決

2人が**同じファイルの同じ行**を別々に変えると、Gitは「どっちを残すか決めて」と聞いてくる。これがコンフリクト。

```mermaid
gitGraph
    commit id: "共通の出発点"
    branch feature
    checkout feature
    commit id: "Aさんが3行目を変更"
    checkout main
    commit id: "Bさんが3行目を変更"
    merge feature type: REVERSE id: "⚠️ 衝突"
```

mergeしようとすると、ファイルの中に**マーカー**が差し込まれる:

```mermaid
flowchart TB
    F["📄 衝突したファイル"] --> M["<<<<<<< HEAD<br/>Bさんの変更（main側）<br/>=======<br/>Aさんの変更（feature側）<br/>>>>>>>> feature"]

    style F fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style M fill:#FFE5E5,color:#000,stroke:#C0392B,stroke-width:2px
```

**解決の3ステップ:**

```mermaid
flowchart LR
    S1["① どちらを残すか<br/>（または両方）<br/>人間が判断"] ==> S2["② マーカーを削除<br/><<<<<<< や ======= を消す"]
    S2 ==> S3["③ git add → git commit<br/>解決完了"]

    style S1 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

コンフリクトはエラーではなく、**人間に判断を求めているだけ**。怖くない。

---

## 7. やらかした時の戻し方

「どこまで進んだか」で使うコマンドが変わる。

```mermaid
flowchart TB
    Q["😱 やり直したい"] ==> A{"どこまで進んだ？"}
    A ==>|"① 編集しただけ"| R1["<b>git restore ファイル名</b><br/>変更を破棄して元に戻す"]
    A ==>|"② git add した"| R2["<b>git restore --staged ファイル名</b><br/>ステージから下ろす"]
    A ==>|"③ commit した<br/>（未push）"| R3["<b>git reset --soft HEAD^</b><br/>commitだけ取り消す<br/>変更内容は残る"]
    A ==>|"④ push した"| R4["<b>git revert コミットID</b><br/>打ち消しcommitを追加<br/>履歴は消さない"]

    style Q fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style A fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style R1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R3 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style R4 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
```

**鉄則:** pushした後は`reset`ではなく`revert`。履歴を書き換えるとチーム全員が死ぬ。

> 📝 補足: `HEAD^`は「今のcommitの1つ前」を指す書き方。`HEAD~2`なら「2つ前」。

---

## 8. .gitignore — 追跡しないファイル

`.gitignore`に書いたファイルは、Gitから見えなくなる（commit対象外）。

```mermaid
flowchart LR
    F1["📄 main.py"] ==> G{".gitignore<br/>フィルタ"}
    F2["📄 README.md"] ==> G
    F3["🔐 .env<br/>（APIキー）"] ==> G
    F4["📦 node_modules/"] ==> G
    F5["🗂 .DS_Store"] ==> G

    G ==>|"追跡する"| OK["✅ commit可能"]
    G ==>|"無視する"| NG["❌ 見えない"]

    F1 -.-> OK
    F2 -.-> OK
    F3 -.-> NG
    F4 -.-> NG
    F5 -.-> NG

    style G fill:#F4C430,color:#000,stroke:#333,stroke-width:3px
    style OK fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style NG fill:#C0392B,color:#FFFFFF,stroke:#333,stroke-width:2px
    style F3 fill:#FFE5E5,color:#000,stroke:#C0392B
    style F4 fill:#FFE5E5,color:#000,stroke:#C0392B
    style F5 fill:#FFE5E5,color:#000,stroke:#C0392B
```

**絶対にcommitしてはいけないもの:**

- 🔐 `.env` / APIキー / パスワード → 漏洩事故の常連
- 📦 `node_modules/` / `venv/` → 巨大で再生成可能
- 🗂 `.DS_Store` / エディター設定 → OS/個人依存のゴミ

このリポジトリの[.gitignore](../../.gitignore)が実例なので見てみよう。

### ⚠️ よくある罠: 「すでに追跡中のファイルは無視できない」

`.gitignore`は**新しく追加されるファイル**にしか効かない。一度commitしてしまったファイルは、後から`.gitignore`に書いても無視されない。

```mermaid
flowchart LR
    S1["😱 .envをcommit<br/>しちゃった"] ==> S2["❌ .gitignoreに<br/>追加しただけでは効かない"]
    S2 ==> S3["✅ <b>git rm --cached .env</b><br/>で追跡から外す"]
    S3 ==> S4["✅ commit & push<br/>今後は無視される"]

    style S1 fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 🚨 APIキーなどを一度pushしてしまった場合、`git rm --cached`だけでは**過去のコミット履歴には残る**。外部に漏れた可能性があるなら、**そのキーは無効化して再発行**するのが鉄則。

---

## 9. git stash（作業中の変更を一時退避）

**stash**は、作業ディレクトリにある**まだコミットしていない変更**を、いったん**別の場所に退避**する仕組みです。別ブランチに切り替えたいが変更を捨てたくない、急ぎの修正に移りたい、などのときに使います（コミットではないので、履歴には残りません）。

```mermaid
flowchart LR
    WD["作業ディレクトリ<br/>未コミットの変更"] ==>|"git stash"| ST["stash のスタック"]
    ST ==>|"git stash pop"| WD2["変更を戻す"]

    style WD fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style ST fill:#F4C430,color:#000000,stroke:#333,stroke-width:2px
    style WD2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 イメージは「**机の上の作業を、一旦引き出しにしまう**」感覚。あとで引き出しから取り出して続きを再開できる。

### こんなときに使う（具体例）

#### 例1: 作業中に緊急バグの修正依頼が来た

`feature/login`で開発中、まだコミットできるキリのいい状態じゃない。でも本番でバグが見つかり、急いで`main`で修正したい、という王道シナリオ。

```mermaid
sequenceDiagram
    participant U as 自分
    participant F as feature/login
    participant M as main
    U->>F: 編集中...（未コミット）
    Note over U,F: 🚨 緊急バグ報告！
    U->>F: git stash push -m "ログイン画面 途中"
    U->>M: git checkout main
    U->>M: バグ修正 → add → commit → push
    U->>F: git checkout feature/login
    U->>F: git stash pop（作業再開）
```

> 💡 commitしてからブランチを移ることもできるが、**「中途半端な状態を履歴に残したくない」**ときにstashが便利。

#### 例2: ブランチを間違えて作業していた

`main`で作業を始めてしまい、コミット直前に「これ`feature`ブランチでやるやつだ…」と気づいたとき。

```mermaid
flowchart LR
    S1["😱 main で<br/>編集してた"] ==> S2["<b>git stash</b><br/>変更を退避"]
    S2 ==> S3["<b>git checkout -b feature/xxx</b><br/>正しいブランチを切る"]
    S3 ==> S4["<b>git stash pop</b><br/>変更を呼び戻す"]
    S4 ==> S5["✅ feature/xxx で<br/>add → commit"]

    style S1 fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S3 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S4 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S5 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

#### 例3: `git pull` したら「変更を確定してくれ」と怒られた

リモートに新しいcommitが入っていて取り込みたいが、ローカルに未コミットの変更があるとpullが拒否されることがある。

```mermaid
flowchart LR
    P1["<b>git pull</b><br/>❌ 失敗"] ==> P2["<b>git stash</b><br/>変更を一旦避ける"]
    P2 ==> P3["<b>git pull</b><br/>✅ 取り込み成功"]
    P3 ==> P4["<b>git stash pop</b><br/>変更を戻す"]

    style P1 fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style P3 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style P4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

#### 例4: 「この変更を一時的に外した状態」で挙動を確認したい

書きかけのコードを残したまま、**変更前の状態でちゃんと動くか確認したい**ときも便利。

```mermaid
flowchart LR
    E1["変更を加えてみた<br/>でも本当に必要？"] ==> E2["<b>git stash</b><br/>一旦外して<br/>変更前の状態に"]
    E2 ==> E3["動作確認<br/>挙動を比較"]
    E3 ==> E4["<b>git stash pop</b><br/>変更を戻す"]

    style E1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style E2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style E3 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style E4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

### よく使うコマンド

| コマンド | 説明 |
|---------|------|
| `git stash` | 追跡中ファイルの変更を退避（メッセージは自動） |
| `git stash push -m "メモ"` | メモ付きで退避 |
| `git stash push -u` | **未追跡ファイルも含めて**退避（新規ファイルも一緒に避けたいとき） |
| `git stash list` | 退避一覧を表示 |
| `git stash pop` | いちばん新しい stash を**適用してから削除** |
| `git stash apply` | 適用するが**stash は残す**（同じ内容を複数ブランチに当てたいとき） |
| `git stash drop stash@{n}` | 指定した stash だけ削除 |
| `git stash clear` | stash を**すべて削除**（取り消しにくいので注意） |

### 注意点

- `pop` したブランチによっては**コンフリクト**が出ることがある（退避時と違うコードが入っている場合）。
- stash はローカル向けの退避であり、**`git push` ではリモートに送られない**（チーム共有の仕組みではない）。

---

## 10. 補足: `main` と `master` の違い

```mermaid
flowchart LR
    M["<b>master</b><br/>Git 元々のデフォルト名"] ==>|"2020年〜<br/>名称変更の流れ"| N["<b>main</b><br/>現在のデフォルト名"]

    style M fill:#95A5A6,color:#FFFFFF,stroke:#333,stroke-width:2px
    style N fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

**結論: 技術的には完全に同じもの。名前が違うだけ。**

- Gitのデフォルトブランチ名は元々`master`
- 2020年頃、用語への配慮から業界的に`main`への変更が広がった
- GitHubは2020年10月以降、新規リポジトリのデフォルトが`main`
- 古いチュートリアルでは`master`、最近のものは`main`で書かれている

### 設定方法

```mermaid
flowchart TB
    A["🆕 これから作るリポジトリ"] ==> A1["<b>git config --global init.defaultBranch main</b><br/>以降 git init すると main で始まる"]

    B["🔄 既存の master リポジトリを main に変えたい"] ==> B1["<b>git branch -m master main</b><br/>ローカルのブランチ名変更"]
    B1 ==> B2["<b>git push -u origin main</b><br/>リモートに main を push"]
    B2 ==> B3["GitHub画面で<br/>Default branch を main に変更<br/>(Settings → Branches)"]
    B3 ==> B4["<b>git push origin --delete master</b><br/>リモートの master を削除"]

    style A fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style A1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B fill:#E67E22,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B1 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B2 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style B3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style B4 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
```

> 📝 現在自分の環境で何がデフォルトか確認: `git config --global init.defaultBranch`
