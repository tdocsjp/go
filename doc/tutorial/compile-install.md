---
title: "アプリケーションをコンパイルしてインストールする"
---

# アプリケーションをコンパイルしてインストールする

この最後のトピックでは、新しい `go` コマンドをいくつか学びます。`go run` コマンドは頻繁に変更を加えながらプログラムをコンパイル・実行する際に便利な短縮方法ですが、バイナリの実行ファイルは生成しません。

このトピックでは、コードをビルドするための2つの追加コマンドを紹介します。

- [`go build` コマンド](/cmd/go/#hdr-Compile_packages_and_dependencies) は、パッケージとその依存関係をコンパイルしますが、結果をインストールはしません。
- [`go install` コマンド](/ref/mod#go-install) は、パッケージをコンパイルしてインストールします。

> **Note:** このトピックは [Go モジュールを作成する](/doc/tutorial/create-module) から始まる、複数パートからなるチュートリアルの一部です。

1. hello ディレクトリのコマンドラインで、`go build` コマンドを実行してコードを実行ファイルにコンパイルします。

   ```
   $ go build
   ```

2. hello ディレクトリのコマンドラインで、新しくできた `hello` 実行ファイルを実行し、コードが動作することを確認します。

   テスト後に greetings.go のコードを変更したかどうかによって、結果が異なる場合があることに注意してください。

   - Linux や Mac の場合:

     ```
     $ ./hello
     map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
     ```

   - Windows の場合:

     ```
     $ hello.exe
     map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
     ```

   これでアプリケーションを実行ファイルにコンパイルし、実行できるようになりました。しかし現状では、実行するにはプロンプトが実行ファイルのディレクトリにあるか、実行ファイルのパスを指定する必要があります。

   次は、パスを指定せずに実行できるように実行ファイルをインストールします。

3. `go` コマンドが現在のパッケージをインストールする、Go のインストールパスを確認します。

   [`go list` コマンド](/cmd/go/#hdr-List_packages_or_modules) を次のように実行すると、インストールパスを確認できます。

   ```
   $ go list -f '{{.Target}}'
   ```

   例えば、このコマンドの出力は `/home/gopher/bin/hello` のようになり、バイナリが /home/gopher/bin にインストールされることを意味します。次のステップでこのインストールディレクトリが必要になります。

4. Go のインストールディレクトリを、システムのシェルパスに追加します。

   これにより、実行ファイルがどこにあるかを指定せずにプログラムの実行ファイルを実行できるようになります。

   - Linux や Mac では、次のコマンドを実行します。

     ```
     $ export PATH=$PATH:/path/to/your/install/directory
     ```

   - Windows では、次のコマンドを実行します。

     ```
     $ set PATH=%PATH%;C:\path\to\your\install\directory
     ```

   代わりに、すでに `$HOME/bin` のようなディレクトリがシェルパスにあり、そこに Go のプログラムをインストールしたい場合は、[`go env` コマンド](/cmd/go/#hdr-Print_Go_environment_information) を使って `GOBIN` 変数を設定することで、インストール先を変更できます。

   ```
   $ go env -w GOBIN=/path/to/your/bin
   ```

   または

   ```
   $ go env -w GOBIN=C:\path\to\your\bin
   ```

5. シェルパスを更新したら、`go install` コマンドを実行してパッケージをコンパイル・インストールします。

   ```
   $ go install
   ```

6. アプリケーション名を入力するだけでアプリケーションを実行します。これを面白くするために、新しいコマンドプロンプトを開いて、別のディレクトリで `hello` 実行ファイル名を実行してみましょう。

   ```
   $ hello
   map[Darrin:Hail, Darrin! Well met! Gladys:Great to see you, Gladys! Samantha:Hail, Samantha! Well met!]
   ```

これで、この Go チュートリアルは終わりです！

---

**前へ:** [テストを追加する](/doc/tutorial/add-a-test)　|　**次へ:** [まとめと参考情報](/doc/tutorial/module-conclusion)

本ページは [go.dev/doc/tutorial/compile-install](https://go.dev/doc/tutorial/compile-install) の非公式日本語訳です。原文の著作権は The Go Authors に帰属し、[Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/) で提供されています。
