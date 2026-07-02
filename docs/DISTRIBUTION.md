# venture-discovery スキルを「どのチャットでも」使えるようにする

Claude Code の **web(claude.ai/code)** はセッションごとにコンテナが破棄され、起動時にリポジトリをクローンし直す。ホームディレクトリ(`~/.claude/`)も毎回消える。したがって「全セッション・全リポジトリで使う」には、**リポジトリ外の永続レイヤー**にスキルを載せる必要がある。3つの公式ルートがあり、立場によって選ぶ。

出典: https://code.claude.com/docs/en/claude-code-on-the-web / https://code.claude.com/docs/en/server-managed-settings / https://code.claude.com/docs/en/skills

---

## ルートA(個人向け・推奨): claude.ai でスキルを有効化する

web ドキュメントの明記:「**Skills you enable on claude.ai are loaded into cloud sessions automatically**(claude.ai で有効化したスキルはクラウドセッションに自動ロードされる)」。個人アカウントで全リポジトリ横断にする最短ルート。

**手順**:
1. `venture-discovery` を1つのフォルダ(`SKILL.md` + `workflow.js` + `DESIGN.md`)にまとめ、zip 化する(このリポジトリの `dist/venture-discovery-skill.zip` に用意済み。再生成は下記コマンド)。
2. claude.ai の設定 → Capabilities / Skills(スキル)で当該スキルをアップロードして有効化する。
3. 以後、新規に起動する Claude Code(web)セッションに自動で載る。リポジトリを問わない。

**注意**:
- 有効化スキルのアップロードUI・可否はプランに依存する。設定画面にスキル項目が無い場合はルートB/Cへ。
- claude.ai の**通常チャット(Codeでない)では Workflow ツールが無い**ため、スキルが載っても本パイプラインは実行できない(会話のみ)。実行は Claude Code セッションで行う。

zip 再生成:
```bash
cd .claude/skills && zip -r ../../dist/venture-discovery-skill.zip venture-discovery \
  -x '*.DS_Store'
```

### A-2: 単一 `SKILL.md` で登録する場合(フォルダ不可のUI向け)

登録UIが**単一 md ファイル**しか受け付けない場合は、`dist/standalone/SKILL.md` を使う。これは通常の SKILL.md に**ワークフロー本体(`workflow.js`)を末尾付録として同梱**した自己完結版で、実行時に付録のコードをディスクへ書き出してから `Workflow` を起動する。フォルダ登録が可能ならフォルダ版(ディスク上の `workflow.js` を直接使う)の方が確実。

単一ファイル再生成:
```bash
# head(frontmatter+手順)は dist/standalone を生成したスクリプトを参照。
# 手動なら: SKILL.md の付録フェンス直後に .claude/skills/venture-discovery/workflow.js を挿入し、末尾に閉じフェンスを付ける。
cat dist/standalone/_head.md .claude/skills/venture-discovery/workflow.js > dist/standalone/SKILL.md && printf '```\n' >> dist/standalone/SKILL.md
```
注意: 付録は「一字一句そのまま書き出す」指示付き。`workflow.js` を更新したら単一ファイルも再生成すること(二重管理になる — フォルダ登録が使えるならそちらを優先)。

---

## ルートB(チーム/組織向け・最も確実): server-managed settings + SessionStart フック

組織の Owner / Primary Owner が [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code) で設定する managed settings は、**全セッションが起動時にサーバーから取得**する(コンテナが破棄されても再取得される)。ここに SessionStart フックを仕込み、毎セッションでスキルを取得・配置する。

**managed settings に入れる JSON(例)**:
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "mkdir -p ~/.claude/skills && git clone --depth 1 https://github.com/w-udagawa/fable5-test /tmp/vd-src 2>/dev/null && cp -r /tmp/vd-src/.claude/skills/venture-discovery ~/.claude/skills/venture-discovery"
          }
        ]
      }
    ]
  }
}
```

- managed settings は組織全ユーザーに一律適用される(現状グループ別指定は不可)。
- クローン元は専用の「スキル配布リポジトリ」を用意するとクリーン(本リポジトリを直接指してもよい)。
- フックは Claude Code 起動前に走るため、`~/.claude/skills/` に置いたスキルが起動時スキャンで登録される。
- ネットワークは既定で Trusted のため GitHub へ到達可能。適用確認は `/status`、権限は `/permissions`。

---

## ルートC(単純・確実だが横断でない): 各リポジトリに置く

スキルを使いたいリポジトリそれぞれの `.claude/skills/venture-discovery/` にコミットする。「グローバル」ではないが、対象リポジトリの新規セッションで確実に登録される。テンプレートリポジトリや `git subtree` で複数リポに配ると管理が楽。

---

## どのルートでも共通の前提

- **新規セッションであること**: スキルは**セッション開始時にスキャン・登録**される。既に開いているセッションに後から入れても、そのセッションでは `Unknown skill` のまま(作業ツリーにファイルがあれば Claude が手動フォールバックはできるが、それは簡易版であり本 `workflow.js` ではない)。
- **エージェント実行モードであること**: 本スキルは Workflow / サブエージェントを起動する。web の Claude Code セッションでは利用可能。Workflow の無い会話専用モードでは実行不可。
- **登録の確認**: 新規セッションで `/venture-discovery` が補完に出る、または「venture-discovery を回して」で `Skill` ツールが解決すれば登録成功。

## 意思決定の早見表

| 立場 | 選ぶルート |
|------|-----------|
| 個人・全リポジトリ横断 | **A**(claude.ai で有効化) |
| 組織で全員に配布 | **B**(managed settings + SessionStart フック) |
| 特定の数リポジトリだけ | **C**(各リポにコミット) |
| A のUIが無い/不可 | B(自分が Owner の個人組織なら可)または C |
