# tdocsjp/go

Go 公式サイト（[go.dev](https://go.dev)、ソースは [golang/website](https://github.com/golang/website) の `_content/`）の非公式日本語訳。[tdocsjp/website](https://github.com/tdocsjp/website) の submodule として取り込まれる。

ページは go.dev の URL パスをそのままミラーする（`_content/doc/tutorial/getting-started.html` → `/doc/tutorial/getting-started` → このリポジトリの `doc/tutorial/getting-started.md` など）。`index.md` はトップページ（翻訳済みページへの目次）。

- 原文: https://github.com/golang/website の `_content/`
- ライセンス: サイトの内容は [Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/)、コードは BSD license。著作権は The Go Authors に帰属する
- 本訳は非公式であり、Go プロジェクトによる承認を受けたものではない

## 翻訳済み

- `doc/tutorial/index.md` — [go.dev/doc/tutorial](https://go.dev/doc/tutorial)（チュートリアル一覧）
- `doc/tutorial/getting-started.md` — [go.dev/doc/tutorial/getting-started](https://go.dev/doc/tutorial/getting-started)
- `doc/tutorial/create-module.md`、`call-module-code.md`、`handle-errors.md`、`random-greeting.md`、`greetings-multiple-people.md`、`add-a-test.md`、`compile-install.md`、`module-conclusion.md` — 「Create a Go module」チュートリアルシリーズ全7トピックを完訳
- `blog/errors-are-values.md` — [go.dev/blog/errors-are-values](https://go.dev/blog/errors-are-values)（Rob Pike「エラーは値である」）
- `blog/defer-panic-and-recover.md` — [go.dev/blog/defer-panic-and-recover](https://go.dev/blog/defer-panic-and-recover)

## 未翻訳

`_content/` の大部分が未着手。特に:

- `doc/`: FAQ（faq.md）、Effective Go（effective_go.html）、言語仕様へのリンク、modules リファレンスなど
- `doc/tutorial/`: workspaces・json・database-access・web-service-gin・generics・fuzz・govulncheck 系（上記以外の個別チュートリアル）
- `blog/`: 数百本の記事（上記2本以外）
- `learn/`、`solutions/`、`tour/`、`wiki/` など

必要になったページから追加していく。
