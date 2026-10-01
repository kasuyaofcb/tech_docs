# Git / GitHub 環境構築手順 — インストールから初めての push まで

> 📝 [git.md](git.md) が**概念の地図**なら、このドキュメントは**実際に手を動かす順番**。**Part 1（Git）だけで「壊したコードを戻せる」状態になる。** GitHub は必要になってから Part 2 を開けばよい。

🧠 想定する到達点:

- `git --version` でバージョンが表示される
- 自分のフォルダで `git init` → `git commit` ができる
- 間違えたとき `git restore` で直前の状態に戻せる
- （Part 2）GitHub にリポジトリを作って `git push` できる

---

## 0. 全体マップ

### 🟢 Part 1 — Git（30分）

```mermaid
flowchart LR
    S1["1.1<br/>入っているか<br/>確認"] ==> S2["1.2<br/>インストール<br/>「無い場合だけ」"]
    S2 ==> S3["1.3<br/>初期設定<br/>名前・メール"]
    S3 ==> S4["1.4<br/>.gitignore<br/>+ 最初のコミット"]
    S4 ==> S5["1.5<br/>戻し方"]
    S5 ==> G1["🎉<br/><b>壊しても<br/>戻せる</b>"]

    style S1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style G1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

> ⬇️ **ここで一度止まってよい。** 「人に見せたい / 別のPCから触りたい / PCが壊れても残したい」が出てきたら、下へ進む。

### 🟡 Part 2 — GitHub（15分）

```mermaid
flowchart LR
    T1["2.1<br/>アカウント"] ==> T2["2.2<br/>認証<br/>gh auth login"]
    T2 ==> T3["2.3<br/>リポジトリ作成<br/>+ push"]
    T3 ==> G2["🎉<br/><b>公開・共有<br/>できる</b>"]

    style T1 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style T2 fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style T3 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style G2 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

| Part | 内容 | 所要時間 | いつやる |
| --- | --- | --- | --- |
| 1 | Git のインストール + 初期設定 + 最初のコミット | 30分 | **最初に** |
| 2 | GitHub アカウント + 認証 + 初めての push | 15分 | 人に見せたくなったとき |

> 📝 **Part 1 と Part 2 は切り離してよい。** Part 1 だけでバージョン管理は成立する（自分のPCの中に履歴が残る）。GitHub は「人に見せる / 別のPCから触る / PCが壊れても残す」ための箱で、**無くても作業は進む**。

---

# Part 1: Git

ここだけで「壊しても戻せる」状態になる。

## 1.0 ターミナルを開く — コマンドはすべてここに打つ

🚨 **この手順書に出てくるコマンドは、すべて Cursor のターミナルに打つ。** 別のアプリを開く必要はない。

```mermaid
flowchart LR
    S1["①<br/>Cursor を開く"] ==> S2["②<br/>メニューの<br/>ターミナル →<br/>新しいターミナル"]
    S2 ==> S3["③<br/>画面の下半分に<br/>パネルが開く"]
    S3 ==> S4["④<br/>カーソルが<br/>点滅する位置に<br/>貼る"]

    style S1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S3 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S4 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

| | やること |
| --- | --- |
| 開き方 | Cursor 上部メニューの **ターミナル** → **新しいターミナル** |
| ショートカット | `Ctrl` + `` ` ``（バッククォート） |
| 開いた合図 | 画面の下半分に `TERMINAL` のパネルが出て、カーソルが点滅する |
| 打つ場所 | その**カーソルの位置**。貼って `Enter` |

![Cursor のターミナルを開いた状態](images/cursor-terminal.png)

**この画面の見方:**

| 場所 | 何が映っているか |
| --- | --- |
| 画面の下半分 | `TERMINAL` タブ。**ここが打つ場所** |
| `MacBook-Pro-2:test ...$` の行 | プロンプト。`$` の右でカーソルが点滅している |
| `$` の右の四角 | カーソル。**ここに貼って `Enter`** |

> 📝 **行末の記号は環境で違う。** `$`（この画面）／`%`（zsh）／`>`（Windows の PowerShell）。**どれでもよい。カーソルが点滅していれば打てる状態。**

> 📝 **上のほうに英語のメッセージが出ていても気にしない。** この画面の `The default interactive shell is now zsh.` のように、ターミナルは起動時にお知らせを出すことがある。エラーではない。

> 📝 **バッククォートの場所**: 日本語キーボードは `Shift` + `@`。英語キーボードは `Esc` の下。
> 📝 **Windows でも同じ。** Cursor のターミナルは PowerShell が開く。

### コマンドの打ち方 — 4つのルール

| ルール | なぜ |
| --- | --- |
| 🟢 **コードブロックの右上のコピーボタンで貼る** | 手で打つと必ずどこか間違える。GitHub 上ではブロックにマウスを乗せると右上にボタンが出る |
| 🚨 **1行コピーしたら 1行 Enter。まとめて貼らない** | 途中で失敗したとき、どこまで進んだかが分かる |
| 🚨 **`#` で始まる行は打たない** | 説明文。打っても何も起きないが、混乱する |
| 🚨 **`<` `>` で囲まれた部分は自分の値に置き換える** | 例: `git restore <ファイル名>` → `git restore index.html` |

> 🚨 **何も表示されなくても、たいてい成功している。** Git は成功したとき黙っているコマンドが多い。**エラーが出ていなければ次へ進む。**

---

## 1.1 Git がすでに入っているか確認

1.0 で開いたターミナルに、これを貼って Enter。**入っていればインストールは丸ごと不要。**

```sh
git --version
```

| 出た結果 | 次にやること |
| --- | --- |
| `git version 2.39.3` のようにバージョンが出た | **1.3 へ飛ぶ。インストールは不要** |
| `command not found: git` | 1.2 でインストール |
| 「コマンドライン・デベロッパ・ツール」のダイアログが出た（Mac） | 「インストール」を押す。終わったらもう一度 `git --version`。これで入れば 1.3 へ |

> 📝 **Mac には最初から Git が入っていることが多い。** Xcode Command Line Tools に同梱されているため。バージョンが少し古いことはあるが、学習やふつうの開発には問題ない。

---

## 1.2 インストール（入っていなかった場合だけ）

### Mac — Homebrew 経由

```mermaid
flowchart LR
    S1["①<br/>Homebrew<br/>インストール"] ==> S2["②<br/>🔑 Macのパスワード<br/>入力"]
    S2 ==> S3["③<br/>⚠️ 画面に出た<br/>Next steps を実行"]
    S3 ==> S4["④<br/>brew -v<br/>で確認"]
    S4 ==> S5["⑤<br/>brew install git"]

    style S1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S3 fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style S4 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

**操作の要点:**

| ステップ | やること | 注意 |
| --- | --- | --- |
| ① | 下のコマンドを1行まるごと貼って Enter | **5〜10分かかる。** 文字が流れ続けるが、待っていればよい |
| ② | Mac のログインパスワードを入力 | 🚨 **打っても画面には1文字も出ない。** そのまま Enter |
| ③ | 最後に出た `==> Next steps:` の行を実行 | 🚨 **ここだけは自分の画面を見る。** 機種で中身が変わる |
| ④ | `brew -v` | `Homebrew 4.x.x` と出れば成功 |
| ⑤ | `brew install git` → `git --version` | |

**① Homebrew のインストール:**

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

> 📝 **Homebrew とは**: Mac に開発ツールを入れるための道具。`brew install 〇〇` でツールが入る。この先 `gh`（GitHub CLI）など何度か使う。

> 🚨 **③が最大の詰まりどころ。** インストールが終わると、画面の最後にこう出る。

```text
==> Next steps:
- Run these commands in your terminal to add Homebrew to your PATH:
    echo >> /Users/あなたの名前/.zprofile
    echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/あなたの名前/.zprofile
    eval "$(/opt/homebrew/bin/brew shellenv)"
```

この**インデントされた行だけ**を選んでコピーし、ターミナルに貼って Enter。

⚠️ **この手順書に書いてある行ではなく、自分の画面に出た行を使う。** Apple シリコンは `/opt/homebrew`、Intel Mac は `/usr/local` と、パスが変わる。

> 🚨 **`brew: command not found` が出たら**: ③が終わっていない。上の行を貼り直す。

> 📝 **なぜ2種類の行があるのか**: `echo ... >> ~/.zprofile` は「**次にターミナルを開いたとき**のための設定ファイルへの追記」、`eval "$(...)"` は「**いま開いているターミナル**への即時反映」。役割が違うので両方必要。片方だけだと、閉じたら消える／今は使えない、のどちらかになる。

> 🚨 **それでも `brew` が見つからないときは、ターミナルを開き直す。** Cursor のターミナルパネル右上の 🗑 でいまのターミナルを閉じ、`Ctrl` + `` ` `` でもう一度開く。`.zprofile` が読み直されて `brew` が使えるようになる。

### Windows — 公式インストーラー

```mermaid
flowchart LR
    S1["①<br/>git-scm.com から<br/>exe をダウンロード"] ==> S2["②<br/>ダブルクリック"]
    S2 ==> S3["③<br/>PATH の設定<br/>(真ん中を選ぶ)"]
    S3 ==> S4["④<br/>他はデフォルトで<br/>Next → Install"]
    S4 ==> S5["⑤<br/>PowerShellで<br/>git --version"]

    style S1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#E74C3C,color:#FFFFFF,stroke:#333,stroke-width:3px
    style S4 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S5 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

**選択肢が出る画面だけ、選ぶものを決めておく:**

| 画面 | 選ぶもの | なぜ |
| --- | --- | --- |
| Adjusting your PATH environment | **Git from the command line and also from 3rd-party software**（真ん中） | PowerShell でも VSCode でも Git が使える |
| Choosing HTTPS transport backend | **Use the native Windows Secure Channel library** | Windows の証明書をそのまま使える |
| Configuring the line ending conversions | **Checkout Windows-style, commit Unix-style**（既定） | 1.3 の `core.autocrlf` と同じ効果 |
| それ以外 | すべて既定のまま Next | 変える理由がない |

> 🚨 **インストールが終わったら Cursor を再起動する。** Windows は起動中のアプリに PATH の変更が届かないため、再起動しないと Cursor のターミナルで `git --version` が `command not found` のままになる。

> 📝 Git for Windows には **Git Credential Manager** が同梱されている。Part 2 の認証が、初回 push のときにブラウザが開いて終わる。

---

## 1.3 初期設定（2つだけ）

コミットに「誰がやったか」を刻むための設定。🚨 **これをしないとコミットできない。**

**1行ずつ**貼って、`<>` の部分を自分の値に置き換えてから Enter。

```sh
git config --global user.name "<あなたの名前>"
```

```sh
git config --global user.email "<あなたのメールアドレス>"
```

→ 例: `git config --global user.name "Taro Yamada"`

確認:

```sh
git config --global --list
```

| 項目 | 何に使われるか | 注意 |
| --- | --- | --- |
| `user.name` | コミット履歴に出る名前 | 本名でもハンドルネームでもよい |
| `user.email` | コミット履歴に出るメール | 🚨 Part 2 で GitHub を使うなら、**GitHub に登録したメールと同じにする** |

> 🚨 **メールアドレスは公開される。** GitHub に上げると、コミット履歴から誰でも見える。隠したい場合は GitHub の `noreply` アドレス（`12345678+username@users.noreply.github.com`）を使う。GitHub の Settings → Emails → **Keep my email addresses private** で確認できる。

> 🚨 **ここが違うと、自分のコミットとして数えられない。** GitHub の自分のページに出る活動記録（緑のマス目）が増えず、コミットの横に自分のアイコンも出ない。あとから直すのは面倒なので、Part 2 をやる予定があるなら最初から揃えておく。

**改行コードの設定（Windows のみ）:**

```sh
git config --global core.autocrlf true
```

> 📝 Windows と Mac / Linux で改行の文字が違う。これを入れておくと「**1文字も直していないのに全行が変更扱いになる**」が起きにくい。

---

## 1.4 最初のコミット

```mermaid
flowchart LR
    S1["①<br/>Cursorで<br/>フォルダを開く"] ==> S2["②<br/>.gitignore<br/>を作る"]
    S2 ==> S3["③<br/>git init<br/>履歴を取る対象にする"]
    S3 ==> S4["④<br/>git add .<br/>記録する候補に入れる"]
    S4 ==> S5["⑤<br/>git commit<br/>セーブする"]

    style S1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

### ① フォルダを開く — `cd` は打たない

🚨 **Cursor で「作業したいフォルダ」を開いていれば、ターミナルは最初からそのフォルダにいる。** 移動のコマンドは要らない。

| | やること |
| --- | --- |
| 開き方 | Cursor 上部メニューの **ファイル** → **フォルダを開く** → 自分のプロジェクトのフォルダを選ぶ |
| 開いたあと | ターミナルを開き直す（`Ctrl` + `` ` ``） |

いま自分がどこにいるかを確認する:

```sh
pwd
```

→ **自分のプロジェクトのフォルダ名で終わっていればOK。**

> 🚨 **ここが違うと、関係ないフォルダの履歴を取り始める。** `pwd` の結果が `/Users/あなたの名前` で終わっていたら、フォルダを開けていない。開き方からやり直す。

### ② `.gitignore` を作る — `git add` より先に

🚨 **これを先に作らないと、記録してはいけないファイルまで記録される。**

Cursor の左のファイル一覧で右クリック → **新しいファイル** → `.gitignore` という名前で作り、中にこれを貼る。

```text
.env
.env.local
node_modules/
.DS_Store
```

| 書いたもの | なぜ除外するか |
| --- | --- |
| `.env` / `.env.local` | 🔴 **APIキーやパスワードが入っている。** 一度記録すると履歴に残り続け、あとから消すのは非常に面倒 |
| `node_modules/` | ライブラリの置き場。数万ファイルあり、記録する意味がない（`npm install` で作り直せる） |
| `.DS_Store` | Mac が勝手に作る管理ファイル |

> 🚨 **一度 `commit` したファイルは、あとから `.gitignore` に書いても記録され続ける。** だから `git add` より先に作る。

### ③〜⑤ 履歴を取り始める

**1行ずつ**貼って Enter。

```sh
git init
```

何が記録されようとしているか、先に見ておく:

```sh
git status
```

> 🚨 **ここに `.env` が出てきたら、② をやり直す。** `.gitignore` に書けていない。

```sh
git add .
```

```sh
git commit -m "最初のコミット"
```

確認:

```sh
git log --oneline
```

→ `a1b2c3d 最初のコミット` のように1行出れば成功。

| コマンド | 意味 |
| --- | --- |
| `git init` | このフォルダを「履歴を取る対象」にする（隠しフォルダ `.git` ができる） |
| `git status` | いまの状態を見る。**迷ったらこれ** |
| `git add .` | 今の状態を「記録する候補」に入れる |
| `git commit -m "..."` | セーブする |
| `git log --oneline` | セーブ履歴を1行ずつ見る |

> 📝 `.git` を消すと履歴も消える。**フォルダをコピーするときは `.git` ごとコピー**する。

> 📝 作業ディレクトリ → ステージング → ローカルリポジトリ、という3段構えの意味は [git.md](git.md) の「全体の流れ」を読む。

---

## 1.5 戻し方（Part 1 の目的はここ）

**コードを大きく書き換える前に `commit` しておけば、壊れても戻せる。** これが Part 1 をやる理由。

**まず、何が変わったかを見る:**

```sh
git diff
```

**1ファイルだけ戻す:**

```sh
git restore <ファイル名>
```

→ `<ファイル名>` は自分の値に置き換える。例: `git restore index.html`

**編集を全部捨てて戻す:**

```sh
git restore .
```

> 🚨 **`git restore` は編集を消す。取り消せない。** 残したい変更があるなら、先に `git commit` する。**迷ったら必ず `git diff` で中身を見てから。**

> 📝 **AI にコードを書き換えてもらうなら、指示を出す前に1回 `commit` しておくのがいちばん安い保険。** 気に入らなければ `git restore .` で指示前に戻る。

> 📝 もっと踏み込んだ戻し方（コミット済みを取り消す / 直前のコミットを直す）は [git.md](git.md) の「やらかした時の戻し方」にある。

---

### ✅ Part 1 のチェック

- [ ] `git --version` でバージョンが出る
- [ ] `git config --global --list` に `user.name` と `user.email` がある
- [ ] `pwd` が自分のプロジェクトのフォルダを指している
- [ ] `git status` に `.env` が出てこない
- [ ] `git log --oneline` にコミットが1つ以上出る

**最後に、戻せることを1回だけ試す:**

1. 適当なファイルを開いて、**消えても困らない1行**（コメントなど）を足して保存する
2. `git diff` → その1行が出る
3. `git restore .` → 足した1行が消えて、元に戻る

> 🚨 **この練習は、保存していない大事な編集が無い状態でやる。** `git restore .` は、まだ `commit` していない編集を**すべて**消す。

**ここまでできたら、GitHub は後回しでよい。**

---

# Part 2: GitHub

> 📝 **必要になってから開く。** 「コードを人に見せたい」「別のPCから触りたい」「PCが壊れても残したい」のどれかが出てきたタイミング。

## 2.1 アカウントを作る

| ステップ | やること | 注意 |
| --- | --- | --- |
| ① | [github.com](https://github.com) で Sign up | 画面の案内どおりで迷わない |
| ② | メールアドレスを認証 | 🚨 **1.3 の `user.email` と同じメールにする**（違う場合は Settings → Emails で追加登録する） |
| ③ | 2要素認証（2FA）を設定 | 必須。下を読む |

### ③ 2要素認証（2FA）— ここだけ丁寧に

🚨 **GitHub は 2FA が必須。** 設定しないまま使い続けると、アカウントが制限される。

| ステップ | やること |
| --- | --- |
| 1 | スマホに認証アプリを入れる（**Google Authenticator** / **Microsoft Authenticator** / 1Password など） |
| 2 | GitHub の **Settings** → **Password and authentication** → **Enable two-factor authentication** |
| 3 | 画面に出た QRコードを、認証アプリで読み取る |
| 4 | アプリに出た6桁の数字を GitHub に入力 |
| 5 | 🔴 **リカバリーコード（16個の文字列）が表示される。必ずダウンロードして保存する** |

> 🚨 **5を飛ばさない。** スマホを機種変更したり失くしたりすると、**リカバリーコードが無い限りアカウントに二度と入れなくなる。** パスワードマネージャーに入れるか、印刷して手元に置く。

> 📝 SMS でも設定できるが、認証アプリのほうが安全で、電波が無くても使える。

---

## 2.2 認証（ここが一番詰まる）

🚨 **GitHub のパスワードは `git push` に使えない。** 2021年に廃止されたので、別の方法で認証する。

| 方法 | 手間 | 向き |
| --- | --- | --- |
| 🅰️ **GitHub CLI**（`gh auth login`） | **2分** | 🟢 まずこれ。自分でトークンを保管しなくてよい |
| 🅱️ VSCode / Cursor のサインイン | 3分 | エディタの中で完結させたい |
| 🅲 パーソナルアクセストークン（PAT） | 10分 | CLI が入れられない環境 / CI |

### 🅰️ GitHub CLI（推奨）

**Mac — `gh` を入れる:**

```sh
brew install gh
```

> 🚨 **`brew: command not found` と出たら、Homebrew が入っていない。** 1.1 で Git がすでに入っていた人は、1.2 を飛ばしているのでこの状態になる。**1.2 の「Mac — Homebrew 経由」の ①〜④ だけ**をやってから戻ってくる（`brew install git` は不要）。

**Windows:** Git for Windows 同梱の Credential Manager でもよい（初回 push でブラウザが開いて終わる）。`gh` を使うなら `winget install GitHub.cli`。

**ログインする:**

```sh
gh auth login
```

対話で聞かれるので、こう答える。

| 質問 | 選ぶ |
| --- | --- |
| What account do you want to log into? | **GitHub.com** |
| What is your preferred protocol for Git operations? | **HTTPS** |
| Authenticate Git with your GitHub credentials? | **Yes** |
| How would you like to authenticate GitHub CLI? | **Login with a web browser** |

→ 8桁のコードが表示される。Enter でブラウザが開くので、そのコードを貼って承認。

確認:

```sh
gh auth status
```

> 📝 **順番はバージョンで少し変わる。** 聞かれた内容で判断する。
> 🚨 `Authenticate Git with your GitHub credentials?` を **Yes** にしないと、`gh` のログインは通るのに `git push` だけ失敗する。

### 🅲 PAT（CLI が使えないとき）

GitHub → Settings → Developer settings → Personal access tokens → **Fine-grained tokens** → Generate new token

| 項目 | 入れる値 |
| --- | --- |
| Token name | **使う場所が分かる名前**（`macbook-cursor` など） |
| Expiration | **No expiration（無期限）** |
| Repository access | **All repositories** |
| Permissions → Contents | **Read and write**（これだけ） |

> 📝 **Expiration は無期限にしておく。** 期限が切れると、ある日とつぜん `git push` だけが通らなくなり、原因探しから始まることになる。**学習用途では、切れて詰まるコストのほうが大きい。**

> 📝 **Repository access も All repositories でよい。** Only select にすると、**新しくリポジトリを作るたびにトークンの設定を変えに行く**ことになり、そのたびに push が止まる。

> 🚨 **そのかわり、Permissions は必要な1つだけにする。** `Contents: Read and write` だけ付けておけば、仮に漏れても**ファイルの読み書き**までで止まる。⛔ Administration（リポジトリの削除）や Secrets には触らせない。

> 🚨 **無期限 × 全リポジトリなので、漏れたときの影響は大きい。** 漏らしたかもしれないと思ったら、すぐ **Settings → Developer settings → Fine-grained tokens** から **Delete** する。**消せばその瞬間に無効**になるので、気づいたら消すのが唯一で確実な対処。Token name を使う場所の名前にしておくと、どれを消せばよいかが分かる。

🚨 **表示されたトークンは一度しか見られない。** その場でパスワードマネージャーに保存する。

> 📝 **失くしても作り直せばよい。** 消して新しく発行するだけで、リポジトリには何も起きない。
> 🚨 **`Tokens (classic)` ではなく `Fine-grained tokens` を使う。** classic は `repo` にチェックを入れた時点で、コードの読み書きだけでなく**リポジトリの削除・設定変更・Webhook まで全部できる**トークンになる。fine-grained なら `Contents` だけに絞れる。
> 🚨 **トークンを `.env` やコードに書かない。** 一度 commit すると履歴に残り続ける。

> 📝 **`No expiration` が選べないときは、一番長いものを選ぶ。** 会社や学校のアカウントだと、組織のポリシーで上限が決まっていることがある。

push 時に聞かれたら、Username は GitHub のユーザー名、**Password の欄にトークンを貼る**。

---

## 2.3 リポジトリを作って push

```mermaid
flowchart LR
    S1["①<br/>GitHub上に<br/>リポジトリを作る"] ==> S2["②<br/>git remote add origin<br/>場所を教える"]
    S2 ==> S3["③<br/>git branch -M main<br/>名前を揃える"]
    S3 ==> S4["④<br/>git push -u origin main"]
    S4 ==> S5["⑤<br/>ブラウザで<br/>見えるか確認"]

    style S1 fill:#8E44AD,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:2px
```

**GitHub CLI があるなら1行で済む:**

```sh
gh repo create my-app --private --source=. --remote=origin --push
```

**手でやる場合:**

まず GitHub 上で**空の**リポジトリを作る（**README にチェックを入れない**）。そのあと、**1行ずつ**貼る。

```sh
git remote add origin https://github.com/<ユーザー名>/<リポジトリ名>.git
```

→ `<>` の部分は自分の値に置き換える。GitHub のリポジトリ画面に出ている URL をそのままコピーするのが確実。

```sh
git branch -M main
```

```sh
git push -u origin main
```

> 🚨 **最初は `--private` にする。** 公開は後からいつでも切り替えられるが、一度公開したものは取り消せない。API キーなどを混ぜていた場合に手遅れになる。

> 📝 2回目以降は `git push` だけでよい（`-u` で紐づけ済みのため）。

---

## 2.4 詰まったときの対処表

| 出たメッセージ | 原因 | 対処 |
| --- | --- | --- |
| `Support for password authentication was removed` | パスワードを入れている | 2.2 の認証をやる |
| `remote origin already exists` | すでに origin が登録されている | `git remote set-url origin <新しいURL>` |
| `src refspec main does not match any` | コミットが1つもない | 1.4 の `git commit` を先にやる |
| `failed to push some refs` / `fetch first` | GitHub 側に自分が持っていないコミットがある | `git pull --rebase origin main` → もう一度 push |
| `Permission denied` / `403` | 別アカウントの認証が残っている | `gh auth logout` → `gh auth login` |
| `Repository not found` | URL のタイポ、または private への権限不足 | `git remote -v` で URL を確認 |
| push は成功したが自分のコミットとして出ない | `user.email` が GitHub のメールと違う | 1.3 を直す（過去分は残る） |
| `gh` は通るのに `git push` で聞かれる | `Authenticate Git with your GitHub credentials?` を No にした | `gh auth login` をやり直す |

---

### ✅ Part 2 のチェック

- [ ] `gh auth status` が通る（または初回 push が成功した）
- [ ] ブラウザで自分のリポジトリにファイルが見える
- [ ] 適当に1行変えて `commit` → `push` が通る
- [ ] GitHub 上でそのコミットが**自分のアイコン付き**で出ている

---

## 次に読むもの

| | 内容 |
| --- | --- |
| [git.md](git.md) | ブランチ / Pull Request / コンフリクト解決 / `.gitignore` / `git stash` |
| [../01_app-release/app-release.md](../01_app-release/app-release.md) | このGitが、デプロイまでどう繋がるか |
