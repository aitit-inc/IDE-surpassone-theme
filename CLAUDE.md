# CLAUDE.md

## Project Overview

SurpassOne IDE テーマファミリーの **アンブレラ(メタ)リポジトリ**。
IDE ごとに独立した配信リポジトリを 1 か所にまとめて作業するための「親ディレクトリ」であり、
共通の設計方針・横断的なドキュメントだけをここで管理する。

テーマ本体(JSON / TextMate スコープ定義)は各サブディレクトリ側にあり、
このリポジトリには **含めない**(後述のとおり `.gitignore` で除外)。

## Repository Layout

```
IDE-surpassone-theme/            ← このリポジトリ (aitit-inc/IDE-surpassone-theme)
├── CLAUDE.md                    ← 追跡する (このファイル)
├── README.md                    ← 追跡する
├── .gitignore                   ← サブディレクトリ2つを除外
├── vscode-surpassone-theme/     ← 別リポジトリ・このリポでは追跡しない
└── zed-surpassone-theme/        ← 別リポジトリ・このリポでは追跡しない
```

| ディレクトリ | 対象 IDE | GitHub リポジトリ | 配信レジストリ |
|---|---|---|---|
| `vscode-surpassone-theme/` | VS Code / Cursor / VSCodium | `aitit-inc/vscode-surpassone-theme` | VS Code Marketplace, Open VSX, GitHub Releases |
| `zed-surpassone-theme/` | Zed | `aitit-inc/zed-surpassone-theme` | Zed extension registry (`zed-industries/extensions`) |

## ⚠️ サブディレクトリは独立リポジトリ — 統合しない

2 つのサブディレクトリは **それぞれが独立した git リポジトリ + GitHub リポジトリ** であり、
このアンブレラリポジトリはそれらを **追跡しない**(`.gitignore` で除外済み)。これは必須の制約:

- **VS Code 系**: タグ push をトリガーに各サブリポの GitHub Actions が Marketplace / Open VSX / GitHub Releases へ自動公開する。`package.json` の `repository` もサブリポの URL を指す。
- **Zed**: extension registry (`zed-industries/extensions`) は **特定のリポジトリ + タグ/コミット** を参照して取り込む。配信元が独立リポジトリであることが前提。

そのため、トップレベルで `git add .` してサブディレクトリを取り込んだり、submodule 化したりしてはいけない。
各 IDE 向けの編集・コミット・タグ付け・リリースは **対応するサブディレクトリ内の git リポジトリで** 行う。

## Shared Design — 設計仕様 (canonical)

全 IDE 共通のコンセプト:

- **ワンカラー**: シンタックスで使う「色」はコーラルのみ。それ以外はグレーの濃淡で階層を作る
- **フラット**: ボーダーや影を極力排除し、背景色の差だけで領域を区別する
- **視認性**: 色数を減らす代わりに、トーンの段階で読みやすさを担保する

### Syntax Tonal Hierarchy

| 層 | Dark | Light | 対象 |
|---|---|---|---|
| コーラル (bold) | `#DC7C68` | `#C05248` | 関数定義, タグ, this, エスケープ, 見出し |
| コーラル (型) | `#D0806E` | `#9A5548` | 型, クラス, インターフェース, enum |
| コーラル (キーワード) | `#B88A80` | `#8A6358` | キーワード, デコレータ, CSS属性, JSON/YAMLキー |
| 明グレー | `#DEE0E5` | `#333333` | 変数, 関数呼出, プロパティ |
| 中グレー | `#B8B9BE` | `#4C4C4C` | 文字列, 数値, 定数, 属性 |
| 淡グレー (italic) | `#64656A` | `#A7A19E` | コメント |
| 句読点 | `#969AA0` | `#706A68` | 演算子, 括弧 |

### Font Style Rules

- **bold** → コーラル最濃トーン (`#DC7C68` 系: 関数定義・タグ・this・エスケープ・見出し) のみ
- **italic** → コメントのみ
- **underline** → 使わない

この表が全 IDE の正本。各 IDE への移植はこの定義を基準に色を対応付ける。
IDE ごとのスコープ差異(VS Code TextMate ⇄ Zed tree-sitter 等)は [`rules/release.md`](rules/release.md) のマッピングを参照。

## Development — ローカルプレビュー

テーマ JSON は保存即反映でプレビューできる(ビルドステップなし)。

- **VS Code / Cursor**: `publisher.name-version` 形式のシンボリックリンクで認識させる。
  ```bash
  cd vscode-surpassone-theme
  VERSION=$(node -p "require('./package.json').version")
  ln -s "$(pwd)" ~/.cursor/extensions/surpassone.surpassone-theme-$VERSION   # 初回のみ
  ```
  リンクがある間は JSON 保存で即反映。任意トークンのスコープは `Developer: Inspect Editor Tokens and Scopes` で確認できる。
- **Zed**: コマンドパレットで `zed: install dev extension` → `zed-surpassone-theme/` を指定。`themes/surpassone.json` 保存で再読込。tree-sitter キャプチャの確認は `zed --foreground` のログ等。

## Version Sync

全 IDE でバージョン番号を揃える(現在いずれも `0.3.4`)。

- VS Code: `vscode-surpassone-theme/package.json` の `version`
- Zed: `zed-surpassone-theme/extension.toml` の `version`

片方だけ上げない。リリースは全チャンネルで同じバージョン番号に合わせる
(完全なリリース手順は [`rules/release.md`](rules/release.md) を参照)。

## Working in this repo

- アンブレラ側で扱うのは **このファイルと横断ドキュメント(`rules/` 等)のみ**。テーマの中身・設定・配信ファイルは各サブディレクトリ側で編集する(サブディレクトリには独自の dev ドキュメントは置かず、設計・運用の正本はこのリポジトリに集約する)。
- 特定 IDE のテーマを直すときは、対応するサブディレクトリに入り、上の設計仕様と [`rules/release.md`](rules/release.md) に従う。
- 新しい IDE 向けポートを追加するときは、**新しい独立リポジトリ**として作り、このディレクトリ配下にクローンしたうえで、上の表と `.gitignore` にエントリを追加する。

## Organization

- 会社名: SurpassOne
- GitHub org: `aitit-inc`
- VS Code Marketplace publisher ID: `surpassone`

## 作業方針
**探索は広く、出力は狭く**: 調査・ブレスト・思考のときは視点を広く取って多角的に検討する。一方、ドキュメント・設計・コードとして出力するときは、結論を伝えるのに必要な最小限まで削る。「念のため」「ついでに書いておく」式の埋め草は読まれず、重要な部分を埋もれさせ、長期的には記述の不整合を生む — コストだけでメリットはない。
