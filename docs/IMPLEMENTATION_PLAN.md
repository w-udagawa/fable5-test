# ToriiGate — マスター実装計画書

**版**: v1.0(2026年7月2日起点 / M1 = 2026年7月)
**前提**: ソロファウンダー(日本在住・日英中トライリンガル・エンジニア)、ブートストラップ、frontier AIツール活用、収益はグローバル(主にUS/EU)。
**設計思想**: 本計画はピッチ原案ではなく**デューデリジェンス後の修正版現実**に基づく。すなわち (1) 2028年4月プラットフォーム課税により純Amazon申告サブスクは約21ヶ月で構造的に縮小する、(2) 中国セラー直販はRMB 3,100〜10,000の価格傘により閉じている、(3) Sphere($100/mo, a16z)とAVASK–SimplyVAT統合が両側から収斂中、(4) 税理士法52条・通関業法により申告・通関行為は有資格パートナー専管 — の4点を所与として設計する。

---

## 1. エグゼクティブサマリー

**何を**: 外国EC事業者向け「日本市場コンプライアンスOS」。JCT(消費税)登録準備・申告カレンダー・申告データ自動集計(Amazon SP-API)・国税庁/税関文書のAIトリアージ・納税管理人/税関事務管理人(ACP)選任・**2028年プラットフォーム課税に向けたIOR(輸入者名義)再構築パッケージ**を、self-serveのWebアプリ+提携有資格者(税理士/ACP)ネットワークで提供する。ソフトウェアがデータ準備と進行管理を行い、署名・提出は必ず提携税理士/ACPが行う(税理士法52条準拠)。

**誰に**: Amazon.co.jp FBAで年商$200k–$5Mの外国セラー。直販は英語圏(US/EU)セラーに限定。中国語圏は深セン系代理店への**white-label提供**($39/entity/mo)で間接的に取る — 直販でRMB 3,100の価格帯と戦わない。

**なぜ今**: 規制の三段ロケット。(1) 2023年10月インボイス制度→AmazonがJCT登録番号(T番号)を要求、(2) FY2026税制改正(2025年12月発表)で¥10,000低額輸入免税の撤廃、(3) **2028年4月1日プラットフォーム課税開始** — Amazonが納税者になる一方、仕入税額控除(輸入JCT回収)はセラー自身がImporter of Recordである場合のみ。つまり全FBAセラーが2028年までにACP経由の輸入構造再編を迫られる。2026–2028年は確定した駆け込み期間であり、この一回性需要をサブスクの入口として使う。

**いくらで**: Starter $129/mo(Amazon専業・年次申告)/ Growth $349/mo(マルチチャネル・中間申告あり)+ onboarding $990、**2028 IOR Readiness Package $1,990–2,990(one-time)**、white-label $39/entity/mo。12ヶ月目標: 65–70 entities、月次収益$22–25k、Year-1グロス収益$110–140k(パートナー支払後ネット$60–85k)。$1M ARRは30–36ヶ月想定。2028年以降の成長はIOR/ACPライン+マルチチャネル(Shopify/Rakuten)+隣接領域(所得税・ラベリング)が担う。

**一行戦略**: 「申告代行のSaaS化」ではなく「**2028年輸入構造再編を錨とするコンプライアンスOS**」。申告サブスクは賞味期限付きと認めた上で、期限そのものを最大の営業材料にする。

---

## 2. プロダクト仕様

### 2.1 MVP機能リスト(M1–M3で出荷)

**必須(MUST)**

| # | 機能 | 仕様概要 |
|---|------|---------|
| 1 | **Guided JCT Registration Intake**(EN) | ウィザード形式(15分)。入力: 法人情報、基準期間/特定期間の日本売上、FBA在庫搬入日、決算期。出力: 課税義務判定+必要届出リスト+見積 |
| 2 | **NTA様式の自動下書き** | 「消費税納税管理人届出書」「適格請求書発行事業者の登録申請書(国外事業者用)」「税務代理権限証書」等を、様式PDFテンプレート+座標マッピング(pdf-lib)で記入済みPDF生成 → **必ず提携税理士レビューキューへ**。自動提出はしない |
| 3 | **納税管理人選任ワークフロー** | 提携税理士との三者契約をアプリ内e-signで締結。委任状・本人確認書類(登記簿・パスポート)の収集チェックリスト付き |
| 4 | **Filing Calendar Engine** | entity毎に基準期間・課税期間・中間申告回数(前期税額>¥48万/400万/4,800万で年1/3/11回)・法人は期末後2ヶ月、個人は3/31等の期限を自動算出。email+アプリ内リマインダー(D-60/30/14/7/3) |
| 5 | **Amazon SP-API連携** | LWA OAuthで接続。Reports API(注文レポート)+Finances API(手数料・返金)から日本marketplace売上を月次取込→課税売上集計(10%/8%区分、返品調整)→申告用ワークシート出力(税理士向けCSV/e-Taxソフト取込形式) |
| 6 | **Document Vault** | S3(SSE-KMS)。カテゴリ: 登録通知書、**輸入許可書(import permit — 2028年控除防衛のコア資産)**、インボイス、NTA通知。期限・entity紐付け |
| 7 | **Correspondence Triage** | NTA/税関からの郵送物をパートナー経由でスキャン→LLM OCR+分類(督促/照会/通知/要アクション)→英語サマリー+推奨アクション→税理士確認→顧客通知。SLA: 受領後2営業日 |
| 8 | **Filing Workflow State Machine** | `upcoming → data_collection → draft → partner_review → partner_approved → filed → payment_confirmed → closed`。チャットではなく状態遷移が製品(汎用LLMが構造的に踏み込めない領域) |
| 9 | **無料リードジェンツール×3** | (a) JCT納税義務カルキュレーター(基準期間ロジック内蔵)、(b) 2028 Platform-Tax Readiness Checker(「今のimport permitは誰の名義か」診断)、(c) 輸入JCT回収エスティメーター(月商入力→取り戻せる円額出力) |
| 10 | **Stripe Billing** | subscription + one-time(onboarding/package)。USD建て |
| 11 | **Ops Console** | founder+パートナー用ケース管理画面。manual-behind-the-curtainをここで回す。パートナーが直接ステータス更新できる設計(key-man緩和) |

**明示的に除外(MVPではやらない)**

- e-Tax直接電子申告(法的に税理士署名必須 — パートナーの既存e-Taxソフトで提出。当社はデータ受け渡しまで)
- 税務アドバイスを返すchatbot(税理士法2条・52条の無資格税務相談リスク。一般情報表示+「パートナーに確認」導線のみ)
- 中国語UI(white-label契約が固まるM6以降に、代理店要件として実装)
- Shopify/Rakutenコネクタ(M7–8)
- 関税計算・HSコード自動分類(ACPの専管領域。参考表示のみ)
- 法人税/所得税・労務・商標(2027年の隣接領域バックログ)
- モバイルアプリ、SOC 2(初年度はセキュリティ実務のみ)

### 2.2 ユーザージャーニー

**ジャーニーA — パニック新規(「AmazonからJCT番号を求めるメールが来た」)**
1. Reddit/検索 → LP or カルキュレーター → 義務判定「あなたは初回販売から課税事業者です」+未対応コスト円換算
2. サインアップ → intake wizard(15分)→ 見積提示 → onboarding $990決済
3. 書類アップロード(登記簿・パスポート・Amazonストア情報)→ 様式自動下書き生成
4. 提携税理士がレビュー・署名 → NTA提出 → T番号取得(通常1–2ヶ月)→ Seller Centralへ登録手順ガイド
5. サブスク定常運転: SP-API自動集計 → 申告期限前にdraft → partner_review → filed。顧客は英語ダッシュボードで全状態を見るだけ

**ジャーニーB — 2028再構築(既登録セラー)**
1. Readiness Checker → 「輸入許可書がforwarder/Amazon名義 → 2028年4月以降、輸入JCTの控除が消失。年間損失推定 ¥X」
2. IOR Readiness Package購入 → ACP(税関事務管理人)選任 → 輸入名義の切替(forwarder/通関業者への指示書テンプレート込み)
3. import permitがVaultに自動蓄積 → 控除証憑が揃った状態で2028年を迎える → Growthサブスクへアップセル

**ジャーニーC — white-label(中国代理店)**
代理店が自社ブランドの中国語UIで顧客entityを登録 → 当社のrules engine・様式生成・カレンダー・SP-API集計を利用 → 申告は代理店側の提携税理士 or 当社パートナー(選択制)。

---

## 3. システムアーキテクチャ

### 3.1 技術スタック(ソロで3ヶ月出荷を最優先)

| レイヤー | 選定 | 理由 |
|---------|------|------|
| Frontend/BFF | **Next.js 15 (App Router) + TypeScript + Tailwind + shadcn/ui** | ソロ最速。LP・ツール・アプリを単一repoで |
| API | Next.js server actions + **tRPC**(内部)/ REST(webhook) | 型安全、スキーマ二重管理回避 |
| DB | **PostgreSQL 16(AWS RDS, ap-northeast-1)+ Drizzle ORM** | 書類とデータは日本リージョン(APPI・顧客心理) |
| Auth | Clerk(email+Google、**2FA必須**) | 自前実装しない |
| Storage | S3(SSE-KMS、pre-signed URL、versioning) | 書類vault |
| Jobs/Queue | **Inngest**(cron+event駆動) | SP-APIレポートポーリング、リマインダー、LLMパイプライン。自前Redis運用を避ける |
| PDF | pdf-lib + 様式テンプレートJSON(座標マップ) | NTA様式は固定レイアウト。様式改定はテンプレート差し替えのみで対応 |
| Billing | Stripe(subscription + invoice) | |
| LLM | **Claude API**(Sonnet標準/難案件のみOpus) | 下記3.3 |
| Infra | Vercel(front)+ AWS(RDS/S3/Inngest cloud)、IaC: Terraform最小構成 | |
| 監視 | Sentry + Axiom(log)+ Better Uptime | |

### 3.2 データモデル概要(コアテーブル)

```
organizations(id, name, country, locale, referral_source)
users(id, org_id, role, email, locale)
entities(id, org_id, legal_name, entity_type[corp/individual],
         home_country, fiscal_year_end, jct_registration_no,
         taxable_status, base_period_sales JSONB, ior_status)
partners(id, type[zeirishi/acp], firm_name, capacity_max, capacity_used, sla_terms)
partner_assignments(entity_id, partner_id, role, non_solicit_signed_at)
marketplace_connections(entity_id, channel[amazon_jp/shopify/rakuten],
                        credentials_ref, status, last_sync_at)
sales_periods(entity_id, period_ym, gross_sales_std10, gross_sales_red8,
              refunds, fees, source, locked_at)
filings(entity_id, type[jct_annual/jct_interim/registration/notification],
        period_start, period_end, due_date, state, partner_id, filed_at)
filing_line_items(filing_id, code, amount_jpy, source_ref)
documents(entity_id, category[import_permit/registration/invoice/nta_notice/...],
          s3_key, ocr_json, expires_at)
correspondence_cases(entity_id, received_at, classification, llm_summary_en,
                     action_required, state, sla_due)
deadlines(entity_id, rule_id, due_date, status)
regulatory_rules(id, domain, effective_from, effective_to, payload JSONB, source_url)
audit_log(actor, entity_id, action, before, after, ts)  -- 全書き込みに必須
```

**設計上の要点**:
- `regulatory_rules` は**宣言的データとしてバージョン管理**(effective_date付きYAML→DB)。中間申告回数・様式・期限ロジックをコードから分離し、税制改正をPR一本で反映する。**これがmoatの実体**なので初日からこの形にする。
- `partners.capacity_max` を最初から持つ。1税理士事務所の現実的上限は150–300 entity — パートナールーティングはスケーラビリティの本丸であり、後付け不可。
- `documents.category=import_permit` は2028年ビジネスのデータ資産。**多年度のfiling history+import permit蓄積がスイッチングコスト**(基準期間計算に過去2年データが必要 — 移行すると再収集地獄)。
- 個人情報(パスポート等)はfield-level encryption、audit_logは全変更に強制。

### 3.3 LLM/API構成と原価試算

| パイプライン | モデル | 処理 | 月間/entity | 概算原価 |
|------------|-------|------|------------|---------|
| Correspondence OCR+triage | Sonnet(vision) | 郵送物スキャン→分類→EN要約 | 〜10通 ×(5k in / 1k out) | 〜$0.30 |
| 様式下書き生成 | Sonnet | intakeデータ→様式フィールド値+和文備考 | 登録時のみ | 〜$0.10 |
| サポート下書き(EN/CN/JP) | Sonnet | 問い合わせ→回答draft(人間承認制) | 〜30通 | 〜$0.50 |
| Regulatory Watch | Sonnet+週次cron | 国税庁/税関/財務省ページ・通達のdiff→変更要約→founderレビュー→rules更新PR | 固定費 | 〜$50/mo全体 |
| **合計** | | | | **<$3/entity/mo**(retry・Opusエスカレーション込み) |

インフラ固定費: 〜$250/mo(RDS+S3+Vercel+監視)、100 entity時点で$500–800/mo。**単位経済はcomputeではなくパートナー報酬に支配される**(4章)。モデルが賢くなるほどCOGSは下がる — AIは仕入れであり、moatはrules engine・パートナー網・データ蓄積側に置く。

**ハードコンストレイント(法務)**: 生成物はすべて「draft」ラベル+partner_reviewゲートを通過しないと顧客に確定提示されない(state machineで強制)。マーケ文言は「tax return preparation software + licensed partner network」で統一し、「tax advice」を謳わない。利用規約に責任制限、**E&O保険(年¥300–500k)をM6までに付保**。

### 3.4 スケーリング方針
- 技術: モノリスで1,000 entityまで問題なし。スケール課題は**人間側** — パートナー容量ダッシュボード、ケースSLA計測、パートナー3–5社への分散(M7に2社目、M12に3社目)。
- 国際化: i18nは初日からnext-intl。EN先行、CN(white-label)、JPは管理画面のみ。
- 冪等性: SP-API取込・請求・リマインダーはすべてidempotency key必須(申告データの二重計上は事故)。

---

## 4. 収益化設計

### 4.1 料金プラン表

| プラン | 価格 | 対象 | 含むもの |
|-------|------|------|---------|
| **Starter** | **$129/mo** + onboarding $990 | Amazon.co.jp専業、年次申告(中間1回まで) | JCT登録準備、納税管理人、カレンダー、SP-API集計、年次申告(パートナー申告料込み)、correspondence対応 |
| **Growth** | **$349/mo** + onboarding $990 | マルチチャネル or 中間申告年3回以上 | Starter全部+Shopify/Rakuten集計、四半期ペース申告prep、優先SLA |
| **2028 IOR Readiness Package** | **$1,990–2,990 one-time**(複雑度で見積) | 既存FBAセラー全員 | 現行輸入構造診断、ACP選任、IOR名義切替の実務指示書、import permit回収体制構築 |
| **White-label** | **$39/entity/mo**(最低20 entity・年間契約) | 中国系代理店 | rules engine+様式生成+カレンダー+SP-API集計のOEM。申告は代理店側資格者 |
| Add-on | 追加チャネル $49/mo、過年度遡及対応は見積制 | | |

### 4.2 単位経済(Starter、¥150/$想定)

| 項目 | Year 1 | Year 2+ |
|------|--------|---------|
| 収益 | $990 + $129×12 = **$2,538** | $1,548 |
| 税理士報酬(登録¥30k+年次申告¥90k) | −$800 | −$600 |
| LLM+infra | −$40 | −$40 |
| 決済3% | −$76 | −$46 |
| **粗利** | **$1,622(64%)** | **$862(56%)** |

- パートナー報酬が¥120k側に振れると粗利率は45–50%へ低下 — **粗利45–60%のservices-marketplace的構造を直視し、SaaSの85%を装った計画は立てない**。
- Package粗利: ACP実費pass-through後 〜50%($1,000–1,500/件)。white-label粗利 〜85%(申告COGSなし)— **スケールした時に最も美しいのはwhite-labelライン**。
- Churn 3%/mo想定(申告履歴・基準期間データ・納税管理人切替の痛みで構造的に低い)→ 平均寿命〜33ヶ月 → LTV粗利 〜$2,600(Starter)。CAC目標 <$500(content/community主体)→ **LTV/CAC > 5x**。
- 円安はUSD建て価格×円建てパートナー報酬なのでマージン追い風。年1回USD価格改定条項を規約に。

### 4.3 価格の根拠
- **上限アンカー**: 英語対応税理士 ¥150k–500k/yr(バイリンガル・プレミアム50–150%は実証済み)— Starter年額$2,538はこの帯の下限で、UX・可視性・SP-API自動集計は圧倒的に上。
- **下限アンカー**: Sphere $100/mo(ただしdigital services限定、goods/輸入非対応)。$129はSphereの真上に置き、値下げ競争せず「goods/FBA/import対応」で差額を正当化。
- **戦わない価格帯**: 中国系RMB 3,100–10,000オールイン。直販撤退、white-label $39で代理店の原価側に入る。
- **緊急性と利幅はPackageに集約**: 一回性の再構築作業を月額に溶かさず一回性で課金する(2028年に月額が萎んでも収益が立つ構造)。

---

## 5. GTM計画 — 最初の100顧客(英語セグメント直販+white-label)

### 5.1 チャネル優先順位

1. **Interactive free tools(SEOの主兵装)**: 「Japan JCT guide」系はSafariStar/MailMate/AVASK/ACP各社で既に飽和 — ガイドでは勝てない。**計算機で勝つ**: JCT義務カルキュレーター/2028 Readiness Checker/輸入JCT回収エスティメーター。既存ガイド勢からのbacklink獲得先にもなる。ロングテールLP(例: "Amazon asked for my JCT number — what to do")を週1枚。
2. **コミュニティ**: r/FulfillmentByAmazon・r/AmazonSellerのJapan taxスレは**毎週パニック投稿が再生産される** — 全スレに実名で最良回答(宣伝なし、プロフィールにリンク)。ASGTG listserv。EcomCrew/My Amazon Guy/Seller Sessionsに「the Japan compliance guy」としてポッドキャスト出演売り込み。
3. **パートナーシップ**: 日本レーンのfreight forwarder/prep center(Forceget、Flexportエコシステム等)にreferral $200/entity — 彼らは**IOR意思決定の瞬間**に顧客の目の前にいる。Amazon SPN(Seller Central Partner Network)tax servicesへ掲載申請(M2)。
4. **Outbound**: Amazon.co.jpストアフロント(特定商取引法ページ)をスクレイピング→外国住所×インボイス登録番号非表示のセラーを抽出。メール文面は申告売り込みではなく**「2028年4月以降、Amazonがあなたの消費税を納めます。しかし輸入時に払った税金は、あなた自身が輸入者名義でない限り戻りません — 現在の構造を10分で無料診断」**。死にゆく商品(filing)ではなく伸びる商品(restructuring)で当てる。
5. **White-label(中国セグメント、M6着手)**: AMZ123・知無不言で日本向けサービスを掲げる中堅深セン代理店をリストアップ→「日本ソフトウェアを持たない代理店」2–3社とOEM契約。直接WeChat GTMはしない。

### 5.2 週次アクション(M3以降の定常オペ、founder週50h中GTM約15h)

| 曜日 | アクション | KPI |
|------|-----------|-----|
| 月 | コミュニティ回答5件(Reddit/ASGTG)、前週スレの追跡 | 回答→プロフィール流入 |
| 火 | Outbound 50通(スクレイプ済みリスト、2028アングル) | 返信率>3% |
| 水 | コンテンツ/ツール改善1本(LP新規 or カルキュレーター精度) | organic訪問 |
| 木 | forwarder/prep centerアウトリーチ3件+podcastピッチ1件 | 提携パイプ |
| 金 | onboarding/サポート、metrics review(訪問→email capture 8%目標、email→有料10%目標) | CVR |

### 5.3 ペーシング(現実値)
M3: 累計3–5(design partner転換込み)→ M6: 15 → M9以降: 月8–10新規 → M12: 65–70。100顧客到達はM14–16見込み。これより速い計画は立てない(速く行けたら投資でなく利益)。

---

## 6. 12ヶ月ロードマップ(M1 = 2026年7月)

| 月 | マイルストーン | 累計entity | 月次収益目標 |
|----|--------------|-----------|------------|
| **M1** 2026-07 | 税理士1社+ACP1社とLOI→**本契約**着手(non-solicit・SLA・料金表込み)。rules engine v0、SP-API developer申請、LP+JCTカルキュレーター公開、法人設立/口座/Stripe | 0 | $0 |
| **M2** 08 | MVP中核(intake wizard・様式autofill・vault・calendar)。design partner 3社募集(初年度50% off)。SPN掲載申請。**税理士本契約締結(必達)** | 0(design 3) | $0 |
| **M3** 09 | billing on・**初有償顧客3件**・correspondence pipeline稼働・コミュニティローンチ | 3 | $3.5k(onboarding中心) |
| **M4** 10 | outbound開始(リスト2,000件)、2028 Readiness Checker公開 | 6 | $2.5k |
| **M5** 11 | SP-API集計→申告ワークシートv1(年末中間申告対応)、podcast初出演 | 10 | $4k |
| **M6** 12 | **2028 IOR Package販売開始(初件)**、white-label協議開始、E&O保険付保 | 15 | $6k |
| **M7** 2027-01 | 12月決算entityの申告繁忙を無事故で回す(オペの実証)。**2社目税理士LOI** | 21 | $8k |
| **M8** 02 | Shopify connector beta、Package月2件ペース | 28 | $10k |
| **M9** 03 | 個人事業主3/31申告期限の繁忙。SEOツールのcompounding確認(organic>40%) | 36 | $13k |
| **M10** 04 | white-label 1社目稼働(+20 entityパイプ)、2社目税理士稼働 | 45 | $16k |
| **M11** 05 | Rakuten対応調査、Package月3件、隣接領域(所得税/ラベリング)の需要検証インタビュー10件 | 55 | $19k |
| **M12** 06 | **65–70 entity、月次$22–25k(サブスクMRR $12–13k+onboarding/Package)**。2028ピボット比率レビュー(下記) | 65–70 | $22–25k |

**Year-1合計**: グロス$110–140k、パートナー支払後ネット$60–85k。**2027年末チェックポイント**: MRRの40%以上をnon-pure-Amazon-filing(Package/マルチチャネル/white-label)由来にする — 未達なら2028年に絶壁。

### Kill criteria / ピボット基準(事前コミット)

| 条件 | 判定時期 | アクション |
|------|---------|-----------|
| 税理士**本契約**が未締結 | M4末 | 法的にビジネス不成立 → **撤退**(LOIでは継続不可) |
| 有償顧客0 | M4末 | メッセージ/価格pivot 1回のみ許容 |
| 累計<8 entity or CAC>$1,500が3ヶ月継続 | M6末 | チャネル全面見直し(outbound+partner onlyへ) |
| 累計<20 entity or MRR<$4k | M9末 | **撤退検討ライン**(ネット収益がfounder機会費用を下回る) |
| ツール→signup CVR<2%が3ヶ月継続 | 随時 | SEO投資停止、referral/outboundへ全振り |
| SphereまたはAVASKがgoods+IOR self-serveを≦$150/moでローンチし、win rate<20% | 随時 | **infrastructure pivot**: rules engine+SP-API pipeline+様式生成を競合/代理店にAPI/white-labelで販売 |
| 2028年政省令でセラーIOR要件が実質消滅 | 公布時 | Packageライン前提崩壊 → マルチチャネル申告+隣接領域へ全振り、Package販売即停止(評判防衛) |

---

## 7. リスク登録簿

| # | リスク | 確率 | 影響 | 緩和策 |
|---|-------|------|------|--------|
| R1 | **2028年4月プラットフォーム課税で純Amazon申告サブスクが構造的に縮小**(確定事項) | 確実 | 致命的(放置時) | 収益重心を初日からIOR Package・マルチチャネル・white-labelへ。KPI: 2027年末 non-pure-Amazon比率40%。申告サブスクは「入口商品」と定義 |
| R2 | 税理士法52条/通関業法違反(無資格税務代理・税務相談) | 中 | 致命的 | 全成果物にpartner_reviewゲート(state machineで技術的に強制)、マーケ文言の弁護士レビュー(M2)、chatbotアドバイス機能を作らない、E&O保険 |
| R3 | パートナー依存: 容量上限(150–300 entity/社)・離反・**disintermediation(パートナー自身が最有力競合化)** | 高 | 高 | 3–5社分散(M7に2社目)、non-solicit条項、顧客契約・データ・課金はToriiGate側に保持、filing history lock-inで顧客の移動コストを当社側に蓄積 |
| R4 | Sphere(a16z、$100/mo、英語対応税理士内製)がgoodsへ拡張 | 中〜高 | 高 | 彼らが持たない資産を先に積む: SP-API申告集計、import permit vault、ACP網。価格追随はしない。最悪時はinfrastructure pivot(kill criteria参照) |
| R5 | AVASK–SimplyVAT統合+self-serveティア | 中 | 中 | 統合直後の混乱期に「Japan-only深度+価格透明性」で刈り取る。彼らにとって日本は60ヶ国の1つ、当社にとって日本が製品のすべて |
| R6 | **Enforcement softness**: Amazonの systematic suspension証拠なし、TAM大半が非遵守を選択中 | 高 | 高(CVR低下) | 恐怖訴求を捨て**金額訴求**へ: B2B売上逸失+2028年輸入JCT控除喪失を円で見せる(estimator)。「罰」ではなく「取り戻せる金」を売る |
| R7 | 中国系価格傘(RMB 3,100〜)で beachhead の半分が直販不能 | 確実 | 中 | 直販断念済み。white-label $39/entityで代理店の原価側に入る |
| R8 | 英語SEOが既に混雑(MailMate/SafariStar/ACP各社) | 確実 | 中 | ガイドではなくinteractive toolsで差別化。ガイド勢はreferral提携候補に転換 |
| R9 | ソロfounderのkey-man(繁忙期の申告事故・休暇不能) | 高 | 高 | ops consoleでパートナーが直接ケース処理可能な設計、全手順runbook化、M9以降パートタイムops(日本語可)1名検討 |
| R10 | NTA/税関の実務変更への追随漏れ(rules engine陳腐化) | 中 | 高 | Regulatory Watch pipeline(週次自動diff+founderレビュー)、パートナーとの月次改正レビュー会を契約に明記 |
| R11 | 賠償リスク($500k+/yrセラーのアカウント/控除に影響) | 低 | 高 | E&O保険(M6)、規約の責任制限、全申告物はパートナー署名(責任の法的所在を資格者側に) |
| R12 | 為替(円高でパートナー報酬のUSDコスト増) | 中 | 低 | 年次USD価格改定条項、粗利計算は¥140/$で保守的に検証済み |

---

## 8. 競合対応

| 競合 | 実態(検証済み) | 当社の差別化 | 維持戦略 |
|------|---------------|-------------|---------|
| **AVASK–SimplyVAT**(統合、420+名、self-serveティア開始) | サービス第一、日本は60+ヶ国の1つ、価格不透明(〜£1,200–2,500/yr/国) | Japan単一特化の深度、SP-API自動集計、公開価格、ACP/IORまで一気通貫 | 統合混乱期(2026)に比較LP・乗換オファー。「Japan is 1 of 60 for them; Japan IS the product for us」 |
| **Sphere**($100/mo、YC/a16z $21M、英語税理士内製、日本JCT対応) | **最大の脅威**。ただし現在digital services限定 — goods/FBA/輸入は通関・IOR・輸入消費税という別次元 | goods/FBA/import特化、Amazon統合、import permitデータ資産 | 彼らの参入前にACP提携網とpermitデータを押さえる。価格追随せず$129を維持。参入したらkill criteria発動→infrastructure pivot(下層を売る側に回る) |
| **中国系代理店**(J&P、欧税通 他、RMB 3,100–10,000オールイン) | 中国語圏を価格で完全支配、ソフトウェアなし | 競合ではなく**チャネル**と再定義 | white-label OEM($39/entity)。彼らの営業力×当社のrules engine |
| **ACP/IORファーム**(SK Advisory、ACP Japan 等) | **最重要frenemy**: 2028コンテンツで英語SEO上位、ライセンス保有、ただしソフトウェア/セラー向けworkflowなし。当社の不可欠なパートナーであり最も信憑性ある将来競合 | セラー側demand aggregation(顧客関係・Amazonデータ・多年度filing history)は当社が保持 | 複数社と非独占提携で単独依存回避、revenue share設計で「当社経由が最も楽で儲かる」状態を維持、顧客契約は必ずToriiGate名義 |
| **MailMate/SafariStar等の英語コンテンツ勢** | ガイドSEOで先行、self-serveプロダクトなし | interactive tools+実プロダクト | 正面のSEO消耗戦を避け、referral提携先に転換打診 |
| **Anrok/Commenda/Fonoa** | digital services JCTのみ | goods非対応のまま静観見込み | 監視のみ |

**差別化の恒久メカニズム(順に強い)**:
1. **データlock-in**: 基準期間計算には過去2年の売上データ、2028年控除防衛には輸入許可書の完全な蓄積が必要 — 乗換=データ再収集の苦役。顧客の在籍期間が長いほど移動不能になる(長期データ資産による最強のスイッチングコスト)。
2. **Rules engineの更新速度**: 日本語一次情報(通達・政省令・税関実務)を毎週diffして製品に反映する運用そのもの。US競合には言語と優先度の両面で構造的に追随不能。
3. **パートナー網+certified badge**: 「Licensed-partner-certified」バッジを全成果物に付す — AI税務ツールへの不信に対する信頼インフラであり、OpenAI/汎用AIが構造的に持てないもの(申告の法的責任主体になれない)。
4. **チャネルの非対称**: 日本の税理士は英語セラーに届かず、グローバル勢は日本を後回しにし、中国代理店はソフトを作らない。三者の間隙=当社のポジションであり、この非対称こそがアービトラージ。競合の動きは四半期毎に本登録簿とkill criteriaに照らしてレビューする。

---

**明日からの着手順(engineering)**: (1) monorepo scaffold(Next.js+Drizzle+Clerk+Stripe)、(2) `regulatory_rules`スキーマとJCT期限計算エンジン+テスト(中間申告回数・基準期間ロジック)、(3) NTA様式3種のPDF座標テンプレート化、(4) SP-API developerアカウント申請(審査リードタイムが最長のため即日)、(5) JCTカルキュレーターLP公開 — 並行してM1の税理士・ACP契約交渉を最優先で進める。契約なくして製品なし。