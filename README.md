# snkを使ってGitHubプロフィールにSnakeを表示する方法

このリポジトリは、**GitHub のコントリビューショングリッドを使った Snake（platane/snk）をプロフィール README に表示する手順**を分かりやすくまとめたサンプルです。以下の内容はそのままコピペして使えるように作っています。

---

## 概要

* `Platane/snk` GitHub Action を使って、毎日（または手動）で `github-snake.svg` を自動生成します。
* 生成した SVG は `output` ブランチ（この例）に push され、README から `raw.githubusercontent.com` 経由で参照します。

---

## 前提

* GitHub アカウントがあること
* プロフィール用リポジトリ（**ユーザー名と同じリポジトリ名**）を作ること（例: `your-username/your-username`）
* Actions が使える（通常は有効）

> このリポジトリ（ドキュメント）は教えるためのサンプルです。実際に手を動かすときは `your-username` をあなたの GitHub ユーザー名（例: `toka-kun`）に置き換えてください。

---

## 全体の手順（サマリ）

1. GitHub にプロフィール用リポジトリを作成（名前はユーザー名と一致）
2. リポジトリに `.github/workflows/generate-snake.yml` を追加
3. README.md に SVG を埋め込む HTML を追加
4. Actions を実行して `output` ブランチを生成（初回は手動実行がおすすめ）

---

## ファイル：`.github/workflows/generate-snake.yml`

このワークフローをそのまま `.github/workflows/generate-snake.yml` に貼り付けてください。

```yaml
name: Generate Snake

on:
  schedule:
    - cron: '0 0 * * *'   # 毎日 00:00 UTC に実行（必要に応じて変更）
  workflow_dispatch: {}

permissions:
  contents: write

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      - name: Generate snake SVGs
        uses: Platane/snk@v3
        with:
          # ここをあなたの GitHub ユーザー名に置き換える
          github_user_name: your-username
          outputs: |
            dist/github-snake.svg
            dist/github-snake-dark.svg?palette=github-dark
          # github_token は自動で渡されるので指定は不要

      - name: Push snake to output branch
        uses: crazy-max/ghaction-github-pages@v4
        with:
          target_branch: output
          build_dir: dist
          allow_empty_commit: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**ポイント**

* `permissions: contents: write` を必ず追加してください（これがないと `git push` が 403 になります）。
* `your-username` を自分のユーザー名に置き換えるのを忘れないでください。

---

## ファイル：`README.md` に貼るコード（そのまま）

以下を README の表示したい場所に貼ってください（`your-username` を置き換え）。

```html
<picture>
  <source media="(prefers-color-scheme: dark)"
          srcset="https://raw.githubusercontent.com/your-username/your-username/output/github-snake-dark.svg" />
  <source media="(prefers-color-scheme: light)"
          srcset="https://raw.githubusercontent.com/your-username/your-username/output/github-snake.svg" />
  <img alt="github-snake"
       src="https://raw.githubusercontent.com/your-username/your-username/output/github-snake.svg" />
</picture>
```

---

## 初回実行（確認手順）

1. ファイルをコミットして GitHub に push
2. リポジトリの `Actions` → `Generate Snake` → `Run workflow` をクリック
3. Workflow が完了すると `output` ブランチが作成され、`github-snake.svg` が生成されます
4. README を開いて画像が表示されることを確認

---

## カスタマイズ例

### 色を変える

`outputs` のオプションで snake やドット（草）の色、パレットを指定できます。例：

```
outputs: |
  dist/github-snake.svg?color_snake=orange&color_dots=#bfd6f6,#8dbdff,#64a1f4,#4b91f1,#3c7dd9&color_background=#aaaaaa
```

### SVG のみ生成したい

GIF を生成しない（高速化）したい場合は `Platane/snk/svg-only@v3` を利用できます。

---

## トラブルシューティング

* **403: Permission denied**

  * ワークフローに `permissions: contents: write` を追加してください。

* **output ブランチが作成されない / 画像がない**

  * Actions の実行ログを確認。`Platane/snk` ステップで `writing to dist/github-snake.svg` のログがあるかどうか確認。
  * `Push snake to output branch` ステップで `git push` に失敗していないか確認（403 が出る場合は権限不足）。

* **README に画像が表示されない**

  * `raw.githubusercontent.com/ユーザー名/リポジトリ名/output/github-snake.svg` の URL をブラウザで直接開き、SVG が取得できるか確認。

---

## FAQ

* **snk を fork する必要はある？**

  * いいえ。`Platane/snk@v3` は Actions として直接使えます。fork は不要。

* **生成物は main ブランチに入る？**

  * いいえ。ワークフロー内で生成され `output` ブランチなどに push されます（ワークフロー次第）。main にコミットする必要はありません。

---

## サンプルファイル構成

```
your-username/your-username
├─ .github/
│  └─ workflows/
│     └─ generate-snake.yml
└─ README.md   <-- ここに picture タグを貼る
```

---

## 参考

* snk (Platane): [https://github.com/Platane/snk](https://github.com/Platane/snk)

---

## ライセンス

このドキュメントは MIT ライクに自由に使って構いません。必要に応じて内容をあなたのリポジトリに合わせて編集してください。

---
