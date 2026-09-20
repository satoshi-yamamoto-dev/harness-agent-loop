# harness-agent-loop

短い依頼から、Planner・Generator・Evaluator の役割で仕様策定、実装、評価を進める開発ハーネスです。Claude Code と Codex に対応します。

[ユーザーガイド（GitHub Pages）](https://satoshi-yamamoto-dev.github.io/harness-agent-loop/)

## インストール：Claude Code

既存の導入コマンドを引き続き使用できます。マーケットプレイスとプラグインはいずれもユーザースコープです。

~~~text
claude plugin marketplace add --scope user satoshi-yamamoto-dev/harness-agent-loop
claude plugin install --scope user agent-loop@harness-tools
~~~

対象プロジェクトで開始します。

~~~text
/agent-loop:start タスク管理アプリを作りたい。期限と優先度で整理したい。
~~~

Playwright MCP を使って評価する場合は、対象アプリを起動し、Claude Code から接続できるようにしてください。

既に Claude Code 版をインストールしている場合は、公開後に次のコマンドでマーケットプレイスとプラグインを更新できます。

~~~text
claude plugin marketplace update harness-tools
claude plugin update --scope user agent-loop@harness-tools
~~~

## インストール：Codex

Codex のマーケットプレイスとプラグインを追加します。

~~~text
codex plugin marketplace add satoshi-yamamoto-dev/harness-agent-loop
codex plugin add agent-loop@harness-tools-codex
~~~

インストール後、対象プロジェクトの Codex の入力欄で Skill を指定します。これはシェルコマンドではありません。

~~~text
$agent-loop:start タスク管理アプリを作りたい。期限と優先度で整理したい。
~~~

Codex の評価には、利用環境で使えるブラウザ操作手段と起動済みの対象アプリが必要です。実操作できない条件は未検証として扱います。

## 共通の進行手順

~~~text
Planner → Generator（契約起票）⇄ Evaluator（契約承認）
        → Generator（実装とUI）→ Evaluator（最終判定）
~~~

Planner が製品仕様書とスプリント計画を作り、Generator がテスト可能な完了条件を起票します。Evaluator が契約を承認してから実装します。最終評価で不合格になった場合は、修正と再評価を繰り返します。

対象プロジェクトの docs/010_製品仕様書/spec.md に仕様を、docs/020_スプリント/sprint-N.md に契約と判定詳細を記録します。docs/020_スプリント/README.md は全スプリントの進捗の正本で、メインエージェントだけが更新します。

## ディレクトリの役割

| パス | 用途 |
|---|---|
| cc/agent-loop/ | Claude Code 専用のプラグイン、エージェント、Skill |
| cx/agent-loop/ | Codex 専用のプラグインと Skill |
| .claude-plugin/marketplace.json | Claude Code 用マーケットプレイスの入口 |
| .agents/plugins/marketplace.json | Codex 用マーケットプレイスの入口 |
| docs/ | 両製品共通の公開ガイド |

オーケストレーション手順は各プラグインの skills/start/SKILL.md にあります。Claude Code 版と Codex 版の実行資源は独立しています。

## セキュリティ

プラグインは対象プロジェクトのファイル編集、開発コマンドの実行、ブラウザ操作を行います。内容を確認した信頼できるプロジェクトで使用してください。仕様書やスプリント資料に認証情報を記載せず、Evaluator には信頼できる対象アプリの URL だけを渡してください。

## GitHub Pages

公開ガイドはリポジトリの docs/ に置いています。GitHub Pages の Source は Deploy from a branch、Branch は main、フォルダーは /docs を使用します。

## 設計の出典とクレジット

設計思想は [Anthropic の Harness Design for Long-Running Apps](https://www.anthropic.com/engineering/harness-design-long-running-apps) に基づきます。本リポジトリは [Shin-sibainu/agent-quartet-harness](https://github.com/Shin-sibainu/agent-quartet-harness)（MIT License）を改変した派生物です。

[MIT License](LICENSE) — 元作者 Shin-sibainu および改変者 Satoshi Yamamoto の著作権表示を含みます。
