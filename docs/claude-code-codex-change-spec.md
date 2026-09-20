# Claude Code・Codex 両対応の変更仕様書兼タスク一覧

## 目的と範囲

既存の Claude Code 用 `agent-loop` を維持しながら、同じ開発フローを使える Codex 用プラグインを追加する。プラグインの実行資源は `cc/` と `cx/` に明確に分け、共通の公開ドキュメントはルートの `docs/` に置く。利用者向けガイドは共通化し、製品ごとに異なる操作だけを切り替えて表示する。

この文書は変更仕様と作業一覧を兼ねる。チェックは今回実施した作業と短時間の確認を示し、実インストールやスプリント全体の検証は含めない。

## ディレクトリ構造

```text
harness-agent-loop/
├── .claude-plugin/
│   └── marketplace.json             # Claude Code のリポジトリ入口
├── .agents/plugins/
│   └── marketplace.json             # Codex のリポジトリ入口
├── cc/                               # Claude Code 専用
│   └── agent-loop/
│       ├── .claude-plugin/plugin.json
│       ├── .claude/agents/
│       │   ├── planner.md
│       │   ├── generator.md
│       │   └── evaluator.md
│       └── skills/start/
│           ├── SKILL.md
│           └── references/sprint-progress-readme.md
├── cx/                               # Codex 専用
│   └── agent-loop/
│       ├── plugin.json
│       └── skills/start/
│           ├── SKILL.md
│           └── references/
│               ├── planner.md
│               ├── generator.md
│               ├── evaluator.md
│               └── sprint-progress-readme.md
├── docs/                             # 共通の公開ガイドと本仕様書
│   ├── index.html
│   ├── setup.html
│   ├── usage.html
│   ├── assets/
│   └── claude-code-codex-change-spec.md
├── README.md                         # 共通のリポジトリ案内
└── LICENSE
```

ルートの二つのマーケットプレイスファイルは発見用の薄い入口とする。実行時の指示、役割定義、依存設定はそれぞれ `cc/agent-loop/` と `cx/agent-loop/` の中に閉じる。`docs/` は両製品共通の公開ドキュメントとしてルートに残し、現在の GitHub Pages の `/docs` 公開を維持する。

インストール時にプラグイン本体の外側がコピーされるとは限らないため、実行時に必要なテンプレートは各プラグインへ同梱し、パッケージ外のファイルを `../` で直接参照しない。

## Claude Code 互換性

Claude Code の既存コマンドを変更しない。

```text
claude plugin marketplace add --scope user satoshi-yamamoto-dev/harness-agent-loop
claude plugin install --scope user agent-loop@harness-tools
```

そのため、ルートの `.claude-plugin/marketplace.json` と、マーケットプレイス名 `harness-tools`、プラグイン名 `agent-loop` を維持する。プラグインの参照先のみ、現在の `./` から `./cc/agent-loop` に変更する。既存利用者向けには、移動後の更新方法と必要なバージョン変更を確認して記載する。Skill の呼び出し名 `/agent-loop:start` も維持する。

## Codex 版の動作

- `cx/agent-loop/plugin.json` を Codex 用パッケージのマニフェストとし、`skills/start/SKILL.md` を入口とする。
- Planner、Generator、Evaluator の役割と、仕様策定 → 契約起票 → 契約レビュー → 実装 → 最終判定の順序を維持する。
- Codex では Claude Code 固有のエージェント登録名やモデル指定を使用しない。Skill が同梱の役割定義を参照し、利用可能なサブエージェント機能へ作業を渡す。利用できない環境では段階ごとに役割を切り替える。
- `docs/020_スプリント/README.md` の作成・更新はメインエージェントだけが行う。Evaluator の判定詳細は `sprint-N.md` に記録する。
- 最終判定では実際にアプリを操作して契約条件を検証する。利用可能なブラウザ操作手段を明記し、実操作できない条件を合格扱いにしない。
- Codex 用マーケットプレイスはルートの `.agents/plugins/marketplace.json` から `./cx/agent-loop` を指す。マーケットプレイス名は Claude Code 版と区別できる名前にする。具体的なインストールコマンドは現行の公式手順と CLI のヘルプを確認してガイドに記載し、実インストールでの確認は運用時に行う。

## ドキュメント仕様

- `README.md` は概要とディレクトリの対応表を共通で記載し、インストール手順を「Claude Code」「Codex」の見出しに分ける。Markdown にはタブ UI を使わない。
- `docs/index.html` は共通の開発フローを説明し、現在の「Claude Code Plugin」という製品限定表記を両対応の表記に直す。「3つのサブエージェント」のような環境依存の断定は避け、3つの役割として説明する。
- `docs/setup.html` は「Claude Code」「Codex」のタブを設け、前提条件、インストール、開始例、確認方法を選択中の製品について一続きで読めるようにする。既存の Claude Code コマンドはそのまま掲載する。
- `docs/usage.html` はスプリントの流れ、成果物、評価基準を共通で記載する。Skill の呼び出し方、ブラウザ設定、アンインストール、トラブル対応、エージェントの扱いだけを製品別にする。
- タブはキーボード操作、選択状態の読み上げ、JavaScript が使えない場合の閲覧を考慮する。実装時には短い手動確認を行う。
- 図、CSS、リンク、GitHub Pages の公開 URL は、共通のガイドから引き続き利用できるようにする。

## 完了条件

1. Claude Code のマーケットプレイス名、プラグイン名、既存インストールコマンド、`/agent-loop:start` を変更しない構成になっている。
2. Codex 用プラグインとマーケットプレイスを独立したディレクトリに配置し、`start` Skill を同梱している。
3. 両版の手順に同じ5段階のフローとスプリント成果物が記載されている。
4. 各プラグインに必要な実行時ファイルが同梱され、パッケージ外への参照がない。
5. 共通ガイドで、どちらの製品のコマンド・設定か判別できる。
6. GitHub Pages の `/docs` 公開構造と既存のページ間リンクを維持する。

実際のインストール、長いスプリント実行、ブラウザ評価の通し検証は、この変更の完了条件に含めない。必要に応じて運用しながら確認し、問題が出た箇所を修正する。

## タスク一覧

### 1. 現状確認と移行準備

- [x] Claude Code 版のマニフェスト、Skill、3役割定義、進捗テンプレートの依存パスを洗い出す。
- [x] 既存の Claude Code インストール・起動手順を README とガイドから確認する。
- [x] Codex 版のブラウザ評価に必要な前提と未検証時の扱いを短く記載する。

### 2. ディレクトリとプラグインの分離

- [x] Claude Code の実行資源を `cc/agent-loop/` へ移し、内部参照を更新する。
- [x] ルートの `.claude-plugin/marketplace.json` の参照先を `./cc/agent-loop` に変更し、名前 `harness-tools` と `agent-loop` を維持する。
- [x] Codex 用の `cx/agent-loop/plugin.json` と `skills/start/SKILL.md` を作成する。
- [x] Planner、Generator、Evaluator の指示を Codex 用に移し、Claude 固有の登録名・モデル・ツール名への依存を除く。
- [x] 必要な進捗テンプレートを各プラグインに同梱し、パッケージ外への相対参照をなくす。
- [x] Codex 用の `.agents/plugins/marketplace.json` を作成し、`./cx/agent-loop` を参照させる。

### 3. ドキュメントの更新

- [x] `README.md` に Claude Code と Codex の構造・導入手順を別見出しで記載する。
- [x] `docs/index.html` の概要と製品名表記を両対応に更新する。
- [x] `docs/setup.html` に製品別タブを追加し、各製品の前提・インストール・開始・確認手順を記載する。
- [x] `docs/usage.html` の共通フローを保ち、製品別の呼び出し・評価環境・削除・トラブル対応を整理する。
- [x] タブの切り替えとキーボード操作が JavaScript に依存しない構造であることを確認する。
- [x] 既存のページ間リンクと図の参照先を確認する。

### 4. 短時間の確認

- [x] マニフェストとマーケットプレイスの JSON 構文を確認する。
- [x] マーケットプレイスの参照先、Skill から読むファイル、ページ間リンクの存在を確認する。
- [x] Claude Code と Codex のコマンド・名前・パスがガイド内で混在していないか読み直す。
