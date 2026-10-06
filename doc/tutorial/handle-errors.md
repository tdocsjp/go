---
title: "エラーを返して処理する"
---

# エラーを返して処理する

エラー処理は、堅実なコードに不可欠な機能です。このセクションでは、greetings モジュールからエラーを返すコードを少し追加し、それを呼び出し側で処理します。

> **Note:** このトピックは [Go モジュールを作成する](/doc/tutorial/create-module) から始まる、複数パートからなるチュートリアルの一部です。

1. greetings/greetings.go に、以下のコードを追加します。

   誰に挨拶すればいいか分からないのに挨拶を返すのは意味がありません。名前が空の場合は呼び出し元にエラーを返しましょう。次のコードを greetings.go にコピーして保存してください。

   ```go
   package greetings

   import (
       "errors"
       "fmt"
   )

   // Hello returns a greeting for the named person.
   func Hello(name string) (string, error) {
       // If no name was given, return an error with a message.
       if name == "" {
           return "", errors.New("empty name")
       }

       // If a name was received, return a value that embeds the name
       // in a greeting message.
       message := fmt.Sprintf("Hi, %v. Welcome!", name)
       return message, nil
   }
   ```

   このコードでは、次のことをしています。

   - 関数が `string` と `error` という2つの値を返すように変更しています。呼び出し側は、2つめの値を見てエラーが発生したかを確認します。（Go の関数は複数の値を返せます。詳しくは [Effective Go](/doc/effective_go.html#multiple-returns) を参照してください。）
   - [`errors.New` 関数](https://pkg.go.dev/errors/#example-New) を使うために、Go 標準ライブラリの `errors` パッケージをインポートしています。
   - 不正なリクエスト（名前が入るべき場所が空文字列になっている）を確認する `if` 文を追加し、不正な場合はエラーを返しています。`errors.New` 関数は、指定したメッセージを含む `error` を返します。
   - 成功時の戻り値に、2つめの値として（エラーがないことを意味する）`nil` を追加しています。これにより、呼び出し側は関数が成功したことを確認できます。

2. hello/hello.go ファイルで、`Hello` 関数が返すようになったエラーを、エラーでない値とともに処理します。

   次のコードを hello.go に貼り付けます。

   ```go
   package main

   import (
       "fmt"
       "log"

       "example.com/greetings"
   )

   func main() {
       // Set properties of the predefined Logger, including
       // the log entry prefix and a flag to disable printing
       // the time, source file, and line number.
       log.SetPrefix("greetings: ")
       log.SetFlags(0)

       // Request a greeting message.
       message, err := greetings.Hello("")
       // If an error was returned, print it to the console and
       // exit the program.
       if err != nil {
           log.Fatal(err)
       }

       // If no error was returned, print the returned message
       // to the console.
       fmt.Println(message)
   }
   ```

   このコードでは、次のことをしています。

   - [`log` パッケージ](https://pkg.go.dev/log/) を設定し、ログメッセージの先頭にコマンド名（"greetings: "）を、タイムスタンプやソースファイル情報なしで出力するようにしています。
   - `Hello` の2つの戻り値（`error` を含む）をどちらも変数に代入しています。
   - `Hello` の引数を、Gladys の名前から空文字列に変更し、エラー処理のコードを試せるようにしています。
   - `nil` ではない `error` の値を確認しています。この場合、処理を続ける意味はありません。
   - 標準ライブラリの `log` パッケージの関数を使ってエラー情報を出力しています。エラーが発生した場合、`log` パッケージの [`Fatal` 関数](https://pkg.go.dev/log?tab=doc#Fatal) を使ってエラーを出力し、プログラムを停止しています。

3. `hello` ディレクトリのコマンドラインで hello.go を実行し、コードが動作することを確認します。

   今回は空の名前を渡しているので、エラーが発生します。

   ```
   $ go run .
   greetings: empty name
   exit status 1
   ```

これが Go における一般的なエラー処理です。エラーを値として返し、呼び出し側がそれを確認できるようにします。

次は、Go のスライスを使って、ランダムに選ばれた挨拶を返すようにします。

---

**前へ:** [別のモジュールからコードを呼び出す](/doc/tutorial/call-module-code)　|　**次へ:** [ランダムな挨拶を返す](/doc/tutorial/random-greeting)

本ページは [go.dev/doc/tutorial/handle-errors](https://go.dev/doc/tutorial/handle-errors) の非公式日本語訳です。原文の著作権は The Go Authors に帰属し、[Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/) で提供されています。
