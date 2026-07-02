# 選定プロセス全記録

**日付**: 2026-07-02
**手法**: multi-agent敵対的選抜パイプライン(61エージェント / 約123万トークン / 209ツール実行、Web一次情報照合を含む)

## 1. パイプライン設計

```
Ideate(8レンズ × 2構想 = 16構想、Web調査付き)
  → Dedup(問題空間の重複統合 — 重複ゼロで16構想全通過)
  → Judge(構想ごとに3人の敵対的審査員 = 48判定、加重: 収益性0.4 / 実現可能性0.3 / 市場0.3)
  → DeepDive(上位3構想に実地デューデリジェンス — 審査員の懸念への反証をWeb検証で要求)
  → Synthesize(勝者の実装計画をデューデリジェンス後の修正済み現実で作成)
```

**前提条件(全構想に共通で課した制約)**:
- ソロ(1–3人)の技術系ファウンダー、日本在住、外部資金なし
- 収益はグローバル(主にUS/EU)
- 3ヶ月で最初の有償顧客に到達できるwedgeがあること
- OpenAI/Anthropic/Googleの機能追加一発で死なないmoatを正直に記述すること

**審査員設計**: 各審査員は「まず構想を殺しにかかれ、steelmanは攻撃の後」と指示。fatal_flaw(そのままでは事業がほぼ確実に失敗する欠陥)を2つ以上宣告された構想は自動棄却。

## 2. リーダーボード(16構想・審査後加重スコア)

| 順位 | スコア | 構想 | レンズ | 一行概要 |
|-----|-------|------|--------|---------|
| 1 | 57.8 | **GutSignal** | consumer-sub | IBS/GERD向けAI gut coach。写真/音声の食事ログから個人トリガーマップを構築し、低FODMAP除去・再導入プロトコルを伴走 |
| 2 | 57.7 | **Yaku** | cross-border | RPG Maker/WOLF/Ren'Py/Unity直結のAI+人間認証ゲームローカライゼーションパイプライン(JP→EN / EN→JP) |
| 3 | 56.9 | **ToriiGate** | cross-border | 外国Amazon.co.jp/Shopifyセラー向け日本市場コンプライアンス自動化(JCT・輸入/IOR構造・申告) |
| 4 | 56.6 | MarginLens | devtools | AIプロダクトの粗利system of record — LLM/推論コストを顧客・機能・Stripe収益ラインに突合 |
| 4 | 56.6 | Fieldnote | prosumer | 独立戦略コンサル/DDブティック向け、15–40本のexpert call・顧客インタビューの統合ワークベンチ |
| 6 | 56.2 | SpeechSprout | consumer-sub | 子どもの構音を音素レベルで聴き取り、毎日10分の言語療法宿題セッションを実行するAIスピーチコーチ |
| 6 | 56.2 | Torii Data | marketplace | frontier AIラボへ審査済み日本語(→韓国語/CJK)ドメイン専門家(弁護士・医師・会計士・エンジニア)を供給するmanaged marketplace |
| 8 | 55.6 | VettingDeck | vertical-ai | SIRE 2.0船舶検査インテリジェンス — 中規模タンカー船隊をオイルメジャーのvetting審査に通すAI準備パック |
| 9 | 53.7 | Upgraid | devtools | テスト通過済みのメジャーバージョンアップグレードPR(コード修正込み)を継続的に出荷するGitHub App |
| 10 | 53.6 | PaidPilot | smb-backoffice | QuickBooks/Xero上のSMBサービス業向けAI債権回収エージェント(返信読解・紛争解決まで) |
| 10 | 53.6 | BidBench | prosumer | ブティックエージェンシー/ITサービス/コンサル(3–50人)向けAI RFP/入札応答ワークベンチ |
| 12 | 53.0 | AgentProof | agent-infra | 本番AIエージェントの改竄検知フライトレコーダー — 実行トレースを規制・調達対応のエビデンスパックに変換(EU AI Act) |
| 13 | 52.7 | TireKick | marketplace | $500k未満のオンライン事業買収(micro-SaaS・newsletter・EC)向け独立AIデューデリジェンス層 |
| 14 | 51.9 | ClaimHound | vertical-ai | 米国中堅輸入業者向けduty drawback(関税還付)請求自動化 — **fatal flaw 1件**(下記) |
| 15 | 51.4 | InvoiceRail | smb-backoffice | vertical SaaS向け開発者ファーストのe-invoicingコンプライアンスAPI(Peppol/各国網) |
| 16 | 46.8 | Fusebox | agent-infra | 本番AIエージェント艦隊向け、階層的予算上限とループ遮断ブレーカーのdrop-in enforcementゲートウェイ |

**fatal flaw棄却該当**: なし(2件以上該当ゼロ)。ClaimHoundのみ市場審査員からfatal flaw 1件 — 「self-serve中堅市場は空白」という前提が2026年時点で既に虚偽(Zollback、Tariff Refund HQ($4.5Mシード)等が参入済み)であり、かつ19 CFR 111により米国市民権のないファウンダーは規制上の当事者関係を保持できない。

## 3. 上位3構想のデューデリジェンス比較

deep-dive工程は「調査で構想が弱まるならconvictionを下げよ」と指示した敵対的診断。3構想とも**ピッチの規制/市場の物語は検証で裏付けられた一方、商業的な主張は調査が触れたほぼ全てで削られた**。

| | ToriiGate | GutSignal | Yaku |
|---|---|---|---|
| 審査スコア | 56.9 | 57.8 | 57.7 |
| **DD後conviction** | **46** | 45 | 44 |
| 検証で強まった点 | 規制の三段ロケットは全て一次情報で確認(2028年4月プラットフォーム課税、¥10,000免税撤廃、政府推計¥500B–1Tの未徴収JCT)。創業者適合(日本語一次情報×EN/CN GTM×エンジニアリング)の希少性 | カテゴリのWTPは想定以上(Nerva 28k有償者・UK年額~£149、Cal AI $30–50M収益でMyFitnessPal被買収、BayerがCara Care買収) | 問題テーゼは実在(JP indie Steamブーム、代理店価格はAIの3–10倍、DLsite「みんなで翻訳」¥370M流通が潜在需要を証明) |
| 検証で崩れた点 | 「英語SEOほぼ空白」は誇張(MailMate/SafariStar/ACP各社が先行)。Sphere($100/mo、a16z $21M)が英語対応税理士内製で日本JCT参入済み。中国語圏はRMB 3,100–10,000の価格傘で直販不能。Amazonの一斉suspension証拠なし(恐怖訴求は弱い) | 中核の「空白席」主張が虚偽化 — Triggerbites($39.99/yr)、IBS Pal(6グループ再導入プロトコル出荷済み)ほか2025–26年に少なくとも8本のクローンが同ポジションに参入。保険適用で米国の栄養士アンカー($600–1,500)も侵食 | 2大moat主張が実証的に崩壊 — エンジンパーサーはTranslator++が無料提供済み、「AIコスト×日本コミュニティ」はDMM GAME Translate(¥2/字、半年で50タイトル)が既占。Steam AI開示タグに約43–53%の売上ペナルティの実測研究 |
| DD後の現実的な姿 | 2026–2028の実在するアービトラージ。$300–600k ARR(粗利45–60%)+2028年以降はIOR/マルチチャネル/white-labelで成長 | 差別化された後発。12ヶ月$70–150k ARR、$1M ARRは3–4年。防御可能な唯一の角度はアジア料理FODMAP DB(日本市場) | 良質なproductized service。Year-1 $90–140k、Year-2 $300–500k、キャパ上限あり。ソフトウェア資産が複利しない |

## 4. 判断根拠 — なぜToriiGateか

conviction差は1–2点で、数字だけでは決まらない。決定打は**リスクの質**の差:

1. **需要の確実性**: ToriiGateの需要は法律の施行日(2028年4月1日)で確定している。GutSignalの需要は「クローン8本の中で発見されるか」という確率変数、Yakuの需要は「無料ツールとDMMの間で認証バッジに金を払うか」という未証明仮説。**規制は最良のセールスマンであり、納期を守る。**

2. **競合の構造的不在**: ToriiGateの間隙(goods/FBA/輸入特化 × 英語self-serve × Amazon SP-API統合)は、埋めるのに「日本語の一次規制文書を読める × 英中でGTMできる × ソフトウェアを書ける」の3条件が同時に要る。GutSignalの間隙はSwiftが書ける者なら誰でも埋められ、実際8チームが埋めた。Yakuの間隙はDMMが資本で埋めつつある。

3. **スイッチングコストの複利**: filing history(基準期間計算に過去2年データが必須)と輸入許可書の蓄積は時間とともに顧客を移動不能にする。GutSignalのトリガーマップにも同種の性質はあるがconsumerのchurnは構造的に高く、Yakuには蓄積資産がほぼない(翻訳メモリは持ち出し可能)。

4. **敗北時の残存価値**: ToriiGateのkill criteriaが発動しても、rules engine+SP-API集計+様式生成パイプラインは競合/代理店へのinfrastructure売却(white-label/API)という撤退路を持つ。他2構想の残存価値は薄い。

5. **リスクの既知性**: ToriiGate最大のリスク(2028年に純Amazon申告サブスクが縮小)は**公表済みで日付が確定しており、設計で対処可能**(実装計画のR1参照)。GutSignal最大のリスク(相関品質と発見可能性)とYaku最大のリスク(認証経済が成立しない)は、やってみるまで分からない。

**採用**: ToriiGate。実装計画は [`IMPLEMENTATION_PLAN.md`](IMPLEMENTATION_PLAN.md)。
**次点の保存**: GutSignal/Yakuの完全な調査記録と乗り換え条件は [`RUNNERS_UP.md`](RUNNERS_UP.md)。

## 5. この選定の限界(明示)

- 調査はWeb公開情報+モデル知識(cutoff 2026-01)に基づく。**顧客インタビュー0件** — 実装計画のM1–M2 design partner工程が最初の実地検証となる。
- conviction 46は絶対値として低い。これは「ソロ・無資金・24ヶ月$1M ARR」という制約が厳しいことの正直な反映であり、資金調達を許容する・チームを組む・期間を36ヶ月に延ばす、のいずれかで上位構想の順位は変わり得る(特にTorii Data・MarginLensはチーム前提なら再評価に値する)。
- 収益予測は全て仮説。実装計画のkill criteria(M4/M6/M9)が予測への規律を担保する。
