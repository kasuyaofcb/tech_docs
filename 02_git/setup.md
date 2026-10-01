# Git / GitHub 環境構築手順 — インストールから初めての push まで

> 📝 [git.md](git.md) が**概念の地図**なら、このドキュメントは**実際に手を動かす順番**。**Part 1（Git）だけで「壊したコードを戻せる」状態になる。** GitHub は必要になってから Part 2 を開けばよい。

🧠 想定する到達点:

- 自分のフォルダで `git commit` ができ、履歴が残る
- 間違えたとき、直前の状態に戻せる
- （Part 2）GitHub にリポジトリを作って `git push` できる

---

## 0. 全体マップ

### 🟢 Part 1 — Git（30分）

```mermaid
flowchart LR
    S1["1.1<br/>入っているか<br/>確認"] ==> S2["1.2<br/>インストール<br/>「無い場合だけ」"]
    S2 ==> S3["1.3<br/>初期設定<br/>名前・メール・main"]
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

### どれを何回やる？

この手順書には「**最初に1回やれば二度とやらないもの**」と「**毎回やるもの**」が混ざっている。各節の見出しに【 】で書いてある。

| 頻度 | やること | 節 |
| --- | --- | --- |
| **【PCで1回】** このPCで最初の1回だけ。次のプロジェクトでは**やらない** | Git のインストール、初期設定（名前・メール・main）、`gh` のインストールとログイン | 1.1 / 1.2 / 1.3 / 2.2 |
| **【一生に1回】** | GitHub アカウントの作成と 2FA | 2.1 |
| **【プロジェクトごとに1回】** 新しいプロジェクトを始めるたびに1回 | `.gitignore` を作る、`git init`、GitHub にリポジトリを作る | 1.4 の ② ④ / 2.3 |
| **【毎回】** ふだんの作業 | フォルダを開く、`git add` → `git commit`、`git push` | 1.4 の ① ⑤ / 2.3 の最後 |
| **【必要なとき】** 失敗したときだけ | 編集を捨てて戻す | 1.5 |

> 📝 **2つ目のプロジェクトからは、【PCで1回】の節は全部飛ばしてよい。** `git --version` や `git config` をやり直す必要はない。

> 📝 **Part 1 と Part 2 は切り離してよい。** Part 1 だけでバージョン管理は成立する（自分のPCの中に履歴が残る）。GitHub は「人に見せる / 別のPCから触る / PCが壊れても残す」ための箱で、**無くても作業は進む**。

> 🔧 **うまくいかないときは、巻末の[困ったとき](#困ったとき)を見る。** エラーメッセージから引けるようにしてある。

---

# Part 1: Git

## 1.0 はじめに — 進め方の約束

> 🚨 **この印は「取り返しがつかないこと」にだけ付けている。** 消えたら戻らない、漏れたら取り消せない、のどちらか。🚨 が出てきたら、そこだけは必ず読む。📝 は補足なので、急いでいれば飛ばしてよい。

### ターミナルを開く

**この手順書のコマンドは、すべて Cursor のターミナルに打つ。** 別のアプリを開く必要はない。

| | やること |
| --- | --- |
| 開き方 | Cursor 上部メニューの **ターミナル** → **新しいターミナル**（英語表示なら **Terminal** → **New Terminal**） |
| 打つ場所 | 下半分に出たパネルの、**カーソルが点滅している位置**。貼って `Enter` |

> 📝 **この手順書の `Ctrl` は、Mac では `control` キー**（キーボードの左下）。`⌘`（command）とは別のキー。`Cmd` と書いてあるところだけが `⌘`。

![Cursor のターミナルを開いた状態](images/cursor-terminal.png)

行末の記号は環境で違う（`$` / `%` / Windows は `>`）。**どれでもよい。カーソルが点滅していれば打てる状態。**

> 📝 **起動時に英語のメッセージが出ても気にしない。** 上の画面の `The default interactive shell is now zsh.` のような案内が出ることがある。エラーではない。

### コマンドの打ち方 — 5つのルール

| ルール | なぜ |
| --- | --- |
| **コードブロックの右上のコピーボタンで貼る** | 手で打つと必ずどこか間違える |
| **1行コピーしたら 1行 Enter。まとめて貼らない** | 途中で失敗したとき、どこまで進んだかが分かる |
| **`#` で始まる行は打たない** | 説明文。打っても何も起きないが、混乱する |
| **`<` `>` で囲まれた部分は自分の値に置き換える** | 例: `git restore <ファイル名>` → `git restore index.html` |
| **行頭の `$` や `%` はコマンドの一部ではない** | 他のサイトや AI の答えは `$ git status` のように書くことがある。**`$` を除いた `git status` だけを打つ** |

> 📝 **何も表示されなくても、たいてい成功している。** Git は成功したとき黙っているコマンドが多い。**エラーが出ていなければ次へ進む。**

### 止まった・間違えたときの3つのキー

**これだけ覚えておけば、どんな画面になっても抜けられる。**

| 状況 | 押すキー | 何が起きるか |
| --- | --- | --- |
| **貼り間違えた / 打ち間違えた** | `Ctrl` + `C`（Mac は `control` + `C`。`⌘` + `C` ではない） | 打ちかけの行が捨てられ、新しい行から打ち直せる |
| **画面が文字で埋まって止まった** | `q` | 出力が長いとページャが開く。**壊れていない** |
| **見慣れない編集画面になった** | `Esc` → `:q!` → `Enter` | エディタが開いている。保存せずに閉じる |

> 📝 **`Ctrl` + `C` は、インストールが走っている最中には押さない。** 途中で止めると中途半端な状態になる。**打ち間違えたときと、「待っているだけで何も進まない」ときに使う。**

### 詰まったら、AI に聞く

**この手順書どおりに進まないことは必ず起きる。** 画面の表示が違う、知らないエラーが出る、など。そのときは **Cursor のチャット**（`Cmd` + `L`）に聞くのがいちばん早い。Claude Code を入れていればそれでもよい。

**聞き方は1つだけ覚える — エラーをそのまま貼る。**

| ⛔ 伝わらない | 🟢 解決する |
| --- | --- |
| 「git が動きません」 | **ターミナルに出た文字を全部そのままコピーして貼る** ＋「何をしようとして、何をしたか」を1行 |

> 📝 **要約しないのがコツ。** 自分で「たぶんこういうエラー」とまとめると、肝心の情報が落ちる。**原文のまま貼るのがいちばん速い。**

> 🚨 **AI の答えをそのまま実行しない3つ。** 返ってきたコマンドに `rm` / `--force`・`-f` / `reset --hard` が入っていたら、実行する前に「**これは何をするコマンドですか。元に戻せますか**」ともう一度聞く。AI は「直す」ために、**作業を消すコマンドを提案することがある。**

> 🚨 **トークンやパスワードは貼らない。** エラーに混ざっていたら、その部分を `****` に置き換えてから貼る。

---

## 1.1 Git がすでに入っているか確認【PCで1回】

ターミナルにこれを貼って Enter。**入っていればインストールは丸ごと不要。**

```sh
git --version
```

| 出た結果 | 次にやること |
| --- | --- |
| `git version 2.39.3` のようにバージョンが出た | **1.2 は読まずに飛ばして、1.3 へ** |
| `command not found: git` | 1.2 でインストール |
| 「コマンドライン・デベロッパ・ツール」のダイアログが出た（Mac） | 「インストール」を押す。終わったらもう一度 `git --version`。入れば **1.2 は飛ばして 1.3 へ** |

> 📝 **バージョンが出た人は、1.2 をやってはいけない。** Homebrew のインストールに10分かかるうえ、**入れる必要がまったくない**。**読み飛ばして 1.3 に進むのが正解。**

> 📝 **Mac には最初から Git が入っていることが多い。** Xcode Command Line Tools に同梱されているため。少し古くても、学習やふつうの開発には問題ない。

---

## 1.2 インストール（入っていなかった場合だけ）【PCで1回】

### Mac — Homebrew 経由

| | やること | 注意 |
| --- | --- | --- |
| ① | 下のコマンドを貼って Enter | |
| ② | `Password:` と出たら、Mac のログインパスワードを入力 | **打っても画面には1文字も出ない。** そのまま Enter（Apple ID ではなく、**Macにログインするときのパスワード**） |
| ③ | **`Press RETURN/ENTER to continue` で止まる** | **`Enter` を1回押す。** 他のキーを押すと中断される |
| ④ | インストールが走る | **5〜10分かかる。** 文字が流れ続けるが、ここは待つだけ |
| ⑤ | 最後に出た `Next steps:` の行を実行 | **ここだけは自分の画面を見る**（下で説明） |
| ⑥ | `brew -v` で確認 | `Homebrew 4.x.x` と出れば成功 |
| ⑦ | `brew install git` → `git --version` | |

> 📝 **②と③で画面が止まるが、固まったのではない。** インストーラーが返事を待っている。②はパスワードを打って `Enter`、③は `Enter`。順番が前後することもあるので、**出た文字を見て**答える。

**① Homebrew のインストール:**

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**このコマンドの中にも `$` が入っているが、気にしなくてよい。** 貼るとこうなる。

![Homebrew のコマンドを貼り付けた状態](images/homebrew-paste.png)

| 場所 | 中身 |
| --- | --- |
| `MacBook-Pro-2:test ...$` | **プロンプト。もとから出ている。自分で打つものではない**（中身は人によって違う。`ユーザー名@MacBook-Air ~ %` のように `%` で終わることも多い） |
| その右の `/bin/bash -c "$(curl ...` | **貼り付けた中身。ここだけがコピーした分** |

この状態で `Enter`。あとは待つ。

> 📝 **Homebrew とは**: Mac に開発ツールを入れるための道具。`brew install 〇〇` でツールが入る。Part 2 で使う `gh`（GitHub をターミナルから操作する道具）も、これで入れる。

**⑤ 最大の詰まりどころ。** インストールが終わると、画面の最後にこう出る。

```text
==> Installation successful!

==> Homebrew has enabled anonymous aggregate formulae and cask analytics.
（英語の説明が数行続く）

==> Next steps:
- Run these commands in your terminal to add Homebrew to your PATH:
    echo >> /Users/あなたの名前/.zprofile                                                      ← ここから
    echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/あなたの名前/.zprofile
    eval "$(/opt/homebrew/bin/brew shellenv)"                                                 ← ここまで
- Run brew help to get started
- Further documentation:
    https://docs.brew.sh
```

**`==> Next steps:` の下の、字下げされた3行**だけを使う（`←` の印はこの手順書で付けたもので、実際の画面には出ない）。

| 行 | 使う？ |
| --- | --- |
| `Installation successful!` | 見るだけ。**ここまで来れば成功** |
| `analytics` の説明 | 読まなくてよい |
| `Next steps:` の下の**字下げされた3行** | **1行ずつコピーして、ターミナルに貼って Enter** |
| `Run brew help` / `docs.brew.sh` | 実行しなくてよい |

> 📝 **上にスクロールしないと見えないときは、ターミナルのパネルをマウスでスクロールして `Next steps:` を探す。** 文字が流れて見失っても、インストールは成功している。

**この手順書の行ではなく、自分の画面に出た行を使う。** Apple シリコンは `/opt/homebrew`、Intel Mac は `/usr/local` と、パスが変わる。

> 📝 **PATH（パス）とは**: ターミナルが「打たれたコマンドの本体を探しに行く場所」の一覧。Homebrew は入れただけではこの一覧に載らないので、この3行で登録する。登録しないと `brew: command not found` になる。

> 📝 **なぜ2種類の行があるのか**: `echo ... >> ~/.zprofile` は「**次にターミナルを開いたとき**のための設定ファイルへの追記」、`eval "$(...)"` は「**いま開いているターミナル**への即時反映」。役割が違うので両方必要。

### Windows — 公式インストーラー

[git-scm.com](https://git-scm.com/) から `Git-*.exe` をダウンロードして実行する。**選択肢が出る画面だけ、選ぶものを決めておく。**

| 画面 | 選ぶもの | なぜ |
| --- | --- | --- |
| Adjusting your PATH environment | **Git from the command line and also from 3rd-party software**（真ん中） | PowerShell でも Cursor でも Git が使える |
| Choosing HTTPS transport backend | **Use the native Windows Secure Channel library** | Windows の証明書をそのまま使える |
| Configuring the line ending conversions | **Checkout Windows-style, commit Unix-style**（既定） | 改行コードの違いを自動で吸収する |
| それ以外 | すべて既定のまま Next | 変える理由がない |

> 📝 **終わったら Cursor を再起動する。** Windows は起動中のアプリに PATH の変更が届かない。

> 📝 Git for Windows には **Git Credential Manager** が同梱されている。Part 2 の認証が、初回 push のときにブラウザが開いて終わる。

---

## 1.3 初期設定（3つだけ）【PCで1回】

コミットに「誰がやったか」を刻むための設定。**これをしないとコミットできない。**

1つ目と2つ目は、`<あなたの名前>` などを**自分の値に書き換えてから**打つ。ターミナルの中の文字はクリックで直せないので、**先に Cursor のエディタで1行を完成させてから**ターミナルに貼る。

| | やること |
| --- | --- |
| 1 | Cursor で `Cmd` + `N`（新しいタブが開く） |
| 2 | 下のコードブロックをコピーして、そのタブに貼る |
| 3 | `<あなたの名前>` を、**`<` `>` ごと**自分の名前に書き換える（`"` は残す） |
| 4 | タブの中で `Cmd` + `A`（全部選択）→ `Cmd` + `C`（コピー）。ターミナルに貼って Enter |
| 5 | タブの中で `Cmd` + `A` → `delete` キーで中身を消してから、メールアドレスの行で 2〜4 をくり返す |

```sh
git config --global user.name "<あなたの名前>"
```

```sh
git config --global user.email "<あなたのメールアドレス>"
```

→ 完成形の例: `git config --global user.name "Taro Yamada"` / `git config --global user.email "taro@example.com"`

> 📝 **書き換えずに Enter してしまっても、エラーは出ない。** 「あなたの名前」という名前で登録されるだけ。正しい値で同じコマンドをもう一度打てば上書きされる。メールの `@` は**半角**で。

> 📝 **使っていたタブは、閉じるときに「保存しますか？」と聞かれたら「保存しない」でよい。**

3つ目は**そのまま貼って** Enter（書き換える部分はない）。

```sh
git config --global init.defaultBranch main
```

> 📝 **これは「最初のブランチ名を `main` にする」設定。** Mac に最初から入っている Git は、何もしないと昔の名前の `master` で始まり、`git init` のたびに黄色い `hint:` が何行も出る。GitHub は `main` が標準なので、ここで揃えておく（詳しくは [git.md](git.md) の §10）。

確認:

```sh
git config --global --list
```

| 項目 | 何に使われるか | 注意 |
| --- | --- | --- |
| `user.name` | コミット履歴に出る名前 | 本名でもハンドルネームでもよい |
| `user.email` | コミット履歴に出るメール | いまは**ふだん使っているメール**でよい |
| `init.defaultbranch` | `git init` したときの最初のブランチ名 | `main` になっていればよい（一覧では小文字で表示される） |

→ **`<` や `あなたの` が残っていたら、書き換え忘れ。** 正しい値でもう一度打つ。

> 📝 **Part 2 で GitHub のアカウントを作るときは、ここで入れたのと同じメールで登録する。** 違うと、GitHub 上で自分のコミットとして扱われない（コミットの横に自分のアイコンが出ない）。

> 🚨 **メールアドレスは公開される。** GitHub に上げると、コミット履歴から誰でも見える。公開したくない人は、Part 2 でアカウントを作ったあとに、GitHub が用意する非公開用のアドレス（`12345678+username@users.noreply.github.com` の形）に設定し直す。場所は GitHub の Settings → Emails → **Keep my email addresses private**。

> 📝 **Windows でインストーラーの改行設定を既定のまま進めたなら、追加の設定は要らない。** 変えてしまった場合だけ `git config --global core.autocrlf true` を実行する。

---

## 1.4 最初のコミット

**ここからは練習用のフォルダ `git-practice` で進める。** 本物のプロジェクトを壊す心配がなく、画面もこの手順書と同じになる。**本番のプロジェクトでは、1.4 の ②・④・⑤ をそのフォルダでやるだけ**（1.1〜1.3 は【PCで1回】なので要らない。③ は練習用）。

### ① 練習用フォルダを作って開く（作るのは【プロジェクトごとに1回】、開くのは【毎回】）

| | やること |
| --- | --- |
| 作る | Finder の上部メニュー **移動** → **ホーム** を開き、そこで右クリック → **新規フォルダ**。名前を `git-practice` にする |
| 開く | Cursor 上部メニューの **ファイル** → **フォルダを開く**（英語表示なら **File** → **Open Folder**）→ `git-practice` を選ぶ |

![git-practice を開いた直後の Cursor](images/folder-opened.png)

→ 左上に **`GIT-PRACTICE`** と出ていれば開けている。中身はまだ空。

> 📝 **デスクトップや「書類」フォルダに作らないのは、iCloud と同期されていることが多いため。** Git の履歴（`.git`）が同期されると、ファイルが二重になるなどして壊れることがある。ホームフォルダは同期されない。

**Cursor でフォルダを開くと、ターミナルも自動でそのフォルダの中で動く。** だから必ず「フォルダを開いてから」ターミナルを使う。

> 📝 **「このフォルダー内のファイルの作成者を信頼しますか？」と聞かれることがある。** Cursor の設定によっては出ない。出たら、**自分で作ったフォルダなので「はい、作成者を信頼します」を選ぶ**（英語表示なら **Yes, I trust the authors**）。

フォルダを開いたら、**ターミナルを開き直す。**

| | やること |
| --- | --- |
| すでに開いている場合 | ターミナルパネル右上の **🗑（ゴミ箱）** で閉じてから、上部メニュー **ターミナル** → **新しいターミナル** |
| 開いていない場合 | 上部メニュー **ターミナル** → **新しいターミナル** |

いま自分がどこにいるかを確認する:

```sh
pwd
```

→ **`/git-practice` で終わっていればOK**（例: `/Users/あなたの名前/git-practice`）。

> 📝 **ここが違うと、関係ないフォルダの履歴を取り始める。** `/Users/あなたの名前` で終わっていたら、フォルダを開けていないか、ターミナルを開き直していない。

### ② `.gitignore` を作る — `git add` より先に【プロジェクトごとに1回】

**これを先に作らないと、記録してはいけないファイルまで記録される。**

Cursor の左のファイル一覧で右クリック → **新しいファイル**（英語表示なら **New File...**）→ `.gitignore` という名前で作り、中にこれを貼る。

```text
.env
.env.local
node_modules/
.DS_Store
```

| 書いたもの | なぜ除外するか |
| --- | --- |
| `.env` / `.env.local` | **APIキーやパスワードが入っている。** 一度記録すると履歴に残り続け、あとから消すのは非常に面倒 |
| `node_modules/` | ライブラリの置き場。数万ファイルあり、記録する意味がない（`npm install` で作り直せる） |
| `.DS_Store` | Mac が勝手に作る管理ファイル |

![.gitignore に4行を貼った直後（保存前）](images/gitignore-unsaved.png)

**貼ったら `Cmd` + `S`（Windows は `Ctrl` + `S`）で保存する。** 上の画像のように、タブの `.gitignore` の横に `●` が出ている間は、まだ保存されていない。

> 📝 **5行目に薄い文字（上の画像では `.cursorrules`）が出ることがある。** Cursor の AI が「次はこれでは？」と出している**候補**で、まだ書かれていない。`Tab` を押さなければ入らないので、無視して保存してよい。

> 🚨 **一度 `commit` したファイルは、あとから `.gitignore` に書いても記録され続ける。** だから `git add` より先に作る。

### ③ 練習用のファイルを1つ作る（練習のときだけ）

いまの中身は `.gitignore` だけ。これでは 1.5 で「戻す」練習ができないので、**ふつうのファイルを1つ作っておく。**

② と同じやり方で `memo.txt` を作り、中にこれを貼る。

```text
はじめてのGit
```

貼ったら **`Enter` を1回押して改行してから**、`Cmd` + `S` で保存する（下の画像のように、2行目が空いている状態）。

> 📝 **最後に改行を入れるのは、1.5 の練習で表示を見やすくするため。** 入れ忘れると、あとで `git diff` に `\ No newline at end of file`（ファイルの最後に改行がない）という英語が出る。壊れたわけではない。

![.gitignore と memo.txt を作って保存した状態](images/files-created.png)

→ 左のファイル一覧に **`.gitignore` と `memo.txt` の2つ**が並んでいればOK。

### ④ 履歴を取り始める【プロジェクトごとに1回】

**1行ずつ**貼って Enter。

```sh
git init
```

何が記録されようとしているか、先に見ておく:

```sh
git status
```

![git init と git status を打った直後](images/git-status-first.png)

（画像の `Initialized empty Git repository in ...` のパスは撮影環境のもの。自分の画面では `/Users/あなたの名前/git-practice/.git/` のように出る）

→ `Untracked files:` の下に **`.gitignore` と `memo.txt` の2つだけ**が赤字で出ていればOK。

> 📝 **ここに `.env` が出てきたら、② をやり直す。** `.gitignore` に書けていない。

> 📝 **左のファイル一覧に緑の `U` が付く。** Untracked（まだ記録していない）の頭文字。コミットすると消える。

### ⑤ コミットする【毎回】

ここからが、ふだんの作業で毎回やること。**1行ずつ**貼って Enter。

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

→ `10a8d6c (HEAD -> main) 最初のコミット` のように出れば成功（先頭の7文字は人によって違う）。

| コマンド | 意味 |
| --- | --- |
| `git init` | このフォルダを「履歴を取る対象」にする（隠しフォルダ `.git` ができる） |
| `git status` | いまの状態を見る。**迷ったらこれ** |
| `git add .` | 今の状態を「記録する候補」に入れる |
| `git commit -m "..."` | セーブする |
| `git log --oneline` | セーブ履歴を1行ずつ見る |

> 📝 `.git` を消すと履歴も消える。**フォルダをコピーするときは `.git` ごとコピー**する。

> 📝 作業ディレクトリ → ステージング → ローカル履歴、という3段構えの意味は [git.md](git.md) の「全体の流れ」を読む。

---

## 1.5 戻し方（Part 1 の目的はここ）【必要なとき】

**コードを大きく書き換える前に `commit` しておけば、壊れても戻せる。** これが Part 1 をやる理由。

> 📝 **この節は読むだけ。ここのコマンドはまだ打たない。** 実際に試すのは、このあとの「✅ Part 1 のチェック」の下にある「戻せることを1回だけ試す」。

**まず、何が変わったかを見る:**

```sh
git diff
```

**1ファイルだけ戻す:**

```sh
git restore <ファイル名>
```

**編集を全部捨てて戻す:**

```sh
git restore .
```

> 🚨 **`git restore` は編集を消す。取り消せない。** 残したい変更があるなら、先に `git commit` する。**迷ったら必ず `git diff` で中身を見てから。**

> 📝 **新しく作ったファイルは消えない。** `git restore .` が戻すのは「すでに記録されたことがあるファイルへの編集」だけ。新規ファイルはそのまま残るので、不要なら手で削除する。

> 📝 **AI にコードを書き換えてもらうなら、指示を出す前に1回 `commit` しておくのがいちばん安い保険。** 気に入らなければ `git restore .` で指示前に戻る。

> 📝 もっと踏み込んだ戻し方（コミット済みを取り消す / 直前のコミットのメッセージを直す）は [git.md](git.md) の「やらかした時の戻し方」にある。

---

### ✅ Part 1 のチェック

- [ ] `git --version` でバージョンが出る
- [ ] `git config --global --list` に `user.name` と `user.email` と `init.defaultbranch=main` がある
- [ ] `pwd` が `/git-practice` で終わる
- [ ] `git status` に `.env` が出てこない
- [ ] `git log --oneline` にコミットが1つ以上出る

**最後に、戻せることを1回だけ試す:**

> 🚨 **この練習は `git-practice` でやる。** `git restore .` は、まだ `commit` していない編集を**すべて**消す。ほかのプロジェクトのフォルダでは打たない。

1. 1.4 の ③ で作った `memo.txt` を開き、2行目（空いている行）にこれを貼り、**`Enter` を1回押してから**保存する（`Cmd` + `S`）

```text
消えても困らない行
```

2. ターミナルで差分を見る

```sh
git diff
```

![memo.txt に1行足して git diff を打った直後](images/git-diff.png)

→ 緑の **`+消えても困らない行`** が出ればOK（その下に `\ No newline at end of file` と英語が出ても問題ない）。`+` は「足された行」の印。左の一覧の `memo.txt` には、Modified（変更あり）の **`M`** が付く。

3. 編集を捨てて戻す

```sh
git restore .
```

→ `memo.txt` の2行目が消えて、`はじめてのGit` だけに戻ればOK。

**🎉 Part 1 のゴール — こうなっていれば完了:**

![Part 1 のゴール：ファイル2つと「最初のコミット」](images/part1-goal.png)

| 見るところ | こうなっていればOK |
| --- | --- |
| 左のファイル一覧 | `.gitignore` と `memo.txt` の2つ。`U` や `M` が付いていない |
| ターミナルの最後 | `git log --oneline` の結果に `最初のコミット` |
| 左下のステータスバー | `main` と出ている（1.3 の3つ目の設定が効いている） |

**ここまでできたら、GitHub は後回しでよい。** 本番のプロジェクトでは、1.4 の ① で開くフォルダを変えて、②・④・⑤ を同じようにやる（③ の `memo.txt` は練習用なので要らない）。

---

# Part 2: GitHub

> 📝 **必要になってから開く。** 「コードを人に見せたい」「別のPCから触りたい」「PCが壊れても残したい」のどれかが出てきたタイミング。

## 2.1 アカウントを作る【一生に1回】

| | やること | 注意 |
| --- | --- | --- |
| ① | [github.com](https://github.com) で Sign up | 画面の案内のとおりで迷わない |
| ② | メールアドレスを認証 | **1.3 の `user.email` と同じメールにする**（違う場合は Settings → Emails で追加登録する） |
| ③ | 2要素認証（2FA）を設定 | 必須。下を読む |

**③ 2要素認証（2FA）— ここだけ丁寧に**

**GitHub は 2FA が必須。** 設定しないまま使い続けると、アカウントが制限される。

| | やること |
| --- | --- |
| 1 | スマホに認証アプリを入れる（**Google Authenticator** / **Microsoft Authenticator** / 1Password など） |
| 2 | GitHub の **Settings** → **Password and authentication** → **Enable two-factor authentication** |
| 3 | 画面に出た QRコードを、認証アプリで読み取る |
| 4 | アプリに出た6桁の数字を GitHub に入力 |
| 5 | **リカバリーコードが表示される。必ずダウンロードして保存する** |

> 🚨 **5を飛ばさない。** スマホを機種変更したり失くしたりすると、**リカバリーコードが無い限りアカウントに二度と入れなくなる。** パスワードマネージャーに入れるか、印刷して手元に置く。

---

## 2.2 GitHub にログインする — `gh` を使う【PCで1回】

**GitHub のパスワードは `git push` に使えない。** 2021年に廃止された。代わりに **GitHub CLI（`gh`）** という道具に、ログインの手続きを任せる。

> 📝 **GitHub CLI とは**: GitHub の操作を**ターミナルから打てるようにする、GitHub 公式の道具**。コマンド名は `gh`。CLI は Command Line Interface の略で、「**ターミナルで文字を打って使う**」という意味。
>
> `gh auth login` を1回やれば、以降の `git push` で GitHub のパスワードを聞かれなくなる。2.3 のリポジトリ作成も `gh` なら1行で済む。

### ① `gh` を入れる

```sh
brew install gh
```

→ 最後に `🍺` の絵文字が付いた行が出れば完了。

> 📝 **`brew: command not found` と出たら、Homebrew が入っていない。** 1.1 で Git がすでに入っていた人はこの状態になる。**1.2 の「Mac — Homebrew 経由」の ①〜⑥ だけ**をやってから戻る（`brew install git` は不要）。

> 📝 **Windows** は `brew` の代わりに `winget install GitHub.cli` を打つ。終わったら Cursor を再起動する。

### ② ログインする

```sh
gh auth login
```

質問が順番に出るので、**矢印キーで選んで Enter** で答える。

| 質問 | 選ぶ |
| --- | --- |
| What account do you want to log into?（新しい版では `Where do you use GitHub?`） | **GitHub.com** |
| What is your preferred protocol for Git operations? | **HTTPS** |
| Authenticate Git with your GitHub credentials? | **Yes** |
| How would you like to authenticate GitHub CLI? | **Login with a web browser** |

→ `First copy your one-time code: XXXX-XXXX` と8桁のコードが出る。Enter を押すとブラウザが開くので、そのコードを入れて承認する。

> 📝 **質問の順番はバージョンで少し変わる。** 聞かれた内容で判断する。

> 📝 `Authenticate Git with your GitHub credentials?` を **Yes** にしないと、`gh` のログインは通るのに `git push` だけ失敗する。

確認:

```sh
gh auth status
```

→ `✓ Logged in to github.com account <自分のユーザー名>` と出ればOK。

> 📝 **会社のPCなどで `gh` を入れられない場合だけ**、巻末の「[付録 — `gh` が使えないとき](#付録--gh-が使えないとき)」を見る。

---

## 2.3 リポジトリを作って push【プロジェクトごとに1回】

**Part 1 で作った `git-practice` をそのまま GitHub に上げる。** Cursor で `git-practice` を開いたまま、ターミナルで進める。

**1行で済む:**

```sh
gh repo create git-practice --private --source=. --remote=origin --push
```

| 部分 | 意味 |
| --- | --- |
| `git-practice` | GitHub 上のリポジトリ名（フォルダ名と同じにしておくと迷わない） |
| `--private` | 自分だけが見られる設定 |
| `--source=.` | いまいるフォルダを上げる |
| `--remote=origin` | 手元の Git に、GitHub の場所を `origin` という呼び名で登録する |
| `--push` | 作ったらすぐ push する |

→ 最後に `✓ Pushed commits to https://github.com/...` と出れば成功。ブラウザで確かめるなら、これを貼って Enter（GitHub のリポジトリ画面が開く）:

```sh
gh repo view --web
```

> 🚨 **最初は `--private` にする。** 公開は後からいつでも切り替えられるが、**一度公開したものは取り消せない。** APIキーなどを混ぜていた場合に手遅れになる。

> 📝 **このあとは【毎回】 `git push` だけでよい**（最初の push で GitHub と紐づいたため）。ふだんの流れは「`git add .` → `git commit -m "..."` → `git push`」の3つ。

---

### ✅ Part 2 のチェック

- [ ] `gh auth status` に `Logged in` と出る
- [ ] `gh repo view --web` でブラウザが開き、`.gitignore` と `memo.txt` が見える
- [ ] リポジトリ名の横に **`Private`** と出ている
- [ ] `memo.txt` に1行足して `git add .` → `git commit -m "メモを追記"` → `git push` が通る
- [ ] GitHub 上でそのコミットが**自分のアイコン付き**で出ている

**🎉 Part 2 のゴール — こうなっていれば完了:**

![Part 2 のゴール：GitHub の git-practice リポジトリ](images/part2-goal.png)

（灰色で塗った部分には、自分のユーザー名とアイコンが出る）

| 見るところ | こうなっていればOK |
| --- | --- |
| リポジトリ名の横 | **`Private`** |
| ファイル一覧 | `.gitignore` と `memo.txt` |
| 一覧の上の行 | 自分のアイコン ＋ ユーザー名 ＋ **いちばん新しい**コミットメッセージ（チェックの追記をしたあとなら `メモを追記`） |
| 右下の Contributors | 自分が1人。**アイコンが出ていなければ 1.3 の `user.email` が GitHub と違う** |

> 📝 **`Add a README` のボタンは押さなくてよい。** README は「このリポジトリの説明書き」で、練習には要らない。

---

## 困ったとき

エラーメッセージや症状から引く。**ここに無いものは、1.0 の「詰まったら、AI に聞く」。**

### Part 1 — Git

| 症状 | 原因 | 対処 |
| --- | --- | --- |
| 画面が文字で埋まって止まった | 出力が長く、ページャが開いた | **`q` を押す。** 壊れていない |
| 貼り間違えた / 打ち間違えた | — | **`Ctrl` + `C`** で打ちかけの行を捨てて、打ち直す |
| 見慣れない編集画面になって `q` も効かない | エディタが開いている | **`Esc` → `:q!` → `Enter`** |
| `Press RETURN/ENTER to continue` で止まった | インストーラーが返事を待っている | **`Enter` を1回押す**（他のキーは中断） |
| パスワードを打っても1文字も出ない | 仕様 | **そのまま打って Enter。** 正しく入力されている |
| `command not found: git` | Git が入っていない | 1.2 |
| `brew: command not found` | Homebrew が入っていない、または PATH が通っていない | 1.2 の ⑤ をやる。それでも駄目ならターミナルを開き直す |
| Windows: インストールしたのに `git` が見つからない | PATH が届いていない | **Cursor を再起動する** |
| `Please tell me who you are` | `user.name` / `user.email` が未設定 | 1.3 |
| `git status` に `.env` が出る | `.gitignore` に書けていない | 1.4 の ② |
| `pwd` が `/Users/あなたの名前` で終わる | フォルダを開けていない | 1.4 の ① |
| `git restore .` したのにファイルが残る | 新規作成したファイルは対象外 | 手で削除する |

### Part 2 — GitHub

| 出たメッセージ | 原因 | 対処 |
| --- | --- | --- |
| `Support for password authentication was removed` | GitHub のパスワードを入れている | 2.2 で `gh` にログインする |
| `fatal: Authentication failed for 'https://github.com/...'` | ログインが切れている（付録の PAT を使っているなら **期限切れ**） | `gh auth login` をやり直す。PAT なら付録 A で作り直す（2分）。**履歴は無事なので慌てない** |
| `remote origin already exists` | すでに origin が登録されている | `git remote set-url origin <新しいURL>` |
| `src refspec main does not match any` | コミットが1つもない | 1.4 の `git commit` を先にやる |
| `failed to push some refs` / `fetch first` | GitHub 側に自分が持っていないコミットがある | `git pull --rebase origin main` → もう一度 push |
| `Permission denied` / `403` | 別アカウントの認証が残っている | `gh auth logout` → `gh auth login` |
| `Repository not found` | URL のタイポ、または private への権限不足 | `git remote -v` で URL を確認 |
| push は成功したが自分のコミットとして出ない | `user.email` が GitHub のメールと違う | 1.3 を直す（過去分は残る） |
| `gh` は通るのに `git push` で聞かれる | `Authenticate Git with your GitHub credentials?` を No にした | `gh auth login` をやり直す |

---

## 付録 — `gh` が使えないとき

**ふつうは 2.2 の `gh` でよい。** ここは、会社のPCでソフトのインストールが禁止されているなど、`gh` を入れられない場合だけ読む。

### A. トークン（PAT）を作る

> 📝 **トークンとは**: パスワードの代わりに使う、**GitHub が発行する、数十文字の長いランダムな文字列**。パスワードと違って「**できることを絞れる**」「**期限を付けられる**」「**いつでも消せる**」ので、漏れたときの被害を小さくできる。（2.2 の `gh` も、裏では同じようなトークンを自動で作って保管している。）

> 📝 **PAT とは**: Personal Access Token（パーソナル・アクセス・トークン）の略。トークンを**自分で GitHub の画面から作るもの**。作ったトークンは自分でパスワードマネージャーなどに保管し、`git push` でパスワードを聞かれたときに貼る。

GitHub → Settings → Developer settings → Personal access tokens → **Fine-grained tokens** → Generate new token

| 項目 | 入れる値 |
| --- | --- |
| Token name | **使う場所が分かる名前**（`macbook-cursor` など） |
| Expiration | **90日（おすすめ）**／`No expiration`（無期限）も選べる |
| Repository access | **All repositories** |
| Permissions → Contents | **Read and write**（これだけ） |

**Expiration の選び方:**

| 選択 | 向いている場面 | 代償 |
| --- | --- | --- |
| 🟢 **90日（おすすめ）** | ふだんの学習・個人開発 | 90日ごとに作り直す（2分） |
| ⚠️ `No expiration` | 作り直す手間を絶対に避けたい | **漏れたら、自分で消すまで永久に有効** |

> 📝 **期限を切るのをすすめる理由はひとつ。** トークンは、うっかりコードに書いてしまったり、画面共有に映ったりして漏れることがある。**期限があれば、漏れたことに気づかないままでも、いつかは自然に無効になる。**

> 📝 **期限切れは、知っていれば怖くない。** 起きるのは `fatal: Authentication failed` が出て push が止まることだけで、**壊れたわけでも履歴が消えたわけでもない。** 作り直せば2分で戻る。期限が近づくと GitHub からメールも届く。

> 📝 **Repository access は All repositories でよい。** Only select にすると、新しくリポジトリを作るたびに設定を変えに行くことになる。

> 📝 **そのかわり、Permissions は必要な1つだけにする。** `Contents: Read and write` だけなら、仮に漏れても**ファイルの読み書き**までで止まる。Administration（リポジトリの削除）や Secrets には触らせない。

**表示されたトークンは一度しか見られない。** その場でパスワードマネージャーに保存する。push 時に聞かれたら、Username は GitHub のユーザー名、**Password の欄にトークンを貼る**。

> 🚨 **漏らしたかもしれないと思ったら、すぐ Delete する。** Fine-grained tokens の画面から消せば、**その瞬間に無効**になる。期限を待つ必要はない。

> 📝 **`Tokens (classic)` ではなく `Fine-grained tokens` を使う。** classic は `repo` にチェックを入れた時点で、コードの読み書きだけでなく**リポジトリの削除・設定変更・Webhook まで全部できる**トークンになる。

> 🚨 **トークンを `.env` やコードに書かない。** 一度 commit すると履歴に残り続ける。

### B. リポジトリを GitHub の画面で作って push

まず GitHub 上で `git-practice` という名前の**空の**リポジトリを作る（**README にチェックを入れない**）。そのあと**1行ずつ**貼る。

```sh
git remote add origin https://github.com/<ユーザー名>/<リポジトリ名>.git
```

→ GitHub のリポジトリ画面に出ている URL をそのままコピーするのが確実。

```sh
git branch -M main
```

```sh
git push -u origin main
```

→ ユーザー名とパスワードを聞かれたら、Username は GitHub のユーザー名、**Password の欄には A で作ったトークンを貼る**（GitHub のパスワードではない）。

---

## 次に読むもの

| | 内容 |
| --- | --- |
| [git.md](git.md) | ブランチ / Pull Request / コンフリクト解決 / `.gitignore` / `git stash` |
| [../01_app-release/app-release.md](../01_app-release/app-release.md) | このGitが、デプロイまでどう繋がるか |
