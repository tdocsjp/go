---
title: "複数人への挨拶を返す"
---

# 複数人への挨拶を返す

モジュールのコードに加える最後の変更として、一度のリクエストで複数人への挨拶を取得できるようにします。つまり、複数の値からなる入力を処理し、その入力内の値を複数の値からなる出力と対応付けます。そのためには、それぞれに対する挨拶を返せる関数に、名前の集合を渡す必要があります。

> **Note:** このトピックは [Go モジュールを作成する](/doc/tutorial/create-module) から始まる、複数パートからなるチュートリアルの一部です。

ただし、1つ問題があります。`Hello` 関数のパラメータを単一の名前から名前の集合に変更すると、関数のシグネチャが変わってしまいます。すでに `example.com/greetings` モジュールを公開していて、ユーザーが既に `Hello` を呼び出すコードを書いていたら、その変更はユーザーのプログラムを壊してしまいます。

このような状況では、別の名前で新しい関数を書くほうがよい選択です。新しい関数は複数のパラメータを受け取ります。これにより、後方互換性のために古い関数もそのまま残せます。

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

   // Hellos returns a map that associates each of the named people
   // with a greeting message.
   func Hellos(names []string) (map[string]string, error) {
       // A map to associate names with messages.
       messages := make(map[string]string)
       // Loop through the received slice of names, calling
       // the Hello function to get a message for each name.
       for _, name := range names {
           message, err := Hello(name)
           if err != nil {
               return nil, err
           }
           // In the map, associate the retrieved message with
           // the name.
           messages[name] = message
       }
       return messages, nil
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

       // Return one of the message formats selected at random.
       return formats[rand.Intn(len(formats))]
   }
   ```

   このコードでは、次のことをしています。

   - パラメータが単一の名前ではなく名前のスライスである `Hellos` 関数を追加しています。また、戻り値の型の1つを `string` から `map` に変更し、名前と挨拶メッセージを対応付けて返せるようにしています。
   - 新しい `Hellos` 関数から既存の `Hello` 関数を呼び出しています。これにより重複を減らしつつ、両方の関数をそのまま残せます。
   - 受け取った各名前（キー）に、生成したメッセージ（値）を対応付ける `messages` マップを作成しています。Go では、`make(map[`*キーの型*`]`*値の型*`)` という構文でマップを初期化します。`Hellos` 関数はこのマップを呼び出し元に返します。マップについての詳細は、Go blog の [Go maps in action](/blog/maps) を参照してください。
   - 受け取った名前をループし、それぞれが空でない値かどうかを確認してから、各名前にメッセージを対応付けています。この `for` ループでは、`range` が2つの値 -- ループ内の現在の項目のインデックスと、その項目の値のコピー -- を返します。インデックスは不要なので、Go のブランク識別子（アンダースコア）を使って無視しています。詳しくは Effective Go の [The blank identifier](/doc/effective_go.html#blank) を参照してください。

2. hello/hello.go の呼び出し側のコードで、名前のスライスを渡し、返ってきた名前/メッセージのマップの内容を出力します。

   hello.go で、コードを次のように変更します。

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

       // A slice of names.
       names := []string{"Gladys", "Samantha", "Darrin"}

       // Request greeting messages for the names.
       messages, err := greetings.Hellos(names)
       if err != nil {
           log.Fatal(err)
       }
       // If no error was returned, print the returned map of
       // messages to the console.
       fmt.Println(messages)
   }
   ```

   これらの変更によって、次のことをしています。

   - 3つの名前を保持するスライス型の `names` 変数を作成しています。
   - `names` 変数を `Hellos` 関数の引数として渡しています。

3. コマンドラインで hello/hello.go を含むディレクトリに移動し、`go run` を使ってコードが動作することを確認します。

   出力は、名前とメッセージを対応付けたマップの文字列表現で、次のようなものになるはずです。

   ```
   $ go run .
   map[Darrin:Hail, Darrin! Well met! Gladys:Hi, Gladys. Welcome! Samantha:Hail, Samantha! Well met!]
   ```

このトピックでは、名前と値のペアを表現するためのマップを紹介しました。また、モジュールの新規または変更された機能のために新しい関数を実装することで、後方互換性を保つという考え方も紹介しました。後方互換性についての詳細は、[モジュールの互換性を保つ](/blog/module-compatibility) を参照してください。

次は、組み込みの Go の機能を使ってコードのユニットテストを作成します。

---

**前へ:** [ランダムな挨拶を返す](/doc/tutorial/random-greeting)　|　**次へ:** [テストを追加する](/doc/tutorial/add-a-test)

本ページは [go.dev/doc/tutorial/greetings-multiple-people](https://go.dev/doc/tutorial/greetings-multiple-people) の非公式日本語訳です。原文の著作権は The Go Authors に帰属し、[Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/) で提供されています。
