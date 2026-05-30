# Release Runbook

新しいバージョンを **VS Code Marketplace / Open VSX / GitHub Releases / Zed extension registry** の
4 チャンネル全てへ配信する手順。デザインの正本(トーン階層・フォント規則)は [`../CLAUDE.md`](../CLAUDE.md) を参照。

## 前提

- バージョン番号は全チャンネルで揃える(例: `0.4.0`)。以下では `X.Y.Z` をプレースホルダとして使う。片方だけ上げない。
- このアンブレラリポジトリ(`aitit-inc/IDE-surpassone-theme`)配下に、2 つの配信リポジトリがサブディレクトリとしてクローン済みであることを前提とする:
  - `vscode-surpassone-theme/`(`aitit-inc/vscode-surpassone-theme`)— VS Code / Cursor 用。タグ push で GitHub Actions が自動公開。
  - `zed-surpassone-theme/`(`aitit-inc/zed-surpassone-theme`)— Zed 用。手動で更新し、`zed-industries/extensions` への PR で公開。
  - 以下の `cd` はアンブレラ直下からの相対パス。サブディレクトリは互いに兄弟なので、VS Code 側から Zed 側へは `cd ../zed-surpassone-theme` で移動できる。
- `zed-industries/extensions` の fork(`reouno/extensions`)は初回作成済み。再利用する。クローンが無ければ作業ディレクトリで `gh repo fork zed-industries/extensions --clone --default-branch-only` する。

## 手順

### 1. VS Code 側で色を更新

```bash
cd vscode-surpassone-theme

# テーマ JSON を編集
$EDITOR themes/surpassone-dark-color-theme.json themes/surpassone-light-color-theme.json

# version を bump
$EDITOR package.json   # "version": "X.Y.Z"

git add themes package.json
git commit -m "vX.Y.Z"
git push
```

### 2. VS Code / Cursor / GitHub Releases を公開

タグを push すると GitHub Actions(`.github/workflows/publish.yml`)が VS Code Marketplace・Open VSX・GitHub Releases へ自動公開する。

```bash
git tag vX.Y.Z
git push --tags
```

### 3. Zed 側に同じ色変更を反映

Zed のテーマ JSON は VS Code 版とフォーマットが異なるため、手動で対応する変更を入れる。マッピングは `themes/surpassone.json` 内で確立済み(初回ポート時に決定):

- VS Code `colors.*` → Zed `style.*`
- VS Code `tokenColors` / `semanticTokenColors` → Zed `style.syntax.*`
- 注意点(機械変換できない):
  - 関数の定義 vs 呼出: VS Code は TextMate スコープで分けているが、Zed は `function`(共通)+ `function.definition`(定義のみ、文法が出した場合)。
  - CSS property と JS object property: VS Code では別スコープだが Zed では同じ `property` に集約される。
  - JSON キー: Zed は `property.json_key` という JSON 専用 tree-sitter キャプチャを使う(一般の `property` とは別。これだけ上書きすれば他言語に影響しない)。
  - decorator / annotation: Zed では `attribute` に集約。

作業:

```bash
cd ../zed-surpassone-theme

# themes/surpassone.json の Dark / Light 両方の style と syntax を更新
$EDITOR themes/surpassone.json

# extension.toml の version を更新
$EDITOR extension.toml   # version = "X.Y.Z"

git add themes/surpassone.json extension.toml
git commit -m "vX.Y.Z"
git push

# Zed 用にもタグを打つ(慣習合わせ。必須ではない)
git tag vX.Y.Z
git push --tags
```

### 4. Zed extension registry の submodule を更新して PR

`reouno/extensions` の fork を最新化し、submodule の参照 SHA と `extensions.toml` のバージョンを更新する。`pnpm` は不要(`node src/sort-extensions.js` を直接実行する)。

```bash
# fork のクローンへ移動(無ければ gh repo fork で取得)
cd /path/to/extensions

# upstream に追従
git fetch upstream
git checkout main
git merge --ff-only upstream/main
git push origin main

# 作業ブランチ
git checkout -b bump-surpassone-theme-X.Y.Z

# submodule の参照を最新コミットへ
git submodule update --remote extensions/surpassone-theme

# extensions.toml の [surpassone-theme] のバージョンを更新
$EDITOR extensions.toml   # version = "X.Y.Z"

# sort スクリプトの依存(初回のみ; node_modules は gitignore 済み)
npm install --no-save --no-audit --no-fund @iarna/toml@2.2.5 git-submodule-js@1.0.5

# 並びを正規化
node src/sort-extensions.js

# コミット & push & PR
git add .gitmodules extensions.toml extensions/surpassone-theme
git commit -m "Bump surpassone-theme to X.Y.Z"
git push -u origin bump-surpassone-theme-X.Y.Z

gh pr create --repo zed-industries/extensions --base main \
  --head reouno:bump-surpassone-theme-X.Y.Z \
  --title "Bump surpassone-theme to X.Y.Z" \
  --body "Update surpassone-theme to vX.Y.Z. See https://github.com/aitit-inc/zed-surpassone-theme/releases/tag/vX.Y.Z"
```

CLA は初回(PR #6272)で `reouno` が署名済みのため再署名不要。CI(`Danger` / `package`)が green になり次第、メンテナのマージで Zed registry に公開される。

## チェックリスト

- [ ] VS Code 版テーマ JSON 2 ファイルを編集
- [ ] `vscode-surpassone-theme/package.json` の version を bump
- [ ] VS Code リポジトリにコミット & push
- [ ] タグ `vX.Y.Z` を push(GitHub Actions が走るのを確認)
- [ ] Zed 版 `themes/surpassone.json` を同じ意図で更新
- [ ] `zed-surpassone-theme/extension.toml` の version を更新
- [ ] Zed リポジトリにコミット & push & タグ
- [ ] `extensions` fork で submodule 更新 + `extensions.toml` のバージョン更新 + sort
- [ ] PR を作成、CI green を確認

## 参考

- VS Code 側の Secrets: `VSCE_PAT`(Marketplace), `OVSX_PAT`(Open VSX)
- Zed テーマスキーマ: https://zed.dev/schema/themes/v0.2.0.json
- Zed 拡張公開ドキュメント: https://zed.dev/docs/extensions/developing-extensions
