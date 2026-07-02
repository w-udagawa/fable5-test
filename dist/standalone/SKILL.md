---
name: venture-discovery
description: 収益化可能なアプリ/事業構想の発掘・敵対的審査・ファクトチェック・実装計画作成を行うマルチエージェント・パイプライン。ユーザーが「儲かるアプリ/事業を考えて」「アイデアを探索して検証して」「この事業案をデューデリジェンスして」等を求めた時に使用。Workflowツールで多数のサブエージェントを起動する(呼び出し自体がオプトインに相当)。トークン消費が大きい(standardで100万トークン超)ため、軽い相談・単発の壁打ちには使わないこと。この単一ファイルで完結する(末尾付録にワークフロー本体を同梱)。
---

# venture-discovery — 事業構想の発掘・検証・計画化パイプライン(単一ファイル版)

多数のサブエージェントで「構想生成 → 敵対的審査 → 深掘り調査 → ファクトチェック → 実装計画 → 批評・改訂」を決定的に実行する。**あなた(オーケストレーター)の仕事は、入力を集め、ワークフローを起動し、結果をドキュメントに落とすことだけ**。判断ロジックは付録のワークフロースクリプト側に実装済みなので、勝手に工程を省略・代替しないこと。

> このファイルは claude.ai へ単体登録できるよう自己完結している。実行に必要な `workflow.js` は末尾の【付録】に同梱してあり、Step 2 でディスクへ書き出してから起動する。

## 前提の確認(実行できる環境か)

- **エージェント実行モードであること**: 本スキルは `Workflow` ツールとサブエージェント起動を使う。Claude Code(web/CLI/IDE)のセッションで実行可能。Workflow の無い会話専用モードでは実行できない — その場合はユーザーにその旨を伝えて中止する。
- **新規セッションで登録済みであること**: スキルはセッション開始時に登録される。登録直後の既存セッションでは効かないことがある。

## 実行手順

### Step 1: 入力を確定する

ユーザーの発言から以下を埋める。明示されていない項目はデフォルトを使い、**質問で作業を止めない**(ファウンダー情報が全く無い場合のみ1回だけ確認してよい)。

| args キー | 内容 | デフォルト |
|-----------|------|-----------|
| `today` | 今日の日付 YYYY-MM-DD。**必ず Bash で `date +%F` を実行して取得**(スクリプト内では時計を読めない) | なし(必須) |
| `founder` | ファウンダー情報: 人数・拠点・スキル・言語・資金・既存の強み。英語で渡す | ソロ技術系・ブートストラップ・グローバル収益狙い |
| `target` | 収益目標 | $1M ARR within 24 months with a 1-3 person team |
| `language` | 最終計画書の言語 | Japanese(技術用語は英語可) |
| `scale` | `small`(5レンズ/上位2深掘り)/ `standard`(8レンズ/上位3)/ `deep`(10レンズ×3案/上位4) | standard |
| `lenses` | 市場レンズの上書き(文字列配列)。ユーザーが領域を指定した場合のみ設定 | 組み込み10レンズ |

**既存の1案を検証したいだけの場合**: `lenses` にその案の周辺領域を1つ渡し `scale: "small"` にする。生成工程は競合仮説の発見として機能する。

### Step 2: ワークフロー本体を用意して起動する

1. `date +%F` で今日の日付を取得する。
2. **ワークフロースクリプトのパスを解決する**。次の順で探し、最初に見つかったものを使う:
   - 既にディスクにある場合(フォルダ/managed設定でインストール済み): `<現在のリポジトリ>/.claude/skills/venture-discovery/workflow.js`、無ければ `~/.claude/skills/venture-discovery/workflow.js`、無ければ `find / -name workflow.js -path '*venture-discovery*' 2>/dev/null | head -1`。
   - **どこにも無い場合(この単一ファイルで登録した場合はこれ)**: 本ファイル末尾【付録: workflow.js】のコードブロックの中身を**一字一句そのまま** `Write` ツールでスクラッチパッド(例 `<scratchpad>/venture-discovery-workflow.js`)へ書き出す。改変・要約・省略は禁止。書き出したパスを使う。
3. `Workflow` ツールを呼ぶ(argsはJSON値として渡す。文字列化しない):

```
Workflow({
  scriptPath: "<解決したパス>",
  args: { "today": "<YYYY-MM-DD>", "founder": "...", "scale": "standard" }
})
```

バックグラウンドで実行されるので、起動したら概要(工程・想定時間・想定トークン)をユーザーに1段落で報告し、完了通知を待つ。**Bash sleep でのポーリング禁止**。

### Step 3: 結果を検証する(品質ゲート)

完了通知の output ファイルを読み、以下を確認する。**1つでも欠けていたら、成果物を書く前に journal.jsonl を確認して原因を特定し、ユーザーに正直に報告する**:

- [ ] `leaderboard` の件数 ≒ レンズ数×案数(大量欠損は生成工程の失敗)
- [ ] 審査スコアの中央値が35〜65に収まっている(全案70超はスコア校正の失敗 — 結果を信用しない)
- [ ] `winner.factcheck` に CONTRADICTED がある場合、`plan` がその主張に依存していない
- [ ] `critic.pass` の値と、falseの場合は何が不足か
- [ ] `plan` に8セクション+kill criteria表+「明日からの着手順」がある

### Step 4: 成果物を書く

以下の4ファイルを作成する(既存リポジトリ構成に合わせてパス調整可):

| ファイル | 内容(resultのキー → 使い方) |
|---------|------|
| `README.md` | 勝者の概要表(`winner.concept`)、選定方法の要約、**正直な評価**(`adjusted_conviction` をそのまま書く。50未満なら「堅実なニッチ事業でありベンチャースケールの賭けではない」等、数字が示す現実を隠さない)、ドキュメント構成、着手順 |
| `docs/IMPLEMENTATION_PLAN.md` | `plan` をそのまま。冒頭に critic の最終判定(`pass` と残指摘)を注記として追加 |
| `docs/SELECTION.md` | パイプライン設計、`leaderboard` 全件の表(スコア順・一行概要付き)、`killed` とその理由、ファイナリスト比較表(審査スコア/DD後conviction/factcheck結果)、**判断根拠**(スコア差でなくリスクの質で: 需要は日付で確定しているか、間隙は構造的か、最大リスクは既知か、敗北時の残存価値)、選定の限界(顧客インタビュー0件である旨を必ず明記) |
| `docs/RUNNERS_UP.md` | `runnersUp` 各案: 概要、検証で強まった点/崩れた点、生き残った差別化、**乗り換え条件** |

書き終えたら、gitリポジトリ内なら commit する(push はユーザーの指示または既存の指示体系に従う)。

### Step 5: 報告する

最終メッセージは結論先行で: 勝者と一行概要 → 選定理由(リスクの質)→ 正直な評価(conviction値と意味)→ 成果物の場所 → 次のアクション候補。**convictionが低くても取り繕わない**。「確実に儲かる案が見つからなかった」ことは、このパイプラインの正常な出力であり得る。

## コストと時間の目安

| scale | エージェント数 | サブエージェントトークン | 時間 |
|-------|--------------|------------------------|------|
| small | 〜45 | 〜60万 | 30–60分 |
| standard | 〜90 | 〜150万 | 60–120分 |
| deep | 〜180 | 〜300万 | 2–4時間 |

起動前にユーザーがこの規模を了解していることを確認する(スキルを明示的に呼んだ場合は了解済みとみなしてよい)。

## トラブルシューティング

- **resultが空/一部欠損**: `<transcriptDir>/journal.jsonl` を読む(各エージェントの実際の返り値が記録されている)。推測せず必ず読む。
- **途中で停止した**: `Workflow({scriptPath, resumeFromRunId: "<runId>", args: <同じargs>})` で再開。同一の (prompt, opts) は瞬時にキャッシュ再生される。
- **Web検索が全滅**(全evidenceが knowledge-based): 想定内の劣化モード。成果物に「Web検証なし・モデル知識のみ」と明記し、factcheckのUNVERIFIED増加によるconviction低下をそのまま報告する。
- **Workflowツールが無い環境**: 実行不可。会話専用モードの可能性。ユーザーに伝えて中止する。

## 設計意図(弱いモデルでも品質を落とさないための原則・要約)

1. 判断はスクリプト側の決定的ロジック+算術ルーブリックへ移す(モデルに最終順位を決めさせない)。
2. 審査は attack-first(スキーマで攻撃を先に強制)+スコア校正表で忖度を防ぐ。
3. 事実性は独立したファクトチェック工程で検証し、捏造を自動減点に変換する。
4. 広い調査は狭い3リサーチャー+統合者に分割する。
5. 最終計画は厳格テンプレート+批評→改訂ループで締める。
これらの工程・ルーブリック・スキーマは品質の要。改変時は劣化しないことを必ず確認する。

---

## 【付録: workflow.js】

**下記コードブロックの中身を一字一句そのまま** `workflow.js` として書き出してから `Workflow({ scriptPath })` に渡すこと(Step 2 参照)。改変・要約・省略は禁止。

```javascript
export const meta = {
  name: 'venture-discovery',
  description: 'Generate, adversarially judge, fact-check, and deep-dive monetizable product concepts, then synthesize a critic-reviewed implementation plan',
  phases: [
    { title: 'Ideate', detail: 'independent market-lens ideation with web research' },
    { title: 'Dedup', detail: 'merge overlapping concepts (conservative)' },
    { title: 'Judge', detail: '3 adversarial judges per concept, attack-first' },
    { title: 'DeepDive', detail: 'finalists: 3 researchers + consolidator with arithmetic conviction rubric' },
    { title: 'FactCheck', detail: 'verify load-bearing claims, penalize contradictions' },
    { title: 'Synthesize', detail: 'implementation plan from a strict template' },
    { title: 'Critique', detail: 'rubric critic + revision loop' },
  ],
}

// ============================================================
// Args & configuration
// ============================================================
const A = args || {}
if (!A.today) throw new Error('args.today (YYYY-MM-DD) is required — workflow scripts cannot read the clock. The orchestrator must pass it (e.g. from `date +%F`).')
const TODAY = A.today
const FOUNDER = A.founder || 'Solo technical founder, bootstrapped (no outside funding), strong engineer with frontier-AI leverage, targeting GLOBAL revenue.'
const TARGET = A.target || '$1M ARR within 24 months with a 1-3 person team'
const LANG = A.language || 'Japanese (技術用語は英語のままで可)'
const SCALE = A.scale || 'standard'

const CFG = {
  small: { lenses: 5, perLens: 2, finalists: 2, maxClaims: 4, reviseRounds: 1 },
  standard: { lenses: 8, perLens: 2, finalists: 3, maxClaims: 6, reviseRounds: 1 },
  deep: { lenses: 10, perLens: 3, finalists: 4, maxClaims: 8, reviseRounds: 2 },
}[SCALE]
if (!CFG) throw new Error('args.scale must be one of: small | standard | deep')

const DEFAULT_LENSES = [
  { key: 'vertical-ai', prompt: 'Lens: vertical AI for paper-heavy, regulated, or overlooked industries (insurance claims, construction, logistics, immigration/visa, elder care, specialty billing, maritime, waste management). High willingness-to-pay, low tech saturation.' },
  { key: 'smb-backoffice', prompt: 'Lens: SMB back-office automation (bookkeeping, invoicing, collections, procurement, compliance filings, HR admin) where AI now replaces work SMBs currently pay humans or expensive suites for.' },
  { key: 'cross-border', prompt: "Lens: cross-border and localization arbitrage — products that bridge the founder's home market and global markets: export compliance, localization, cross-border e-commerce ops, global hiring edges, inbound tourism B2B." },
  { key: 'devtools', prompt: 'Lens: developer tools and API-first products in the AI era (code quality, migration tooling, testing, security, data pipelines, LLM cost/routing). Usage-based pricing, PLG distribution.' },
  { key: 'prosumer', prompt: 'Lens: prosumer tools for knowledge workers and creators (research, writing, video, sales, recruiting, consulting deliverables) priced $20-200/mo where the tool directly earns the user money or billable time.' },
  { key: 'consumer-sub', prompt: 'Lens: consumer subscription with strong retention loops (health, education, language, parenting, personal finance) where AI enables a 10x better product than the current app-store incumbents.' },
  { key: 'compliance-data', prompt: 'Lens: regulation-driven data/API products and compliance tooling where a law or platform policy WITH A DATE forces purchase (e-invoicing mandates, AI-act obligations, tax regimes, accessibility deadlines). Regulation is the salesperson.' },
  { key: 'marketplace', prompt: 'Lens: marketplaces and network-effect businesses newly enabled by AI (matching, trust/verification, curation, escrow of expertise) where AI collapses the transaction cost that previously blocked the marketplace.' },
  { key: 'agent-infra', prompt: 'Lens: infrastructure/tooling for companies deploying AI agents in production (evals, observability, governance, cost control, agent commerce). B2B, usage-based or seat pricing.' },
  { key: 'services-productization', prompt: 'Lens: productized services where AI collapses COGS by 5-20x (audits, filings, translations, diligence, certifications) — sell the outcome at a fixed price, keep the AI margin, use humans only for the licensed/liability layer.' },
]
const LENSES = (Array.isArray(A.lenses) && A.lenses.length > 0
  ? A.lenses.map((l, i) => typeof l === 'string' ? { key: 'custom-' + i, prompt: 'Lens: ' + l } : l)
  : DEFAULT_LENSES
).slice(0, CFG.lenses)

// ============================================================
// Shared prompt blocks (weaker-model hardened: explicit procedures,
// calibration anchors, hard bans, anti-hallucination rules)
// ============================================================
const WEB = `Today is ${TODAY}.
WEB RESEARCH PROCEDURE (follow exactly):
1. Call ToolSearch with query "select:WebSearch" (add "select:WebFetch" only if you must open a specific page).
2. Use WebSearch with SPECIFIC queries: competitor name + "pricing", "site:reddit.com" + the pain, funding announcements, regulation names + dates.
3. Every web-derived fact must carry its URL. Every fact from memory must be prefixed "knowledge-based:".
4. If ToolSearch or WebSearch fails twice in a row, STOP retrying. Use internal knowledge only and prefix ALL evidence with "knowledge-based:".
ANTI-FABRICATION (absolute): never invent a URL, price, funding round, user count, or company name. An honest "unverified" is worth more than a fabricated citation. Fabrications are caught by a downstream fact-check stage and destroy the analysis.`

const FOUNDER_BLOCK = `FOUNDER CONTEXT (all concepts must fit this):
${FOUNDER}
TARGET: ${TARGET}.`

const CONCEPT_PROPS = {
  name: { type: 'string', description: 'Short product name (English)' },
  one_liner: { type: 'string', description: 'One sentence: what it is and for whom' },
  problem: { type: 'string', description: 'The painful, frequent, monetizable problem. Must name WHO suffers, WHAT it costs them today (money/time), and cite evidence.' },
  target_customer: { type: 'string', description: 'Specific buyer persona + segment size estimate with basis' },
  monetization: { type: 'string', description: 'Revenue model and why this model fits the purchase behavior' },
  pricing_hypothesis: { type: 'string', description: 'Concrete price points benchmarked against what the customer pays TODAY for alternatives' },
  why_now: { type: 'string', description: 'What changed recently (tech, regulation, behavior) that makes this newly possible/urgent. Dates beat vibes.' },
  moat: { type: 'string', description: 'Defensibility over 3 years, especially vs incumbents adding AI and vs model providers bundling the capability. Be honest; "we move fast" is not a moat.' },
  mvp_scope: { type: 'string', description: 'What the stated team ships in 3 months to get the first paying customer' },
  biggest_risk: { type: 'string', description: 'The single most likely reason this fails' },
  evidence: { type: 'string', description: 'Market signals with URL citations or "knowledge-based:" prefix. No naked numbers.' },
  competition: { type: 'string', description: 'NAMED competitors and why they have not already won' },
}

// ============================================================
// Phase 1: Ideate
// ============================================================
phase('Ideate')
log(`Scale=${SCALE}: ${LENSES.length} lenses x ${CFG.perLens} concepts`)

const CONCEPTS_SCHEMA = {
  type: 'object', required: ['concepts'],
  properties: {
    concepts: {
      type: 'array', minItems: CFG.perLens, maxItems: CFG.perLens,
      items: { type: 'object', required: Object.keys(CONCEPT_PROPS), properties: CONCEPT_PROPS },
    },
  },
}

const IDEATION_PROMPT = l => `You are a repeat founder + VC analyst hybrid generating startup concepts.

${FOUNDER_BLOCK}

${l.prompt}

${WEB}

PROCEDURE (do these steps IN ORDER, do not skip):
1. RESEARCH: run 3-6 web searches inside your lens — pain complaints, incumbent pricing pages, recent regulation/platform changes, funding news. Collect concrete numbers.
2. LONGLIST: privately list 5+ candidate problems. For each note: who pays today, how much, for what worse alternative.
3. FILTER with the HARD BANS below. Discard violators without exception.
4. SELECT the ${CFG.perLens} strongest by: (evidence of CURRENT spend) x (wedge shippable in 3 months) x (survives platform vendors bundling the capability).
5. FILL every field of the concept card. Concreteness rules: every number needs a URL or "knowledge-based:" prefix; name real competitors; state prices in actual currency.

HARD BANS (auto-discard):
- Generic "AI chatbot/copilot for X" without a structural reason X pays (regulation, liability, integration depth)
- Anything requiring a license/citizenship/physical presence the founder cannot obtain
- Two-sided marketplaces needing simultaneous supply+demand bootstrap, unless AI collapses one side to near-zero
- Ad-supported consumer apps; virality-dependent social apps
- Thin wrappers on a single model capability (translation, summarization, image gen) sold as the whole product
- Ideas whose evidence section you cannot fill with at least 2 real signals

FIELD QUALITY BAR (calibrate against these):
- GOOD problem: "US freight brokers manually re-key 40+ load confirmations/day from PDF into their TMS; a 20-person brokerage pays 2 FTEs (~$90k/yr) just for re-keying (knowledge-based: r/FreightBrokers threads, 2025)." — names who, quantifies cost, cites.
- BAD problem: "Businesses struggle to manage documents efficiently." — no who, no cost, no evidence. (Do NOT reuse the freight domain just because it appears here.)
- GOOD moat: "Filing history + permit archive accumulate; switching means re-collecting 2 years of data" (structural). BAD moat: "First-mover advantage and superior UX" (not a moat).

Prefer boring-but-profitable over hype. Willingness-to-pay evidence beats novelty. Return exactly ${CFG.perLens} concepts via structured output.`

const ideation = await parallel(LENSES.map(l => () =>
  agent(IDEATION_PROMPT(l), { label: `ideate:${l.key}`, phase: 'Ideate', schema: CONCEPTS_SCHEMA, effort: 'high' })
    .then(r => r ? { lens: l.key, concepts: r.concepts } : null)
))
const rawConcepts = ideation.filter(Boolean).flatMap(x => x.concepts.map(c => ({ ...c, lens: x.lens })))
if (rawConcepts.length === 0) throw new Error('Ideation produced zero concepts — inspect journal.jsonl')
log(`Ideation produced ${rawConcepts.length} raw concepts (${LENSES.length - ideation.filter(Boolean).length} lens agents failed)`)

// ============================================================
// Phase 2: Dedup (conservative)
// ============================================================
phase('Dedup')
const DEDUP_SCHEMA = {
  type: 'object', required: ['groups'],
  properties: {
    groups: {
      type: 'array',
      items: {
        type: 'object', required: ['keep_index', 'duplicate_indices', 'note'],
        properties: {
          keep_index: { type: 'integer' },
          duplicate_indices: { type: 'array', items: { type: 'integer' } },
          note: { type: 'string' },
        },
      },
    },
  },
}
const digest = rawConcepts.map((c, i) => `[${i}] ${c.name} (lens=${c.lens}): ${c.one_liner} | problem: ${String(c.problem).slice(0, 200)}`).join('\n')
const dedup = await agent(
`Here are ${rawConcepts.length} startup concepts, indexed:

${digest}

Identify groups that target essentially the SAME problem AND the SAME customer (>70% overlap in both). For each overlap group pick the strongest formulation as keep_index and list the others as duplicate_indices.
RULES: When unsure, do NOT merge — false merges destroy diversity and are worse than duplicates. Same industry alone is NOT overlap. If nothing overlaps, return an empty groups array.`,
  { label: 'dedup', phase: 'Dedup', schema: DEDUP_SCHEMA, effort: 'medium' })

const dropped = new Set()
if (dedup && dedup.groups) for (const g of dedup.groups) for (const d of g.duplicate_indices || []) if (d !== g.keep_index) dropped.add(d)
const concepts = rawConcepts.filter((c, i) => !dropped.has(i))
log(`After dedup: ${concepts.length} concepts (${dropped.size} merged away)`)

// ============================================================
// Phase 3: Judge (attack-first, calibrated, strict fatal-flaw definition)
// ============================================================
phase('Judge')
const VERDICT_SCHEMA = {
  type: 'object', required: ['attacks', 'score', 'fatal_flaw', 'rationale'],
  properties: {
    attacks: { type: 'array', minItems: 3, maxItems: 5, items: { type: 'string' }, description: 'The strongest CONCRETE ways this business fails, from your lens only. Write these BEFORE deciding the score.' },
    score: { type: 'number', description: '0-100 per the calibration table in the prompt' },
    fatal_flaw: { type: ['string', 'null'], description: 'ONLY per the strict definition in the prompt, else null' },
    rationale: { type: 'string', description: '3-6 sentences. Must state your single strongest attack and whether the concept survives it.' },
  },
}

const CALIBRATION = `SCORE CALIBRATION (anchor to these, do not drift):
- 80-100: you would invest your own savings. Verified willingness-to-pay + structural moat + founder fit. Expect <5% of concepts here.
- 60-79: strong, with one real unresolved risk.
- 40-59: plausible; unproven willingness-to-pay, crowded market, or partner-dependent economics.
- 20-39: broken on one load-bearing assumption (price anchor false, channel closed, incumbent already shipped it).
- 0-19: premise contradicts verifiable reality.
Expected distribution: median around 50; if you are about to score above 70, re-read your own attacks first and justify why none of them lands.

FATAL FLAW — strict definition. Set fatal_flaw ONLY if one of these holds:
(a) operating it is illegal/impossible for THIS founder (license, citizenship, physical presence);
(b) the exact core product already ships free or bundled from the platform the customer already uses;
(c) the named buyer structurally cannot pay (no budget authority, no money);
(d) the claimed acquisition channel is closed AND no substitute channel exists.
NOT fatal (lower the score instead): strong competition, hard GTM, model-provider risk, execution difficulty, thin margins.`

const JUDGE_LENSES = [
  { key: 'monetization', persona: 'You are a skeptical CFO / growth-stage investor. Judge ONLY: willingness to pay, pricing power, CAC vs LTV realism, expansion revenue, and the realistic path to the stated revenue target. Punish nice-to-have tools and one-time purchases dressed as subscriptions. You gain nothing by being nice: a false positive costs the founder a year of their life.' },
  { key: 'feasibility', persona: 'You are a staff engineer who has shipped multiple AI products. Judge ONLY: can the stated team ship the wedge in ~3 months; inference/API cost vs price; platform risk (model providers or OS vendors absorbing it); operational load (support, compliance, data acquisition); whether the moat claim survives incumbents adding AI.' },
  { key: 'market', persona: 'You are a go-to-market operator. Judge ONLY: distribution path for an unknown founder (SEO, communities, PLG, outbound), competition intensity and incumbent funding, timing (too early/late), regulatory exposure, and whether THIS founder can credibly sell to this segment.' },
]
const WEIGHTS = { monetization: 0.4, feasibility: 0.3, market: 0.3 }

const conceptCard = c => `NAME: ${c.name}
ONE-LINER: ${c.one_liner}
PROBLEM: ${c.problem}
TARGET: ${c.target_customer}
MONETIZATION: ${c.monetization}
PRICING: ${c.pricing_hypothesis}
WHY NOW: ${c.why_now}
MOAT: ${c.moat}
MVP (3mo): ${c.mvp_scope}
BIGGEST RISK (self-declared): ${c.biggest_risk}
EVIDENCE: ${c.evidence}
COMPETITION: ${c.competition}`

const judged = await parallel(concepts.map(c => () =>
  parallel(JUDGE_LENSES.map(j => () =>
    agent(
`${j.persona}

${FOUNDER_BLOCK}

Concept under review (today is ${TODAY}):

${conceptCard(c)}

PROCEDURE: (1) write your 3-5 strongest attacks FIRST; (2) steelman the concept against each attack; (3) only then decide the score.

${CALIBRATION}

Return your verdict via structured output.`,
      { label: `judge:${j.key}:${String(c.name).slice(0, 24)}`, phase: 'Judge', schema: VERDICT_SCHEMA, effort: 'high' }
    ).then(v => v ? { lens: j.key, v } : null)
  )).then(verdicts => ({ concept: c, verdicts: verdicts.filter(Boolean) }))
))

const scored = judged.filter(Boolean).map(({ concept, verdicts }) => {
  let total = 0, wsum = 0
  const flaws = [], detail = {}
  for (const { lens, v } of verdicts) {
    total += v.score * WEIGHTS[lens]; wsum += WEIGHTS[lens]
    if (v.fatal_flaw) flaws.push(`[${lens}] ${v.fatal_flaw}`)
    detail[lens] = v
  }
  return { concept, weighted: wsum > 0 ? total / wsum : 0, flaws, detail }
})
const killed = scored.filter(s => s.flaws.length >= 2)
const survivors = scored.filter(s => s.flaws.length < 2).sort((a, b) => b.weighted - a.weighted)
log(`Judged ${scored.length}: ${killed.length} killed (>=2 fatal flaws). Top: ${survivors.slice(0, 5).map(s => `${s.concept.name}=${s.weighted.toFixed(1)}`).join(', ')}`)
const finalistsIn = survivors.slice(0, CFG.finalists)
if (finalistsIn.length === 0) throw new Error('No survivors after judging — inspect journal.jsonl')

// ============================================================
// Phase 4: DeepDive — 3 narrow researchers + consolidator per finalist.
// Narrow scopes suit weaker models; the consolidator's conviction is
// computed with an explicit arithmetic rubric, not vibes.
// ============================================================
phase('DeepDive')

const COMP_SCHEMA = {
  type: 'object', required: ['competitors', 'surviving_gap', 'red_flags'],
  properties: {
    competitors: { type: 'array', items: { type: 'object', required: ['name', 'pricing', 'traction', 'why_not_winning'], properties: { name: { type: 'string' }, pricing: { type: 'string' }, traction: { type: 'string' }, why_not_winning: { type: 'string' } } } },
    surviving_gap: { type: 'string', description: 'The specific gap that survives — or "none survives" if research killed it' },
    red_flags: { type: 'string' },
  },
}
const ECON_SCHEMA = {
  type: 'object', required: ['what_customers_pay_today', 'recommended_packaging', 'unit_economics', 'revenue_model_12mo', 'red_flags'],
  properties: {
    what_customers_pay_today: { type: 'string', description: 'Verified benchmarks with URLs or knowledge-based: prefix' },
    recommended_packaging: { type: 'string', description: 'Tiers and price points that survive the benchmarks' },
    unit_economics: { type: 'string', description: 'Revenue per unit minus ALL COGS incl. human/partner/licensed labor, not just compute. State gross margin honestly.' },
    revenue_model_12mo: { type: 'string', description: 'Month-by-month ramp with every assumption stated' },
    red_flags: { type: 'string' },
  },
}
const GTM_SCHEMA = {
  type: 'object', required: ['gtm_wedge', 'first_100_plan', 'technical_architecture', 'hardest_risks'],
  properties: {
    gtm_wedge: { type: 'string' },
    first_100_plan: { type: 'string', description: 'Concrete channels, communities by name, outbound angle, realistic pacing for THIS founder' },
    technical_architecture: { type: 'string', description: 'Stack sketch, model/API choices, cost per unit of value, hardest technical risk' },
    hardest_risks: { type: 'string' },
  },
}
const DEEPDIVE_SCHEMA = {
  type: 'object',
  required: ['competitive_landscape', 'pricing_benchmarks', 'gtm_wedge', 'technical_notes', 'revenue_model_12mo', 'red_flags', 'verdict_summary', 'conviction_score', 'key_claims'],
  properties: {
    competitive_landscape: { type: 'string' },
    pricing_benchmarks: { type: 'string' },
    gtm_wedge: { type: 'string' },
    technical_notes: { type: 'string' },
    revenue_model_12mo: { type: 'string' },
    red_flags: { type: 'string' },
    verdict_summary: { type: 'string', description: '5-8 sentences, MUST show the conviction arithmetic line by line' },
    conviction_score: { type: 'number' },
    key_claims: {
      type: 'array', minItems: 3, maxItems: CFG.maxClaims,
      items: { type: 'object', required: ['claim', 'how_to_verify'], properties: { claim: { type: 'string', description: 'A falsifiable factual statement the conviction rests on (competitor existence/pricing, regulation date, market spend)' }, how_to_verify: { type: 'string', description: 'Suggested search query' } } },
    },
  },
}

const CONVICTION_RUBRIC = `CONVICTION ARITHMETIC (compute exactly; show the arithmetic in verdict_summary):
Start at 50, then apply every line that holds:
+10 verified willingness-to-pay (customers pay for a WORSE alternative today, with citation)
+10 demand has a DATE (regulation/platform deadline) or an equally hard forcing function
+10 founder-specific asymmetry that competitors cannot copy within 12 months
-10 a named competitor already ships the core promise at equal or lower price
-10 the core acquisition channel is closed or CAC structurally exceeds LTV
-10 economics depend on partners/licenses the founder cannot control
-15 a judge concern that your research CONFIRMED rather than refuted (apply once per confirmed concern, max -30)
Clamp to 0-100. Do not adjust by feel afterwards.`

const dived = await parallel(finalistsIn.map(s => () => {
  const card = conceptCard(s.concept)
  const concerns = Object.entries(s.detail).map(([k, v]) => `- [${k}] score=${v.score}: ${v.rationale}${v.fatal_flaw ? ` FATAL: ${v.fatal_flaw}` : ''}`).join('\n')
  const base = `${FOUNDER_BLOCK}\n\n${WEB}\n\nCONCEPT:\n${card}\n\nJUDGES' CONCERNS (address with evidence):\n${concerns}`
  return parallel([
    () => agent(`You are a competitive-intelligence analyst doing pre-seed diligence.\n\n${base}\n\nTASK: enumerate ALL credible competitors and substitutes: incumbents, funded startups, free/DIY tools, in-house alternatives, and the platform itself absorbing the feature.\nPROCEDURE: (1) run at least 5 distinct web searches ("<concept keywords>", "alternative to <nearest incumbent>", pricing pages, Product Hunt/G2, relevant subreddits); (2) record each competitor's name/pricing/traction and why it has not already won; (3) state the surviving gap in one paragraph — or say "none survives"; (4) list every red flag found.\nOnly name competitors found via search or that you are >90% sure exist from memory (prefix "knowledge-based:").`,
      { label: `dd-comp:${String(s.concept.name).slice(0, 20)}`, phase: 'DeepDive', schema: COMP_SCHEMA, effort: 'high' }),
    () => agent(`You are a pricing/unit-economics analyst doing pre-seed diligence.\n\n${base}\n\nTASK: establish what the target customer actually pays TODAY (tools, labor, agencies) with sources; recommend packaging that survives those benchmarks; compute honest unit economics including ALL human/partner/licensed-labor COGS, not just compute; produce a month-by-month 12-month revenue ramp with every assumption stated. If the concept's pricing collapses against benchmarks, say so plainly.`,
      { label: `dd-econ:${String(s.concept.name).slice(0, 20)}`, phase: 'DeepDive', schema: ECON_SCHEMA, effort: 'high' }),
    () => agent(`You are a go-to-market operator + staff engineer doing pre-seed diligence.\n\n${base}\n\nTASK: (1) the concrete first-100-customers plan for THIS founder — channels and communities BY NAME, outbound angle, realistic pacing; flag any channel that is closed to this founder profile and why. (2) architecture sketch: stack, model/API choices, cost per unit of value delivered, hardest technical risk. Be adversarial about founder-channel fit.`,
      { label: `dd-gtm:${String(s.concept.name).slice(0, 20)}`, phase: 'DeepDive', schema: GTM_SCHEMA, effort: 'high' }),
  ]).then(([comp, econ, gtm]) => {
    const parts = []
    if (comp) parts.push('COMPETITIVE RESEARCH:\n' + JSON.stringify(comp, null, 1))
    if (econ) parts.push('ECONOMICS RESEARCH:\n' + JSON.stringify(econ, null, 1))
    if (gtm) parts.push('GTM/TECH RESEARCH:\n' + JSON.stringify(gtm, null, 1))
    if (parts.length === 0) return null
    return agent(
`You are the diligence lead consolidating three researchers' findings into a final assessment.

${FOUNDER_BLOCK}

CONCEPT:\n${card}

JUDGES' CONCERNS:\n${concerns}

${parts.join('\n\n')}

TASKS:
1. Merge the research into the output fields. Where researchers disagree, prefer the sourced claim over the unsourced one.
2. If research WEAKENED the concept, say so — you are rewarded for accuracy, not optimism.
3. ${CONVICTION_RUBRIC}
4. Extract the ${CFG.maxClaims} most load-bearing FACTUAL claims your conviction rests on, phrased as falsifiable statements (e.g. "Competitor X charges $99/mo", "Regulation Y takes effect 2028-04-01"). These will be independently fact-checked; do not include opinions.`,
      { label: `dd-merge:${String(s.concept.name).slice(0, 20)}`, phase: 'DeepDive', schema: DEEPDIVE_SCHEMA, effort: 'xhigh' }
    ).then(dd => dd ? { ...s, deepdive: dd } : null)
  })
}))
const finalists = dived.filter(Boolean)
if (finalists.length === 0) throw new Error('All deep-dives failed — inspect journal.jsonl')
log(`Deep-dive convictions: ${finalists.map(f => `${f.concept.name}=${f.deepdive.conviction_score}`).join(', ')}`)

// ============================================================
// Phase 5: FactCheck — verify load-bearing claims independently.
// Weaker models' main failure mode is confident fabrication; this stage
// converts that risk into an explicit score penalty.
// ============================================================
phase('FactCheck')
const CLAIM_SCHEMA = {
  type: 'object', required: ['verdict', 'note'],
  properties: {
    verdict: { type: 'string', enum: ['CONFIRMED', 'CONTRADICTED', 'UNVERIFIED'] },
    note: { type: 'string', description: 'One or two sentences: what you found' },
    url: { type: 'string', description: 'Supporting URL if CONFIRMED or CONTRADICTED' },
  },
}
const checked = await parallel(finalists.map(f => () =>
  parallel((f.deepdive.key_claims || []).map(kc => () =>
    agent(
`Fact-check ONE claim. Today is ${TODAY}.

CLAIM: "${kc.claim}"
Suggested search: ${kc.how_to_verify}

PROCEDURE: Call ToolSearch with query "select:WebSearch", then run up to 4 searches.
VERDICT RULES (strict):
- CONFIRMED: only with a URL that directly supports the claim.
- CONTRADICTED: only with a URL that shows the claim is false.
- UNVERIFIED: everything else — including "probably true but I could not find a source" and any web-access failure.
Do NOT reason your way to CONFIRMED without a source.`,
      { label: `check:${String(kc.claim).slice(0, 30)}`, phase: 'FactCheck', schema: CLAIM_SCHEMA, effort: 'low' }
    ).then(v => v ? { claim: kc.claim, ...v } : null)
  )).then(results => {
    const rs = results.filter(Boolean)
    const contradicted = rs.filter(r => r.verdict === 'CONTRADICTED').length
    const unverified = rs.filter(r => r.verdict === 'UNVERIFIED').length
    const adjusted = Math.max(0, f.deepdive.conviction_score - 8 * contradicted - 2 * unverified)
    return { ...f, factcheck: rs, adjusted_conviction: adjusted }
  })
))
const ranked = checked.filter(Boolean)
  .sort((a, b) => (b.adjusted_conviction * 0.6 + b.weighted * 0.4) - (a.adjusted_conviction * 0.6 + a.weighted * 0.4))
for (const f of ranked) {
  const c = f.factcheck.filter(r => r.verdict === 'CONTRADICTED').length
  const u = f.factcheck.filter(r => r.verdict === 'UNVERIFIED').length
  log(`FactCheck ${f.concept.name}: conviction ${f.deepdive.conviction_score} -> ${f.adjusted_conviction} (${c} contradicted, ${u} unverified)`)
}
const winner = ranked[0]
const runnersUp = ranked.slice(1)
log(`Winner: ${winner.concept.name} (adjusted conviction=${winner.adjusted_conviction})`)

// ============================================================
// Phase 6: Synthesize — strict template; forbidden to invent new facts.
// ============================================================
phase('Synthesize')
const PLAN_TEMPLATE = `REQUIRED DOCUMENT STRUCTURE (all 8 sections mandatory, in this order, in ${LANG}):
1. エグゼクティブサマリー — 何を・誰に・なぜ今・いくらで。表形式の要点サマリーを含む。
2. プロダクト仕様 — MVP機能リスト(番号付きテーブル、必須/除外を明記)、主要ユーザージャーニー2本以上。
3. システムアーキテクチャ — 技術スタック選定表(理由付き)、コアデータモデル(スキーマ風)、LLM/APIパイプライン表と1顧客あたり月次原価、スケーリング方針。
4. 収益化設計 — 料金プラン表、単位経済テーブル(粗利率を正直に:人的/パートナーCOGSを含める)、価格の根拠(上限/下限アンカー)。
5. GTM計画 — チャネル優先順位(番号付き)、最初の100顧客の週次アクション表、現実的なペーシング。
6. 12ヶ月ロードマップ — 表形式(月/マイルストーン/累計顧客/月次収益目標)+ Kill criteria表(条件/判定時期/アクション — 日付と数値で測定可能に)。
7. リスク登録簿 — 表形式(リスク/確率/影響/緩和策)、8件以上。fact-checkでCONTRADICTEDになった主張に依存するリスク緩和は書かないこと。
8. 競合対応 — 主要競合ごとの差別化と維持戦略の表、差別化の恒久メカニズムの順位付け。
末尾に「明日からの着手順(engineering)」を5項目。

HARD RULES:
- Use ONLY facts present in the inputs below. If you need a number that is not provided, write it as "仮定: <value>" and state the assumption inline.
- Any input claim marked CONTRADICTED by fact-check must NOT be used; design around it and mention the correction once.
- Target length 12,000-20,000 characters. No filler sentences; every paragraph must be actionable or a decision.
- The plan must be specific enough that engineering can start tomorrow.`

const synthesisInput = w => `WINNING CONCEPT:
${conceptCard(w.concept)}

DEEP-DIVE DILIGENCE:
Competitive landscape: ${w.deepdive.competitive_landscape}
Pricing benchmarks: ${w.deepdive.pricing_benchmarks}
GTM wedge: ${w.deepdive.gtm_wedge}
Technical notes: ${w.deepdive.technical_notes}
12mo revenue model: ${w.deepdive.revenue_model_12mo}
Red flags: ${w.deepdive.red_flags}
Verdict: ${w.deepdive.verdict_summary}

FACT-CHECK RESULTS (respect these):
${w.factcheck.map(r => `- [${r.verdict}] ${r.claim}${r.note ? ' — ' + r.note : ''}`).join('\n')}

IDEAS TO GRAFT from runner-up concepts (steal anything that strengthens the winner):
${runnersUp.map(r => `- ${r.concept.name}: ${r.concept.one_liner} | moat: ${r.concept.moat}`).join('\n') || '(none)'}`

let plan = await agent(
`You are the founding architect writing the master implementation plan. Today is ${TODAY}. Design for DILIGENCE-ADJUSTED reality, not the original pitch: red flags and fact-check corrections are constraints, not footnotes.

${FOUNDER_BLOCK}

${synthesisInput(winner)}

${PLAN_TEMPLATE}

Your entire final response must be the markdown document itself — no preamble, no closing remarks.`,
  { label: 'synthesize:plan', phase: 'Synthesize', effort: 'max' })
if (!plan) throw new Error('Synthesis returned empty — inspect journal.jsonl')

// ============================================================
// Phase 7: Critique + revise loop — catches weaker-model flab,
// internal inconsistency, and template violations.
// ============================================================
phase('Critique')
const CRITIC_SCHEMA = {
  type: 'object', required: ['section_scores', 'pass', 'summary'],
  properties: {
    section_scores: {
      type: 'array',
      items: { type: 'object', required: ['section', 'score', 'required_fixes'], properties: { section: { type: 'string' }, score: { type: 'number', description: '1-5' }, required_fixes: { type: 'string', description: 'Concrete edits, not vibes. "OK" if score is 5.' } } },
    },
    pass: { type: 'boolean' },
    summary: { type: 'string' },
  },
}
const criticPrompt = p => `You are a hard-nosed reviewer checking an implementation plan against its diligence data. Reward accuracy and actionability, punish filler and invention.

DILIGENCE DATA (ground truth):
${synthesisInput(winner)}

PLAN UNDER REVIEW:
${p}

Score each of the 8 required sections 1-5:
- 5 = specific enough to act on tomorrow, internally consistent, numbers traceable to the diligence data
- 3 = generic but usable
- 1 = filler, missing, or contradicts the diligence data
Also verify: (a) every revenue number is consistent with the 12mo model; (b) kill criteria are dated and measurable; (c) nothing relies on a CONTRADICTED claim; (d) MVP scope is shippable by the stated team in 3 months; (e) unit economics include human/partner COGS, not just compute.
pass = every section >= 4 AND checks a-e all hold. required_fixes must be concrete edits.`

let critic = await agent(criticPrompt(plan), { label: 'critic:1', phase: 'Critique', schema: CRITIC_SCHEMA, effort: 'xhigh' })
let rounds = 0
while (critic && !critic.pass && rounds < CFG.reviseRounds) {
  rounds++
  log(`Critic round ${rounds}: pass=false — revising. ${critic.summary}`)
  const fixes = critic.section_scores.filter(s => s.score < 4).map(s => `- [${s.section}] ${s.required_fixes}`).join('\n')
  const revised = await agent(
`You are revising an implementation plan. Apply ALL required fixes below. Keep everything that was not criticized. Do not shorten the document. Use only facts from the diligence data; mark new assumptions "仮定:".

DILIGENCE DATA (ground truth):
${synthesisInput(winner)}

CURRENT PLAN:
${plan}

REQUIRED FIXES:
${fixes}

Your entire final response must be the full revised markdown document.`,
    { label: `revise:${rounds}`, phase: 'Critique', effort: 'max' })
  if (revised) plan = revised
  critic = await agent(criticPrompt(plan), { label: `critic:${rounds + 1}`, phase: 'Critique', schema: CRITIC_SCHEMA, effort: 'xhigh' })
}
log(`Final critic verdict: pass=${critic ? critic.pass : 'unknown'} after ${rounds} revision round(s)`)

// ============================================================
// Return everything the orchestrator needs to write the docs
// ============================================================
return {
  config: { scale: SCALE, today: TODAY, founder: FOUNDER, target: TARGET, lenses: LENSES.map(l => l.key) },
  leaderboard: survivors.map(s => ({ name: s.concept.name, lens: s.concept.lens, one_liner: s.concept.one_liner, weighted: Number(s.weighted.toFixed(1)), flaws: s.flaws })),
  killed: killed.map(s => ({ name: s.concept.name, lens: s.concept.lens, flaws: s.flaws })),
  winner: {
    concept: winner.concept, weighted: winner.weighted, judge_detail: winner.detail,
    deepdive: winner.deepdive, factcheck: winner.factcheck, adjusted_conviction: winner.adjusted_conviction,
  },
  runnersUp: runnersUp.map(r => ({
    concept: r.concept, weighted: r.weighted,
    deepdive: r.deepdive, factcheck: r.factcheck, adjusted_conviction: r.adjusted_conviction,
  })),
  plan,
  critic,
}
```
