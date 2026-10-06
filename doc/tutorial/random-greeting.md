---
title: "ランダムな挨拶を返す"
---

# ランダムな挨拶を返す

このセクションでは、毎回同じ1つの挨拶を返す代わりに、あらかじめ定義した複数の挨拶メッセージのうち1つを返すようにコードを変更します。

> **Note:** このトピックは [Go モジュールを作成する](/doc/tutorial/create-module) から始まる、複数パートからなるチュートリアルの一部です。

これを行うには、Go のスライスを使います。スライスは配列に似ていますが、項目を追加・削除するとサイズが動的に変わる点が異なります。スライスは Go の中でも特に便利な型の一つです。

3つの挨拶メッセージを含む小さなスライスを追加し、コードがそのうちの1つをランダムに返すようにします。スライスについての詳細は、Go blog の [Go slices](/blog/slices-intro) を参照してください。

1. greetings/greetings.go で、コードを次のように変更します。

   ```go
   package greetings

   import (
       "errors"
       "fmt"
       "math/rand"
   )

   // Hello returns a greeting for the named person.
   func Hello(name string) (string, error) {
       // If no name was given, return an error with a message.
       if name == "" {
           return name, errors.New("empty name")
       }
       // Create a message using a random format.
       message := fmt.Sprintf(randomFormat(), name)
       return message, nil
   }

   // randomFormat returns one of a set of greeting messages. The returned
   // message is selected at random.
   func randomFormat() string {
       // A slice of message formats.
       formats := []string{
           "Hi, %v. Welcome!",
           "Great to see you, %v!",
           "Hail, %v! Well met!",
       }

       // Return a randomly selected message format by specifying
       // a random index for the slice of formats.
       return formats[rand.Intn(len(formats))]
   }
   ```

   このコードでは、次のことをしています。

   - 挨拶メッセージのフォーマットをランダムに1つ選んで返す `randomFormat` 関数を追加しています。`randomFormat` は小文字で始まっているため、同じパッケージ内のコードからしかアクセスできない（つまりエクスポートされていない）ことに注意してください。
   - `randomFormat` の中で、3つのメッセージフォーマットを持つ `formats` スライスを宣言しています。スライスを宣言する際は、`[]string` のように、角かっこの中のサイズを省略します。これにより、Go にはスライスの裏にある配列のサイズを動的に変更できることが伝わります。
   - [`math/rand` パッケージ](https://pkg.go.dev/math/rand/) を使って、スライスから項目を選ぶための乱数を生成しています。
   - `Hello` の中で `randomFormat` 関数を呼び出して返すメッセージのフォーマットを取得し、そのフォーマットと `name` の値を組み合わせてメッセージを作成しています。
   - 以前と同様に、メッセージ（またはエラー）を返しています。

2. hello/hello.go で、コードを次のように変更します。

   hello.go の `Hello` 関数呼び出しの引数として、Gladys の名前（またはお好みの別の名前）を追加するだけです。

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
       message, err := greetings.Hello("Gladys")
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

3. コマンドラインの hello ディレクトリで hello.go を実行し、コードが動作することを確認します。複数回実行して、挨拶が変わることに注目してください。

   ```
   $ go run .
   Great to see you, Gladys!

   $ go run .
   Hi, Gladys. Welcome!

   $ go run .
   Hail, Gladys! Well met!
   ```

次は、スライスを使って複数人に挨拶します。

---

**前へ:** [エラーを返して処理する](/doc/tutorial/handle-errors)　|　**次へ:** [複数人への挨拶を返す](/doc/tutorial/greetings-multiple-people)

本ページは [go.dev/doc/tutorial/random-greeting](https://go.dev/doc/tutorial/random-greeting) の非公式日本語訳です。原文の著作権は The Go Authors に帰属し、[Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/) で提供されています。
