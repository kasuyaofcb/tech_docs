# tech_docs

技術に関する**全体像・概要**をまとめた資料リポジトリ。

細かいAPIリファレンスやハウツーではなく、「これは何で、どう繋がっているのか」を図解中心で掴むことを目的とする。

---

## 📚 資料一覧

| # | トピック | 内容 | 資料 |
| --- | --- | --- | --- |
| 01 | **アプリリリース全体像** | **最初に読むやつ**。ローカル/Git/Supabase/Vercel/Azureがどう繋がるかの地図 | [01_app-release/app-release.md](01_app-release/app-release.md) |
| 02 | Git | バージョン管理の全体像（作業ディレクトリ → ステージング → ローカル → リモート） | [02_git/git.md](02_git/git.md) |
| 03 | Supabase | BaaSの全体像（Postgres中心にAuth / Storage / Realtime / Edge Functions / RLS） | [03_supabase/supabase.md](03_supabase/supabase.md) |
| 04a | Vercel | フロントエンドCloudの全体像（Git連携デプロイ / Preview URL / Functions / 環境変数） | [04a_vercel/vercel.md](04a_vercel/vercel.md) |
| 04b | Azure | 総合クラウドの全体像（テナント/サブスク階層 / Static Web Apps / CI-CDの安全弁 / 個人開発→本格運用） | [04b_azure/azure.md](04b_azure/azure.md) |
| 04b' | Azure 環境構築手順 | アカウント開設→デプロイ完了までの一連の流れ（CI/CDは GH Actions / Azure Pipelines、DBは Supabase / Azure SQL / PostgreSQL から選択） | [04b_azure/setup.md](04b_azure/setup.md) |

> 📖 **推奨学習順**: 01 → 02 → 03 → 04a または 04b（配信先は二者択一）
>
> 今後追加予定: Next.js / Docker / 認証まわり など

---

## 🎯 このリポジトリの方針

- **全体像ファースト**: まず1枚の図で構造を掴めるようにする
- **図解中心**: Mermaidでフローチャート・シーケンス図を活用
- **よくある勘違い**を明示: 正しい挙動と並べて誤解を解く
- **手を動かす前に読む資料**: コマンド集ではなく概念の整理

---

## 🗂 ディレクトリ構成

```text
tech_docs/
├── README.md         ← この資料（index）
└── <topic>/
    └── <topic>.md    ← トピックごとの全体像資料
```

新しいトピックを追加するときは、`<topic>/<topic>.md`の形でフォルダーを切る。

---

## ➕ 資料を追加するときの目安

1. **1枚目の図で全体像がわかる**こと
2. 各要素の **役割と関係性** を言語化する
3. **誤解されやすいポイント** を「勘違い vs 実際」で並べる
4. 詳細仕様への深入りは避け、公式ドキュメントへのリンクで補う

---

## 📄 ライセンス

本リポジトリの資料は [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) の下で公開しています。

出典を明記すれば、改変・再配布・商用利用も自由です。詳細は [LICENSE](LICENSE) を参照してください。
