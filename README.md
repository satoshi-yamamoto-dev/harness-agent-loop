# harness-agent-loop

短いプロンプトから、自律的なエージェントループでアプリケーションを設計・実装・評価するための Claude Code Plugin です。

## インストール（Claude Code）

ターミナルで次のコマンドを実行してください。マーケットプレイスとプラグインはいずれもユーザースコープへインストールされます。

```text
claude plugin marketplace add --scope user satoshi-yamamoto-dev/harness-agent-loop
claude plugin install --scope user agent-loop@harness-tools
```

再起動後、対象プロジェクトで `start` Skill を実行します。

```text
/agent-loop:start タスク管理アプリを作りたい。期限と優先度で整理したい。
```

プロジェクト名は `harness-agent-loop`、マーケットプレイス名は `harness-tools`、プラグイン名は `agent-loop` です。`@` の前にバックスラッシュは不要です。

オーケストレーション手順は `skills/start/SKILL.md` に含まれています。`/agent-loop:start` で明示的に実行できるほか、新規開発・機能追加・スプリント再開の依頼に応じて Claude が Skill を選択できます。Evaluator の実行には Playwright MCP と起動済みの対象アプリが必要です。

スプリント進捗は対象プロジェクトの `docs/020_スプリント/README.md` で一元管理します。更新はメインエージェント専任で、Planner / Generator / Evaluator の完了報告を受けるたびに、該当 Step の状態を更新します。Evaluator の詳細な判定結果は各 `sprint-N.md` に分離して記録します。

[ユーザーガイド（GitHub Pages）](https://satoshi-yamamoto-dev.github.io/harness-agent-loop/)

## セキュリティ

このプラグインは、対象プロジェクトのファイル編集、開発コマンドの実行、Playwrightによるブラウザ操作を行います。内容を確認した信頼できるプロジェクトで使用し、Claude Codeの権限確認を無効にしないでください。仕様書やスプリント資料に認証情報を記載せず、Evaluatorには信頼できる対象アプリのURLだけを渡してください。

## GitHub への公開

この README.md があるフォルダーの内容を、`satoshi-yamamoto-dev/harness-agent-loop` リポジトリのルートに配置します（外側のフォルダーを含めないでください）。

1. リポジトリの **Settings → Pages → Build and deployment → Source** を **Deploy from a branch** に設定する。
2. Branchを **main**、フォルダーを **/docs** に設定して保存する。
3. デプロイの成功後、上記のユーザーガイド URL を開く。

ローカルで配布設定を検証する場合:

```sh
claude plugin validate .
claude plugin validate .claude-plugin/marketplace.json
claude --plugin-dir .
```

## 設計の出典

このハーネスの設計思想は、Anthropic のエンジニアリングブログに基づいています。

- [Harness Design for Long-Running Apps](https://www.anthropic.com/engineering/harness-design-long-running-apps)

記事の中核である「**生成と評価の分離**」「**実装前のスプリント契約の合意**」「**主観的品質の定量化（デザイン4基準）**」「**必要になってから複雑さを足す**」という原則を反映しています。

## エージェント構成

```
planner → generator(契約起票) ⇄ evaluator(契約承認) → generator(実装+UI) → evaluator(最終判定)
                                                            ↑                       |
                                                            └───────────────────────┘ (不合格時フィードバック)
```

| エージェント | 役割 | 定義ファイル |
|------------|------|------------|
| **planner** | 短いプロンプトを製品仕様書・スプリント計画に展開 | [.claude/agents/planner.md](.claude/agents/planner.md) |
| **generator** | スプリント契約の起票と、機能＋UIの実装 | [.claude/agents/generator.md](.claude/agents/generator.md) |
| **evaluator** | Playwright による機能テスト・デザイン評価・契約レビュー | [.claude/agents/evaluator.md](.claude/agents/evaluator.md) |

エージェント間の連携手順は [skills/start/SKILL.md](skills/start/SKILL.md) を参照してください。

## ディレクトリ構成

```
.
├── .claude/
│   └── agents/          # サブエージェント定義（planner / generator / evaluator）
├── skills/
│   └── start/
│       ├── SKILL.md     # 開始処理とオーケストレーション手順
│       └── references/  # 進捗READMEのひな形
├── docs/
│   ├── 010_製品仕様書/  # planner が生成する製品仕様書
│   └── 020_スプリント/  # READMEで進捗管理、sprint-N.mdに計画・契約・評価詳細
├── CLAUDE.md            # プロジェクト固有情報（技術スタック・起動方法など）を記載
└── README.md
```

## 使い方

1. `CLAUDE.md` にプロジェクト固有情報（技術スタック、開発サーバーの起動方法など）を記入する
2. `/agent-loop:start` に作りたいものを短いプロンプト（1〜4行）で渡す
3. 以降は [start Skill](skills/start/SKILL.md) の手順に沿ってループが進む

## クレジット

本リポジトリは [Shin-sibainu/agent-quartet-harness](https://github.com/Shin-sibainu/agent-quartet-harness)（MIT License）を改変した派生物です。

## ライセンス

[MIT License](LICENSE) — 元作者 Shin-sibainu および改変者 Satoshi Yamamoto の著作権表示を含みます。
