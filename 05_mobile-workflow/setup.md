# スマホから PC の Claude Code を動かす手順 — リモートコントロール

> 📝 PC の Claude Code を**つけたまま**にしておき、外出先からスマホで話しかけて作業を頼む方法。**GitHub は不要**で、PC にあるフォルダ・ファイルをそのまま使える。
>
> PC を閉じていても使いたくなったら → [setup-web.md](setup-web.md)（GitHub を使う方法）

🧠 想定する到達点:

- 自転車や電車の移動中に思いついたことを、**スマホから Claude Code に話しかけて**、PC のプロジェクトにメモや資料として残せる
- PC で途中まで進めた作業の続きを、スマホから指示できる

🧠 こんな使い方ができる:

| 場面 | スマホから送る言葉の例 |
| --- | --- |
| アイデアを思いついた | 「ミニバスの練習試合マッチングのアイデアを話すので、`ideas/` に企画メモとして保存して」 |
| 資料を作っておいてほしい | 「○○議員の過去の発言を議事録から集めて、出典つきで `reports/` にまとめておいて」 |
| PCの作業の続き | 「さっきの企画書、マネタイズの章を足しておいて」 |

> 💡 文字を打たなくてよい。Claude アプリの**入力欄にあるマイクボタン**を押して話せば、そのまま文字になる。

---

## 0. 全体マップ

### 🟢 全体の流れ（15分）

PC の Claude Code を**つけたまま**にしておき、スマホから操作する方法。PC にあるフォルダ・ファイルをそのまま使える。**最初に1回設定すれば、あとはいつもどおり Cursor で Claude を起動しておくだけ。**

```mermaid
flowchart LR
    S1["1.1<br/>スマホに<br/>Claudeアプリ"] ==> S2["1.2<br/>PCで<br/>常にオンにする"]
    S2 ==> S3["1.3<br/>出かける前に<br/>Claudeを起動"]
    S3 ==> S4["1.4<br/>スマホで<br/>セッションを開く"]
    S4 ==> S5["1.5<br/>話しかける"]
    S5 ==> G1["🎉<br/><b>外から<br/>頼める</b>"]

    style S1 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S2 fill:#3498DB,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S3 fill:#F4C430,color:#000,stroke:#333,stroke-width:2px
    style S4 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style S5 fill:#16A085,color:#FFFFFF,stroke:#333,stroke-width:2px
    style G1 fill:#27AE60,color:#FFFFFF,stroke:#333,stroke-width:3px
```

### このやり方と GitHub を使うやり方の違い

| | この資料（リモートコントロール） | [setup-web.md](setup-web.md)（Claude Code on the web） |
| --- | --- | --- |
| 作業する場所 | **自分の PC** | Anthropic のクラウド |
| PC | **つけたまま**にしておく | 閉じてよい |
| 使えるファイル | PC にあるもの全部 | GitHub に置いたものだけ |
| 準備 | アプリを入れて、設定を1回オンにするだけ | GitHub を使えることが前提 |
| おすすめの人 | **まずはこちら** | Git / GitHub を学んだ人 |

### どれを何回やる？

| 頻度 | やること | 節 |
| --- | --- | --- |
| **【一生に1回】** | スマホに Claude アプリを入れてログイン | 1.1 |
| **【一生に1回】** | PC でリモートコントロールを常にオンにする | 1.2 |
| **【出かける前に毎回】** | Cursor でプロジェクトを開き、Claude を起動したままにする | 1.3 |
| **【毎回】** | スマホの「Code」から PC のセッションを開いて話しかける | 1.4 / 1.5 |

---

### 1.1 スマホに Claude アプリを入れる 【一生に1回】

1. スマホに **Claude** アプリを入れる
   - iPhone: [App Store](https://apps.apple.com/us/app/claude-by-anthropic/id6473753684)
   - Android: [Google Play](https://play.google.com/store/apps/details?id=com.anthropic.claude)
2. **PC の Claude Code と同じアカウント**でログインする
3. 左上の **≡（メニュー）** を押し、**「Code」** があることを確認する

<img src="images/app-home-code.png" alt="Claudeアプリのメニュー。Codeの場所" width="300">

> ⚠️ ふだんのチャット画面とは別に「Code」がある。リモートコントロールは**「Code」の中**で使う。

---

### 1.2 PC でリモートコントロールを常にオンにする 【一生に1回】

1. Cursor でプロジェクトのフォルダを開き、ターミナルで **いつもどおり `claude` を起動**する
2. Claude の入力欄に次の1行を打って Enter

```text
/config
```

3. 設定の画面が開いたら、そのまま `remote` と打つ（設定が1行に絞り込まれる）
4. **「Enable Remote Control for all sessions」** の行を ↓ キーで選び、Enter で **`true`** にする
5. 行の右側が `true` になっていれば完了。**Esc** で設定の画面を閉じる

![/config で remote と打ち、Enable Remote Control for all sessions を true にする](images/rc-config.png)

> 💡 これで、**これから起動する Claude は全部、自動でスマホからつながる状態**になる。毎回コマンドを打つ必要はない。

---

### 1.3 出かける前に Claude を起動しておく 【出かける前に毎回】

1. Cursor で、**作業したいプロジェクトのフォルダ**を開く
2. ターミナルで `claude` を起動する（ふだんどおり）
3. **そのまま出かける**

> 💡 **このフォルダが Claude の作業場所になる。** 外から頼んだメモや資料は、このフォルダの中に保存される。

> ⚠️ **PC は閉じない・電源につないでおく。** ターミナルを閉じたり Cursor を終了したりすると、スマホからつながらなくなる。一時的なスリープやネットの切断なら、戻れば自動でつながり直す。

---

### 1.4 スマホでセッションを開く 【毎回】

1. スマホの Claude アプリで **「Code」** を開く
2. 「セッション」の一覧から、**緑のノートPCのアイコン**がついたものを探してタップする
   - アイコンの左の小さい文字が、1.3 で開いた**フォルダの名前**（例: `tech_docs · main`）

<img src="images/app-session-list.png" alt="Codeの一覧。ノートPCのアイコンがPCのセッション" width="300">

> 💡 **ノートPCのアイコン = 自分の PC で動いているセッション**。雲のアイコンは [setup-web.md](setup-web.md) のクラウドのセッションなので、ここでは使わない。

> 💡 一覧に出てこない・開いても返事がないときは、PC の Claude が止まっている（帰ってから 1.3 をやり直す）。

---

### 1.5 スマホから話しかける 【毎回】

1. 入力欄をタップ
2. 入力欄の右にある **マイクボタン** を押して話す（文字で打ってもよい）
3. 送信 → PC の上で Claude が作業し、結果がスマホに出る

<img src="images/app-voice-input.png" alt="入力欄のマイクボタン" width="300">

**最初はこれを送ってみる**（ちゃんとつながっているかの確認）:

```text
このフォルダに memo.md を作って、「スマホから書きました」と1行書いて
```

帰ってから PC の Cursor を見て、`memo.md` ができていれば成功。

<img src="images/part1-goal.png" alt="Cursorのファイル一覧にmemo.mdができている" width="400">

> 💡 ファイル名の右の **`U`** は「新しくできて、まだ Git に記録していないファイル」という印。

> 💡 **うまく頼むコツ**: 「どこに」「何を」「どんな形で」を入れる。
> - ❌「さっきのアイデアまとめて」
> - ⭕「`ideas/` フォルダに、ミニバスのマッチングアプリの企画メモを作って。課題・使う人・お金の取り方の3つの見出しで」

> ⚠️ Claude が「このファイルを変更してよいか」と聞いてくることがある。内容を読んで、よければ許可する。**分からないときは許可せず、帰ってから PC で確認**すればよい。

---

### 🎉 ゴール

- 最初に1回 `/config` でリモートコントロールを常にオンにしておけば
- 出かける前に Cursor で `claude` を起動しておくだけで
- 移動中にスマホの Claude アプリ →「Code」→ ノートPCのアイコンのセッションから話しかけるだけで
- PC のプロジェクトにメモや資料が残る

---

## 困ったとき

| こうなった | こうする |
| --- | --- |
| スマホの「Code」にセッションが出ない | PC の Cursor で `claude` が起動したままか確認。**PC とスマホで同じアカウント**か確認。1.2 の設定が `true` か確認 |
| 一覧にどうしても出ない | PC の Claude の入力欄で `/remote-control` と打つ。出てくる QR コードをスマホのカメラで写すと、そのセッションが開く |
| 「Remote Control は使えない」と出る | Claude の**有料プラン（Pro 以上）**で、`/login` からログインしているか確認 |
| 頼んだファイルが見つからない | 1.3 で開いたフォルダの中を探す。見つからなければスマホで「どこに保存した？」と聞く |

---

## 関連資料

- [mobile-workflow.md](mobile-workflow.md) — スマホ開発の考え方（何ができて、何は PC でやるか）
- [setup-web.md](setup-web.md) — PC を閉じていても使う方法（GitHub を使う）
- 公式: [Remote Control](https://code.claude.com/docs/en/remote-control) / [モバイル](https://code.claude.com/docs/en/mobile)
