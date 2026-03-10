# geolonia-template

Geolonia TypeScript プロジェクトのベーステンプレート。
ライブラリ、API、フロントエンドアプリなど、用途を問わず共通の開発基盤を提供します。

## Getting Started

### 方法 1: 新規リポジトリ（GitHub UI）

1. このリポジトリの **"Use this template"** → **"Create a new repository"** をクリック
2. リポジトリ名と説明を入力して作成
3. クローンしてカスタマイズ（下記「カスタマイズ」参照）

### 方法 2: 新規リポジトリ（CLI）

```bash
gh repo create my-project --template geolonia/geolonia-template --public
gh repo clone my-project
cd my-project
```

### 方法 3: 既存リポジトリに適用（予定）

> **Note**: `npx @geolonia/template apply` は未実装です。実装状況は [#10](https://github.com/geolonia/geolonia-template/issues/10) を参照してください。

```bash
npx @geolonia/template apply
```

現時点では、AI エージェントが `template-manifest.yaml` を読んで必要なファイルを適用できます。

### カスタマイズ

| ファイル | 変更箇所 |
|--------|--------|
| `package.json` | `name`, `description` |
| `catalog-info.yaml` | `{{PLACEHOLDER}}` を実際の値に |
| `.github/CODEOWNERS` | チームスラッグ |
| `CLAUDE.md` | プロジェクト固有のメモ |
| `AGENTS.md` | プロジェクト概要・Gotchas |
| `README.md` | このファイル自体 |

用途に応じて追加:
- **ライブラリ**: `publishConfig`, `main`, `types`, `exports` を package.json に追加。`tsconfig.json` に `declaration: true`。CI に npm publish ジョブ追加
- **フロントエンド**: Vite, React, TailwindCSS 等を追加
- **API**: Lambda handler, Express/Hono 等を追加

### セットアップ

```bash
pnpm install
npx lefthook install
pnpm run typecheck && pnpm run lint && pnpm run test && pnpm run build
```

## What's Included

| ツール | 目的 | 設定ファイル | 採用理由 |
|-------|------|------------|---------|
| TypeScript 5+ strict | 型安全 | `tsconfig.json` | |
| Biome | Lint + Format | `biome.json` | [ADR-0001](docs/decisions/0001-use-biome.md) |
| Lefthook | Git hooks | `lefthook.yml` | [ADR-0002](docs/decisions/0002-use-lefthook.md) |
| Vitest | テスト | (package.json) | |
| Claude Code Hooks | AI エージェント品質ゲート | `.claude/settings.json` | |
| Backstage + ISMS | コンプライアンス | `catalog-info.yaml` | |
| GitHub Actions | CI | `.github/workflows/ci.yml` | |
| Dependabot | 依存関係自動更新 | `.github/dependabot.yml` | |

## Directory Structure

```
.
├── .claude/                    # Claude Code 設定
│   ├── settings.json           #   hooks 設定
│   └── skills/                 #   スキル
├── .github/                    # GitHub 設定
│   ├── workflows/ci.yml        #   CI
│   ├── CODEOWNERS              #   レビュー必須設定
│   └── dependabot.yml          #   依存関係自動更新
├── docs/decisions/             # ADR (Architecture Decision Records)
├── scripts/hooks/              # Claude Code hook スクリプト
│   ├── guard.sh                #   安全ガード (PreToolUse)
│   └── post-tool-format.sh     #   自動フォーマット (PostToolUse)
├── src/                        # ソースコード
├── tests/                      # テスト (src/ と同じ構造)
├── AGENTS.md                   # エージェント共通指示
├── CLAUDE.md                   # Claude Code 固有設定
├── biome.json                  # Biome 設定
├── catalog-info.yaml           # Backstage + ISMS メタデータ
├── lefthook.yml                # Git hooks 設定
└── tsconfig.json               # TypeScript 設定
```

## Claude Code Hooks

**初期状態で有効:**
- `guard.sh` (PreToolUse) — 破壊的操作ブロック、main 直接 push ブロック、push 前 typecheck/lint
- `post-tool-format.sh` (PostToolUse) — ファイル書き込み後に自動フォーマット

**問題が起きたら有効化:**

| 問題 | 対策 | 方法 |
|------|------|------|
| エージェントが biome.json 等のルールを緩めた | 設定ファイル保護 | `guard.sh` に Hook 5 を追加（[コード例](https://github.com/geolonia/multi-agent-shogun/blob/main/scripts/hooks/guard.sh)） |
| push 前レビューを強制したい | code-review-expert 必須化 | `guard.sh` に Hook 6 を追加 |
| エージェントがテストなしで完了宣言 | Stop Hook | `.claude/settings.json` に Stop hook を追加 |

## 設計の原案

このテンプレートは以下の情報源をもとに設計しています。

| 情報源 | 取り込んだ機能 |
|-------|-------------|
| [Harness Engineering Best Practices 2026](https://nyosegawa.github.io/posts/harness-engineering-best-practices-2026/)（逆瀬川） | 4層品質モデル（PostToolUse → pre-commit → CI → Human review）、PreToolUse による破壊的操作ブロック、PostToolUse による即時フォーマット、設定ファイル保護の考え方、Biome / Lefthook の選定根拠 |
| [Claude Code Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks) (Anthropic) | `PreToolUse` / `PostToolUse` hook の仕組み、`guard.sh` と `post-tool-format.sh` の実装パターン |
| [AGENTS.md](https://openai.com/index/introducing-agents-md/) (OpenAI) | エージェント非依存の指示ファイル規約。Claude Code / Codex / Cursor 等どのツールでも読める共通フォーマット |
| [CLAUDE.md](https://docs.anthropic.com/en/docs/claude-code/memory#claudemd) (Anthropic) | Claude Code 固有の設定・ワークフロー指示ファイル |
| [ADR (Architecture Decision Records)](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) (Michael Nygard) | `docs/decisions/` による設計判断の記録。なぜその技術を選んだかを不変に残す |
| [Backstage](https://backstage.io/) (Spotify) | `catalog-info.yaml` によるサービスカタログ・メタデータ管理 |
| [Biome](https://biomejs.dev/) | ESLint + Prettier を統合した高速 Linter / Formatter（[ADR-0001](docs/decisions/0001-use-biome.md)） |
| [Lefthook](https://github.com/evilmartians/lefthook) (Evil Martians) | Go 製の高速 Git hooks マネージャ。並列実行対応（[ADR-0002](docs/decisions/0002-use-lefthook.md)） |

## License

MIT © [Geolonia Inc.](https://geolonia.com)
