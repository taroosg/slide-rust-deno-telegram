---
transition: "slide"
title: "Rust と Deno で Webassembly する Telegram Bot をつくった"
---

Rust と Deno で Webassembly する

Telegram Bot をつくった

---

::: block
自己紹介 {style=background:#3e62ad;width:1000px}
:::

--

|        |                                                                                                    |
| ------ | -------------------------------------------------------------------------------------------------- |
| Name   | 大杉太郎（たろさん，たろ先生）                                                                     |
| Age    | 34 （1987 年生まれ）                                                                               |
| Place  | 茨城県 -> 北海道 -> 東京都 -> 福岡県                                                               |
| Career | 会社員（医療系メーカー企画設計開発）<br>ジーズアカデミー東京チューター<br>ジーズアカデミー福岡講師 |
| Like1  | Rust，Node.js，本，旅行，断捨離                                                                    |
| Like2  | 🥃, 🍺, 🍷                                                                                         |

---

もくじ

- やったことの概要
- Rust とか Deno とか
- 実装したコードなどなど
- デモ
- まとめとか感想とか

---

やったことの概要

--

- 素因数分解のアプリをつくりたい

- Rust で書きたい

- 画面はつくりたくない

--

そうだ，WebAssembly を使おう！

--

WebAssembly はモダンなウェブブラウザーで実行できる新しいタイプのコードです。ネイティブに近いパフォーマンスで動作するコンパクトなバイナリー形式の低レベルなアセンブリ風言語です。さらに、 C/C++ や Rust のような言語のコンパイル対象となって、それらの言語をウェブ上で実行することができます。 WebAssembly は JavaScript と並行して動作するように設計されているため、両方を連携させることができます。

by MDN

--

- ロジックは Rust

- アプリは Deno

- Bot にすれば画面つくる必要なし！

---

Rust ??

--

- C 並にはやい！

- 型と所有権で安全！

- エラーメッセージやドキュメントがやさしい！

---

Deno ??

--

- デノ，ディーノ

- npm が不要な Node.js（TS）

- ES Module で記述する

---

成果物全体の流れ

--

- Deno のアプリケーションが Telegram に接続

- 処理は Rust で書いたものを読み込む

- Telegram から入力して動作

---

実装したコード

---

Rust

--

入力値を 2 から順に割る処理を再帰で実行する

--

```rust
pub fn prime_factorization(n: usize) -> Vec<usize> {
  fn push_number_to_array(number: usize, array: Vec<usize>) -> Vec<usize> {
    array.iter().chain([number].iter()).map(|&x| x).collect()
  }

  fn is_divisor_larger_than_sqrt_dividend(dividend: usize, divisor: usize) -> bool {
    divisor as f64 >= (dividend as f64).sqrt()
  }

  fn get_prime_number_array(dividend: usize, divisor: usize, result: Vec<usize>) -> Vec<usize> {
    match dividend {
      1 => result,
      _ => match dividend % divisor {
        0 => get_prime_number_array(
          dividend / divisor,
          divisor,
          push_number_to_array(divisor, result),
        ),
        _ => match is_divisor_larger_than_sqrt_dividend(dividend, divisor) {
          true => push_number_to_array(dividend, result),
          false => get_prime_number_array(dividend, divisor + 1, result),
        },
      },
    }
  }
  get_prime_number_array(n, 2, vec![])
}
```

---

Deno

--

--

```ts
import { Bot } from "./deps.ts";
import init, { fib, prime_factorization } from "./pkg/rust_calculate_bot.js";

await init(Deno.readFile("./pkg/rust_calculate_bot_bg.wasm"));

// 数値かどうかをチェックする関数
const isNumber = (str: string): boolean => new RegExp(/^[0-9]+$/).test(str);

// レスポンスを作成する関数
const createResponse = (str: string | undefined): string =>
  !str
    ? "NaN"
    : !isNumber(str)
    ? "NaN"
    : !(Number(str) > 0)
    ? "0"
    : prime_factorization(Number(str)).join("\n");

const token = Deno.env.get("BOT_TOKEN") as string;

const bot = new Bot(token);

bot.on("text", async (ctx) => {
  await ctx.reply(createResponse(ctx.message?.text));
});

bot.launch();
```

---

デモ

---

まとめとか感想とか

--

- Rust のコードをビルドして読み込むだけ！

- Rust の処理は入出力を明確にするとやりやすい

- 実装の幅が広がる．．！！
