---
title: "テストを追加する"
---

# テストを追加する

コードが安定した状態になったので（ところで、ここまでよくできました）、テストを追加しましょう。開発中にコードをテストすることで、変更を加える際に紛れ込むバグを見つけられます。このトピックでは、`Hello` 関数のテストを追加します。

> **Note:** このトピックは [Go モジュールを作成する](/doc/tutorial/create-module) から始まる、複数パートからなるチュートリアルの一部です。

Go のユニットテストへの組み込みのサポートにより、開発を進めながらのテストが簡単になります。具体的には、命名規則、Go の `testing` パッケージ、そして `go test` コマンドを使うことで、テストを素早く書いて実行できます。

1. greetings ディレクトリに、greetings_test.go というファイルを作成します。

   ファイル名の末尾を _test.go にすることで、`go test` コマンドにこのファイルがテスト関数を含んでいることを伝えます。

2. greetings_test.go に、次のコードを貼り付けて保存します。

   ```go
   package greetings

   import (
       "testing"
       "regexp"
   )

   // TestHelloName calls greetings.Hello with a name, checking
   // for a valid return value.
   func TestHelloName(t *testing.T) {
       name := "Gladys"
       want := regexp.MustCompile(`\b`+name+`\b`)
       msg, err := Hello("Gladys")
       if !want.MatchString(msg) || err != nil {
           t.Errorf(`Hello("Gladys") = %q, %v, want match for %#q, nil`, msg, err, want)
       }
   }

   // TestHelloEmpty calls greetings.Hello with an empty string,
   // checking for an error.
   func TestHelloEmpty(t *testing.T) {
       msg, err := Hello("")
       if msg != "" || err == nil {
           t.Errorf(`Hello("") = %q, %v, want "", error`, msg, err)
       }
   }
   ```

   このコードでは、次のことをしています。

   - テスト対象のコードと同じパッケージにテスト関数を実装しています。
   - `greetings.Hello` 関数をテストするための2つのテスト関数を作成しています。テスト関数名は `Test`*名前* という形式を取り、*名前* は具体的なテストについて何かを説明します。また、テスト関数はパラメータとして `testing` パッケージの [`testing.T` 型](/pkg/testing/#T) へのポインタを受け取ります。このパラメータのメソッドを使って、テストからの報告やログ出力を行います。
   - 2つのテストを実装しています。

     - `TestHelloName` は `Hello` 関数を呼び出し、関数が正しい応答メッセージを返せるはずの `name` の値を渡します。呼び出しがエラーを返したり、予期しない応答メッセージ（渡した名前を含まないもの）を返したりした場合、`t` パラメータの [`Errorf` メソッド](/pkg/testing/#T.Errorf) を使ってコンソールにメッセージを出力します。
     - `TestHelloEmpty` は空文字列を渡して `Hello` 関数を呼び出します。このテストは、エラー処理が正しく動作することを確認するためのものです。呼び出しが空でない文字列を返したり、エラーを返さなかったりした場合、`t` パラメータの [`Errorf` メソッド](/pkg/testing/#T.Errorf) を使ってコンソールにメッセージを出力します。

3. greetings ディレクトリのコマンドラインで、[`go test` コマンド](/cmd/go/#hdr-Test_packages) を実行してテストを実行します。

   `go test` コマンドは、テストファイル（ファイル名の末尾が _test.go）の中にあるテスト関数（名前が `Test` で始まるもの）を実行します。`-v` フラグを追加すると、すべてのテストとその結果を一覧表示する詳細な出力が得られます。

   テストは成功するはずです。

   ```
   $ go test
   PASS
   ok      example.com/greetings   0.364s

   $ go test -v
   === RUN   TestHelloName
   --- PASS: TestHelloName (0.00s)
   === RUN   TestHelloEmpty
   --- PASS: TestHelloEmpty (0.00s)
   PASS
   ok      example.com/greetings   0.372s
   ```

4. `greetings.Hello` 関数をあえて壊して、失敗するテストを見てみます。

   `TestHelloName` テスト関数は、`Hello` 関数のパラメータとして指定した名前について戻り値を確認します。失敗するテスト結果を見るために、`greetings.Hello` 関数を、名前を含まないように変更します。

   greetings/greetings.go で、次のコードを `Hello` 関数の代わりに貼り付けます。強調された行が、`name` 引数が誤って削除されたかのように、関数が返す値を変更していることに注目してください。

   ```go
   // Hello returns a greeting for the named person.
   func Hello(name string) (string, error) {
       // If no name was given, return an error with a message.
       if name == "" {
           return name, errors.New("empty name")
       }
       // Create a message using a random format.
       // message := fmt.Sprintf(randomFormat(), name)
       message := fmt.Sprint(randomFormat())
       return message, nil
   }
   ```

5. greetings ディレクトリのコマンドラインで `go test` を実行し、テストを実行します。

   今回は `-v` フラグなしで `go test` を実行します。出力には失敗したテストの結果だけが含まれるようになり、テストが多数ある場合に便利です。`TestHelloName` テストは失敗するはずです -- `TestHelloEmpty` は引き続き成功します。

   ```
   $ go test
   --- FAIL: TestHelloName (0.00s)
       greetings_test.go:15: Hello("Gladys") = "Hail, %v! Well met!", <nil>, want match for `\bGladys\b`, nil
   FAIL
   exit status 1
   FAIL    example.com/greetings   0.182s
   ```

次の（そして最後の）トピックでは、ローカルで実行するためにコードをコンパイルしてインストールする方法を見ていきます。

---

**前へ:** [複数人への挨拶を返す](/doc/tutorial/greetings-multiple-people)　|　**次へ:** [アプリケーションをコンパイルしてインストールする](/doc/tutorial/compile-install)

本ページは [go.dev/doc/tutorial/add-a-test](https://go.dev/doc/tutorial/add-a-test) の非公式日本語訳です。原文の著作権は The Go Authors に帰属し、[Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/) で提供されています。
