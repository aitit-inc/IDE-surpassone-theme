# SurpassOne IDE Theme

コーラル 1 色 + グレースケールで構成した、フラット・モノクロームな IDE カラーテーマ
「SurpassOne」を IDE 横断で管理するためのアンブレラリポジトリ。

シンタックスで使う色はコーラルのみ。それ以外はグレーの濃淡で階層を作り、
ボーダーや影を排した「フラット」な見た目で領域を区別する。Dark / Light の 2 バリアント。

## IDE 別リポジトリ

テーマ本体は IDE ごとの独立リポジトリにある。このリポジトリ自体はそれらを追跡せず、
共通方針と横断ドキュメントだけを管理する。

| IDE | リポジトリ | 配信先 |
|---|---|---|
| VS Code / Cursor / VSCodium | [`aitit-inc/vscode-surpassone-theme`](https://github.com/aitit-inc/vscode-surpassone-theme) | VS Code Marketplace / Open VSX / GitHub Releases |
| Zed | [`aitit-inc/zed-surpassone-theme`](https://github.com/aitit-inc/zed-surpassone-theme) | Zed extension registry |

各サブディレクトリは独立した git リポジトリのため、ローカルでまとめて扱う場合は個別にクローンする:

```bash
git clone git@github.com:aitit-inc/IDE-surpassone-theme.git
cd IDE-surpassone-theme
git clone git@github.com:aitit-inc/vscode-surpassone-theme.git
git clone git@github.com:aitit-inc/zed-surpassone-theme.git
```

詳しい運用方針は [`CLAUDE.md`](CLAUDE.md) を参照。

## License

各サブリポジトリの `LICENSE`(MIT)を参照。
