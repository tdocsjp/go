---
title: "別のモジュールからコードを呼び出す"
---

# 別のモジュールからコードを呼び出す

[前のセクション](/doc/tutorial/create-module) では、`greetings` モジュールを作成しました。このセクションでは、作成したモジュールの `Hello` 関数を呼び出すコードを書きます。アプリケーションとして実行できるコードを書き、そこから `greetings` モジュールのコードを呼び出します。

> **Note:** このトピックは [Go モジュールを作成する](/doc/tutorial/create-module) から始まる、複数パートからなるチュートリアルの一部です。

1. Go モジュールのソースコード用に `hello` ディレクトリを作成します。ここに呼び出し側のコードを書きます。

   このディレクトリを作成すると、階層内の同じ階層に hello と greetings の両方のディレクトリがあるはずです。

   ```
   <home>/
    |-- greetings/
    |-- hello/
   ```

   例えば、コマンドプロンプトが greetings ディレクトリにある場合、次のコマンドを使えます。

   ```
   cd ..
   mkdir hello
   cd hello
   ```

2. これから書くコードの依存関係の追跡を有効にします。

   コードの依存関係の追跡を有効にするには、[`go mod init` コマンド](/ref/mod#go-mod-init) を実行し、コードが置かれるモジュールの名前を指定します。

   このチュートリアルでは、モジュールパスとして `example.com/hello` を使います。

   ```
   $ go mod init example.com/hello
   go: creating new go.mod: module example.com/hello
   ```

3. テキストエディタで、hello ディレクトリ内にコードを書くためのファイルを作成し、hello.go と名付けます。

4. `Hello` 関数を呼び出し、その戻り値を出力するコードを書きます。

   そのために、次のコードを hello.go に貼り付けます。

   ```go
   package main

   import (
       "fmt"

       "example.com/greetings"
   )

   func main() {
       // Get a greeting message and print it.
       message := greetings.Hello("Gladys")
       fmt.Println(message)
   }
   ```

   このコードでは、次のことをしています。

   - `main` パッケージを宣言しています。Go では、アプリケーションとして実行されるコードは `main` パッケージに置かれなければなりません。
   - `example.com/greetings` と [`fmt` パッケージ](https://pkg.go.dev/fmt/) の2つのパッケージをインポートしています。これにより、コードはそれらのパッケージの関数にアクセスできます。`example.com/greetings`（先ほど作成したモジュールに含まれるパッケージ）をインポートすることで、`Hello` 関数にアクセスできるようになります。また、入出力テキストを扱う関数（コンソールへのテキスト出力など）を持つ `fmt` もインポートしています。
   - `greetings` パッケージの `Hello` 関数を呼び出して挨拶を取得しています。

5. ローカルの `example.com/greetings` モジュールを使うように、`example.com/hello` モジュールを編集します。

   本番環境での利用であれば、`example.com/greetings` モジュールはそのリポジトリから（公開された場所を反映したモジュールパスで）公開し、Go のツールがそれを見つけてダウンロードできるようにします。しかし今はまだモジュールを公開していないので、`example.com/hello` モジュールがローカルのファイルシステム上にある `example.com/greetings` のコードを見つけられるように調整する必要があります。

   そのためには、[`go mod edit` コマンド](/ref/mod#go-mod-edit) を使って `example.com/hello` モジュールを編集し、モジュールパス（モジュールが存在しない場所）からローカルディレクトリ（モジュールが実際にある場所）へ Go のツールの参照先を変更します。

   1. hello ディレクトリのコマンドプロンプトから、次のコマンドを実行します。

      ```
      $ go mod edit -replace example.com/greetings=../greetings
      ```

      このコマンドは、依存関係の場所を特定する際に `example.com/greetings` を `../greetings` に置き換えるべきことを指定します。コマンドを実行すると、hello ディレクトリの go.mod ファイルに [`replace` ディレクティブ](/doc/modules/gomod-ref#replace) が含まれるはずです。

      ```
      module example.com/hello

      go 1.16

      replace example.com/greetings => ../greetings
      ```

   2. hello ディレクトリのコマンドプロンプトから、[`go mod tidy` コマンド](/ref/mod#go-mod-tidy) を実行して `example.com/hello` モジュールの依存関係を同期し、コードが必要とするがまだモジュールで追跡されていない依存関係を追加します。

      ```
      $ go mod tidy
      go: found example.com/greetings in example.com/greetings v0.0.0-00010101000000-000000000000
      ```

      コマンドが完了すると、`example.com/hello` モジュールの go.mod ファイルは次のようになります。

      ```
      module example.com/hello

      go 1.16

      replace example.com/greetings => ../greetings

      require example.com/greetings v0.0.0-00010101000000-000000000000
      ```

      このコマンドは greetings ディレクトリ内のローカルコードを見つけ、`example.com/hello` が `example.com/greetings` を必要とすることを示す [`require` ディレクティブ](/doc/modules/gomod-ref#require) を追加しました。この依存関係は、hello.go で `greetings` パッケージをインポートした時点で生まれたものです。

      モジュールパスの後に続く番号は、*疑似バージョン番号*です -- （まだそのモジュールが持っていない）セマンティックバージョン番号の代わりに使われる生成された番号です。

      *公開された*モジュールを参照する場合、go.mod ファイルには通常 `replace` ディレクティブはなく、末尾にタグ付けされたバージョン番号を持つ `require` ディレクティブが使われます。

      ```
      require example.com/greetings v1.1.0
      ```

      バージョン番号についての詳細は、[モジュールのバージョン番号付け](/doc/modules/version-numbers) を参照してください。

6. `hello` ディレクトリのコマンドプロンプトで、コードを実行して動作を確認します。

   ```
   $ go run .
   Hi, Gladys. Welcome!
   ```

おめでとうございます！ 2つの動作するモジュールを書き上げました。

次のトピックでは、エラー処理を追加します。

---

**前へ:** [Go モジュールを作成する](/doc/tutorial/create-module)　|　**次へ:** [エラーを返して処理する](/doc/tutorial/handle-errors)

本ページは [go.dev/doc/tutorial/call-module-code](https://go.dev/doc/tutorial/call-module-code) の非公式日本語訳です。原文の著作権は The Go Authors に帰属し、[Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/) で提供されています。
