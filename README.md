# ToriiGate — 収益化アプリ構想と実装計画

> 外国EC事業者向け「日本市場コンプライアンスOS」。Amazon.co.jp FBAで販売する外国セラー(年商$200k–$5M)に対し、JCT(消費税)登録・申告データ自動集計・国税庁/税関文書のAIトリアージ・**2028年プラットフォーム課税に向けたIOR(輸入者名義)再構築**を、self-serveのWebアプリ+提携有資格者(税理士/ACP)ネットワークで提供する。英語・中国語で販売し、収益はグローバル。

## 結論(TL;DR)

| 項目 | 内容 |
|------|------|
| **プロダクト** | ToriiGate — Japan market-entry compliance autopilot for foreign e-commerce sellers |
| **顧客** | Amazon.co.jp FBAの外国セラー(直販は英語圏、中国語圏はwhite-label経由) |
| **価格** | Starter $129/mo + onboarding $990 / Growth $349/mo / 2028 IOR Package $1,990–2,990(one-time)/ white-label $39/entity/mo |
| **Year-1目標** | 65–70 entities、月次収益$22–25k、グロス$110–140k(パートナー支払後ネット$60–85k) |
| **$1M ARR** | 30–36ヶ月想定(400 entities) |
| **なぜ今** | 2023年インボイス制度 → FY2026税制改正(低額輸入免税撤廃)→ **2028年4月プラットフォーム課税開始**という規制の三段ロケット。2026–2028年は確定した駆け込み期間 |
| **moat** | AIではなく:日本語一次情報を毎週反映するrules engine、税理士/ACP提携網、filing history+輸入許可書のデータlock-in、EN/CN/JPの三言語チャネル非対称 |

## 選定プロセス

この構想は単一の思いつきではなく、multi-agentの敵対的選抜パイプラインの生存者である:

1. **Ideate** — 8つの市場レンズ(AIエージェント基盤 / 規制産業vertical AI / SMBバックオフィス / 開発者ツール / プロシューマー / コンシューマーサブスク / クロスボーダー / マーケットプレイス)で各2構想、計**16構想**をWeb調査付きで独立生成
2. **Judge** — 各構想を3人の敵対的審査員(収益性CFO視点 / 技術実現性 / GTM・競争環境)が「まず殺しにかかる」前提で採点(計48判定)
3. **DeepDive** — 上位3構想(GutSignal・Yaku・ToriiGate)に対し、審査員の懸念点への反証を要求する実地デューデリジェンス(競合・価格・流通のWeb検証)
4. **Synthesize** — 勝者についてデューデリジェンス後の修正済み現実に基づく実装計画を作成

計61エージェント、約123万トークン、209回のツール実行(Web検索・一次情報照合を含む)。

## 正直な評価

敵対的デューデリジェンス後のconviction scoreは **46/100**(次点GutSignal 45、Yaku 44)。これは「全候補が凡庸」という意味ではなく、**ソロ・ブートストラップで24ヶ月$1M ARRを確実に達成できる構想は市場に転がっていない**という調査結果の正直な反映である。ToriiGateの実像は:

- **venture-scaleの賭けではなく、粗利45–64%のservices-marketplace構造を持つ堅実なニッチ事業**(計画書はSaaS 85%粗利を装わない)
- 純Amazon申告サブスクには**2028年4月という公表済みの賞味期限**があり、計画はこれを隠さず「最大の営業材料」として設計に織り込む(IOR Package・マルチチャネル・white-labelへの重心移動をKPI化)
- 勝因は市場の大きさではなく**創業者適合の希少性**:日本語一次情報の読解 × 英中でのGTM × エンジニアリング — AVASK/Sphere/中国系代理店/日本の税理士の誰も同時に持たない組み合わせ

## ドキュメント構成

| ファイル | 内容 |
|---------|------|
| [`docs/IMPLEMENTATION_PLAN.md`](docs/IMPLEMENTATION_PLAN.md) | **マスター実装計画書** — MVP仕様、システムアーキテクチャ、データモデル、LLM原価試算、料金設計、単位経済、GTM週次アクション、12ヶ月ロードマップ、kill criteria、リスク登録簿、競合対応 |
| [`docs/SELECTION.md`](docs/SELECTION.md) | **選定プロセス全記録** — 16構想のリーダーボード、審査設計、上位3構想のデューデリジェンス比較、ToriiGate選定の判断根拠 |
| [`docs/RUNNERS_UP.md`](docs/RUNNERS_UP.md) | **次点2構想の完全な調査記録** — GutSignal(IBS向けAI gut coach)とYaku(ゲームローカライゼーション)。前提が変わった場合の乗り換え条件付き |
| [`.claude/skills/venture-discovery/`](.claude/skills/venture-discovery/) | **このパイプラインのスキル化** — 発掘→敵対的審査→ファクトチェック→計画→批評改訂の再実行可能なワークフロー(`SKILL.md` 実行手順 / `workflow.js` 本体 / `DESIGN.md` 弱モデル向け設計原則) |

## 明日からの着手順

1. monorepo scaffold(Next.js 15 + Drizzle + Clerk + Stripe)
2. `regulatory_rules`スキーマとJCT期限計算エンジン+テスト(中間申告回数・基準期間ロジック)
3. NTA様式3種のPDF座標テンプレート化
4. Amazon SP-API developerアカウント申請(審査リードタイムが最長のため即日)
5. JCTカルキュレーターLP公開

**並行して最優先**: 提携税理士1社+ACP1社との本契約交渉。**契約なくして製品なし**(M4末までに本契約未締結なら撤退 — kill criteria参照)。
