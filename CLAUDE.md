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

## Shared Design (canonical は各サブリポ)

全 IDE 共通のコンセプト:

- **ワンカラー**: シンタックスで使う「色」はコーラルのみ。それ以外はグレーの濃淡で階層を作る
- **フラット**: ボーダーや影を極力排除し、背景色の差だけで領域を区別する
- **視認性**: 色数を減らす代わりに、トーンの段階で読みやすさを担保する

設計仕様(トーン階層・スコープ割り当て・フォントスタイル規則)の **正本(canonical)は
[`vscode-surpassone-theme/CLAUDE.md`](vscode-surpassone-theme/CLAUDE.md)**。
他 IDE への移植は VS Code 版の定義を基準に色を対応付ける。

## Version Sync

全 IDE でバージョン番号を揃える(現在いずれも `0.3.4`)。

- VS Code: `vscode-surpassone-theme/package.json` の `version`
- Zed: `zed-surpassone-theme/extension.toml` の `version`

片方だけ上げない。リリースは全チャンネルで同じバージョン番号に合わせる
(VS Code 側のリリース手順は `vscode-surpassone-theme/rules/release.md` を参照)。

## Working in this repo

- アンブレラ側で扱うのは **このファイルと横断ドキュメントのみ**。テーマの中身は各サブディレクトリで編集する。
- 特定 IDE のテーマを直すときは、まず対応するサブディレクトリに入り、そのリポジトリの `CLAUDE.md` / 規約に従う。
- 新しい IDE 向けポートを追加するときは、**新しい独立リポジトリ**として作り、このディレクトリ配下にクローンしたうえで、上の表と `.gitignore` にエントリを追加する。

## Organization

- 会社名: SurpassOne
- GitHub org: `aitit-inc`
- VS Code Marketplace publisher ID: `surpassone`
