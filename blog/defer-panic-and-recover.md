---
title: Defer、Panic、Recover
date: 2010-08-04
by:
- Andrew Gerrand
tags:
- defer
- panic
- recover
- technical
- function
summary: Go の defer・panic・recover という制御フローの仕組みへの入門。
---

Go には、if・for・switch・goto といった一般的な制御フローの仕組みがあります。また、別の goroutine でコードを実行するための go 文もあります。ここでは、あまり一般的ではない仕組み、defer・panic・recover について説明したいと思います。

**defer 文** は、関数呼び出しをリストに積みます。保存された呼び出しのリストは、周囲の関数が return した後に実行されます。defer は、さまざまな後始めの処理を行う関数を簡潔にするためによく使われます。

例として、2つのファイルを開き、片方の内容をもう片方にコピーする関数を見てみましょう。

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }

    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}
```

これは動きますが、バグがあります。os.Create の呼び出しが失敗すると、関数はソースファイルを閉じずに return してしまいます。これは2つめの return 文の前に src.Close の呼び出しを置くことで簡単に修正できますが、関数がもっと複雑であれば、この問題はそう簡単には気づかれず、解決もされないかもしれません。defer 文を導入することで、ファイルが常に閉じられることを保証できます。

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```

defer 文のおかげで、それぞれのファイルを開いた直後にそれを閉じることを考えられるようになり、関数内の return 文の数にかかわらず、ファイルが*必ず*閉じられることが保証されます。

defer 文の動作は単純で予測可能です。3つの簡単なルールがあります。

1. *defer される関数の引数は、defer 文が評価されるときに評価される。*

   この例では、式 "i" は Println の呼び出しが defer されるときに評価されます。defer された呼び出しは、関数が return した後に "0" を出力します。

   ```go
   func a() {
       i := 0
       defer fmt.Println(i)
       i++
       return
   }
   ```

2. *defer された関数呼び出しは、周囲の関数が return した後、Last In First Out（後入れ先出し）の順序で実行される。*

   この関数は "3210" を出力します。

   ```go
   func b() {
       for i := 0; i < 4; i++ {
           defer fmt.Print(i)
       }
   }
   ```

3. *defer された関数は、return する関数の名前付き戻り値を読み取ったり、代入したりできる。*

   この例では、defer された関数が、周囲の関数が return した*後に*戻り値 i をインクリメントします。そのため、この関数は 2 を返します。

   ```go
   func c() (i int) {
       defer func() { i++ }()
       return 1
   }
   ```

   これは関数のエラーの戻り値を変更するのに便利です。この例はすぐ後で見ていきます。

**panic** は組み込み関数で、通常の制御フローを停止し、*パニック*を始めます。関数 F が panic を呼び出すと、F の実行は停止し、F 内の defer された関数は通常どおり実行され、その後 F は呼び出し元に戻ります。呼び出し元にとっては、F は panic の呼び出しのように振る舞います。この過程はスタックを上に向かって続き、現在の goroutine 内のすべての関数が return した時点で、プログラムはクラッシュします。panic は panic を直接呼び出すことで起こせます。また、配列の範囲外アクセスのような実行時エラーによっても引き起こされます。

**recover** は組み込み関数で、パニック中の goroutine の制御を取り戻します。recover は defer された関数の中でのみ有用です。通常の実行中に recover を呼び出すと、nil を返し、それ以外の効果はありません。現在の goroutine がパニック中であれば、recover を呼び出すと panic に渡された値を捕捉し、通常の実行を再開します。

panic と defer の仕組みを示す例のプログラムを見てみましょう。

```go
package main

import "fmt"

func main() {
    f()
    fmt.Println("Returned normally from f.")
}

func f() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in f", r)
        }
    }()
    fmt.Println("Calling g.")
    g(0)
    fmt.Println("Returned normally from g.")
}

func g(i int) {
    if i > 3 {
        fmt.Println("Panicking!")
        panic(fmt.Sprintf("%v", i))
    }
    defer fmt.Println("Defer in g", i)
    fmt.Println("Printing in g", i)
    g(i + 1)
}
```

関数 g は int の i を受け取り、i が 3 より大きければ panic し、そうでなければ引数 i+1 で自分自身を呼び出します。関数 f は recover を呼び出し、（nil でなければ）回復した値を出力する関数を defer します。この先を読む前に、このプログラムの出力がどうなるか想像してみてください。

このプログラムは次のように出力します。

	Calling g.
	Printing in g 0
	Printing in g 1
	Printing in g 2
	Printing in g 3
	Panicking!
	Defer in g 3
	Defer in g 2
	Defer in g 1
	Defer in g 0
	Recovered in f 4
	Returned normally from f.

f から defer された関数を取り除くと、panic は回復されずに goroutine のコールスタックの最上位まで達し、プログラムを終了させます。この変更後のプログラムは次のように出力します。

	Calling g.
	Printing in g 0
	Printing in g 1
	Printing in g 2
	Printing in g 3
	Panicking!
	Defer in g 3
	Defer in g 2
	Defer in g 1
	Defer in g 0
	panic: 4

	panic PC=0x2a9cd8
	[stack trace omitted]

**panic** と **recover** の実際の例としては、Go 標準ライブラリの [json パッケージ](/pkg/encoding/json/) を見てください。このパッケージは、再帰的な関数の集合でインターフェースをエンコードします。値を走査中にエラーが発生すると、panic を呼び出してスタックをトップレベルの関数呼び出しまで巻き戻し、そこで panic から回復して適切なエラー値を返します（[encode.go](/src/pkg/encoding/json/encode.go) の encodeState 型の 'error' メソッドと 'marshal' メソッドを参照してください）。

Go のライブラリでの慣習は、パッケージが内部で panic を使っていても、その外部 API は依然として明示的なエラーの戻り値を提示するということです。

**defer** のその他の使い方（前述の file.Close の例以外にも）には、mutex の解放があります。

```go
mu.Lock()
defer mu.Unlock()
```

フッターの出力:

```go
printHeader()
defer printFooter()
```

など、他にもいろいろあります。

まとめると、defer 文は（panic や recover の有無にかかわらず）制御フローのための、珍しくかつ強力な仕組みを提供します。他のプログラミング言語で専用の構造によって実装されている多くの機能を、この仕組みでモデル化できます。試してみてください。

---

本ページは [go.dev/blog/defer-panic-and-recover](https://go.dev/blog/defer-panic-and-recover) の非公式日本語訳です。原文の著作権は The Go Authors に帰属し、[Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/) で提供されています。
