---
title: "チュートリアル: Go を始める"
---

# チュートリアル: Go を始める

このチュートリアルでは、Go プログラミングの簡単な導入を行います。進めていく中で、次のことをします。

- Go をインストールする（まだの場合）。
- 簡単な "Hello, world" コードを書く。
- `go` コマンドを使ってコードを実行する。
- Go のパッケージ検索ツールを使って、自分のコードで使えるパッケージを探す。
- 外部モジュールの関数を呼び出す。

> **Note:** 他のチュートリアルについては [Tutorials](/doc/tutorial/) を参照してください。

## 前提条件

- **多少のプログラミング経験。** ここで出てくるコードはかなり単純ですが、関数についての知識があると役立ちます。
- **コードを編集するツール。** お使いのテキストエディタなら何でも構いません。多くのテキストエディタは Go を十分にサポートしています。よく使われるのは VSCode（無料）、GoLand（有料）、Vim（無料）です。
- **コマンド端末。** Go は Linux・Mac のどのターミナルでも、Windows の PowerShell や cmd でも問題なく動作します。

## Go をインストールする

[ダウンロードとインストール](/doc/install) の手順に従うだけです。

## コードを書く

Hello, World から始めましょう。

1. コマンドプロンプトを開き、ホームディレクトリに移動します。

   Linux や Mac の場合:

   ```
   cd
   ```

   Windows の場合:

   ```
   cd %HOMEPATH%
   ```

2. 最初の Go ソースコード用に hello ディレクトリを作成します。

   例えば、次のコマンドを使います。

   ```
   mkdir hello
   cd hello
   ```

3. コードの依存関係の追跡を有効にします。

   コードが他のモジュールに含まれるパッケージをインポートする場合、それらの依存関係はコード自身のモジュールを通じて管理します。そのモジュールは、パッケージを提供するモジュールを追跡する go.mod ファイルによって定義されます。この go.mod ファイルは、ソースコードリポジトリを含め、常にコードと一緒に置かれます。

   コードの依存関係の追跡を有効にするには、[`go mod init`](/ref/mod#go-mod-init) コマンドを実行して go.mod ファイルを作成し、コードが置かれるモジュールの名前を指定します。この名前はモジュールのモジュールパスです。

   実際の開発では、モジュールパスは通常ソースコードを置くリポジトリの場所になります。例えば、モジュールパスは `github.com/mymodule` のようになります。作成したモジュールを他の人が使えるように公開する予定がある場合、モジュールパスは Go のツールがモジュールをダウンロードできる場所でなければ*なりません*。モジュールパスの命名についての詳細は、[依存関係の管理](/doc/modules/managing-dependencies#naming_module) を参照してください。

   このチュートリアルでは、単に `example/hello` を使います。

   ```
   $ go mod init example/hello
   go: creating new go.mod: module example/hello
   ```

4. テキストエディタで、コードを書くための hello.go ファイルを作成します。

5. 次のコードを hello.go ファイルに貼り付けて保存します。

   ```go
   package main

   import "fmt"

   func main() {
       fmt.Println("Hello, World!")
   }
   ```

   これが Go のコードです。このコードでは、次のことをしています。

   - `main` パッケージを宣言しています（パッケージは関数をまとめる方法で、同じディレクトリ内のすべてのファイルで構成されます）。
   - よく使われる [`fmt` パッケージ](https://pkg.go.dev/fmt/) をインポートしています。これはコンソールへの出力を含む、テキストの整形のための関数を含みます。このパッケージは、Go をインストールしたときに手に入る[標準ライブラリ](https://pkg.go.dev/std)のパッケージの一つです。
   - コンソールにメッセージを出力する `main` 関数を実装しています。`main` 関数は、`main` パッケージを実行すると、デフォルトで実行されます。

6. コードを実行して、挨拶を確認します。

   ```
   $ go run .
   Hello, World!
   ```

   [`go run`](/cmd/go/#hdr-Compile_and_run_Go_program) コマンドは、Go で作業する際に使う多くの `go` コマンドの一つです。他のコマンドの一覧を得るには、次のコマンドを使います。

   ```
   $ go help
   ```

## 外部パッケージのコードを呼び出す

自分のコードで、誰かが既に実装しているかもしれないことをする必要がある場合、自分のコードで使える関数を持つパッケージを探すことができます。

1. 外部モジュールの関数を使って、出力するメッセージを少し面白くします。

   1. pkg.go.dev にアクセスし、["quote" パッケージを検索](https://pkg.go.dev/search?q=quote)します。
   2. 検索結果から、`rsc.io/quote` パッケージの v1 を見つけてクリックします（`rsc.io/quote/v4` の「Other major versions」として一覧表示されているはずです）。
   3. **Documentation** セクションの **Index** の下にある、コードから呼び出せる関数の一覧を確認します。ここでは `Go` 関数を使います。
   4. このページの先頭で、パッケージ `quote` が `rsc.io/quote` モジュールに含まれていることを確認します。

   pkg.go.dev サイトを使うと、自分のコードで使える関数を持つパッケージが含まれた、公開済みのモジュールを見つけられます。パッケージは `rsc.io/quote` のようなモジュールとして公開され、他の人が使えるようになっています。モジュールは時間とともに新しいバージョンで改善されていき、コードを改善されたバージョンを使うようにアップグレードできます。

2. Go のコードで `rsc.io/quote` パッケージをインポートし、その `Go` 関数の呼び出しを追加します。

   強調された行を追加した後、コードは次のようになります。

   ```go
   package main

   import "fmt"

   import "rsc.io/quote" // 追加

   func main() {
       fmt.Println(quote.Go()) // 追加
   }
   ```

3. 新しいモジュールの要件とチェックサムを追加します。

   Go は `quote` モジュールを要件として追加し、モジュールを認証するための go.sum ファイルも追加します。詳しくは Go Modules Reference の [モジュールの認証](/ref/mod#authenticating) を参照してください。

   ```
   $ go mod tidy
   go: finding module for package rsc.io/quote
   go: found rsc.io/quote in rsc.io/quote v1.5.2
   ```

4. コードを実行して、呼び出した関数が生成するメッセージを確認します。

   ```
   $ go run .
   Don't communicate by sharing memory, share memory by communicating.
   ```

   コードが `Go` 関数を呼び出し、コミュニケーションについての小粋なメッセージを出力していることに注目してください。

   `go mod tidy` を実行すると、インポートしたパッケージを含む `rsc.io/quote` モジュールが見つかり、ダウンロードされます。デフォルトでは最新バージョン（v1.5.2）がダウンロードされます。

## もっとコードを書く

この簡単な導入で、Go をインストールし、基本のいくつかを学びました。さらにコードを書くために、別のチュートリアル [Go モジュールを作成する](/doc/tutorial/create-module.html) を見てみてください。

---

本ページは [go.dev/doc/tutorial/getting-started](https://go.dev/doc/tutorial/getting-started) の非公式日本語訳です。原文の著作権は The Go Authors に帰属し、[Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/) で提供されています。
