<!--
# Static
-->
# スタティックライフタイム

<!--
Rust has a few reserved lifetime names. One of those is `'static`. You
might encounter it in two situations:
-->
Rustにはいくつかの予約されたライフタイム名があります。その1つが`static`で、2つの状況で使用することがあります。

```rust, editable
// A reference with 'static lifetime:
let s: &'static str = "hello world";

// 'static as part of a trait bound:
fn generic<T>(x: T) where T: 'static {}
```

Both are related but subtly different and this is a common source for
confusion when learning Rust. Here are some examples for each situation:

## Reference lifetime

As a reference lifetime `'static` indicates that the data pointed to by
the reference lives for the entire lifetime of the running program.
It can still be coerced to a shorter lifetime.

There are two ways to make a variable with `'static` lifetime, and both
are stored in the read-only memory of the binary:

<!--
* Make a constant with the `static` declaration.
* Make a `string` literal which has type: `&'static str`.
-->
* `static`宣言とともに定数を作成する。
* 文字列リテラル で`&'static str`型を持つ変数を作成する。

<!--
See the following example for a display of each method:
-->
では、それぞれの方法の例を見ていきましょう。

```rust,editable
// Make a constant with `'static` lifetime.
// `'static`ライフタイムを持つ定数を作成
static NUM: i32 = 18;

// Returns a reference to `NUM` where its `'static`
// lifetime is coerced to that of the input argument.
// `NUM`への参照を返す。ライフタイムは`'static`から引数の
// ライフタイムへと圧縮されている。
fn coerce_static<'a>(_: &'a i32) -> &'a i32 {
    &NUM
}

fn main() {
    {
        // Make a `string` literal and print it:
        // 文字列リテラルを用いて変数を作成し、プリントする
        let static_string = "I'm in read-only memory";
        println!("static_string: {}", static_string);

        // When `static_string` goes out of scope, the reference
        // can no longer be used, but the data remains in the binary.
        // `static_string`がスコープから抜けると、参照は使用することが
        // できなくなるが、データはバイナリ中に残る。
    }

    {
        // Make an integer to use for `coerce_static`:
        // `coerce_static`関数を呼び出すために、整数を作成
        let lifetime_num = 9;

        // Coerce `NUM` to lifetime of `lifetime_num`:
        // `NUM`を`lifetime_num`のライフタイムへと圧縮
        let coerced_static = coerce_static(&lifetime_num);

        println!("coerced_static: {}", coerced_static);
    }

    println!("NUM: {} stays accessible!", NUM);
}
```

## Trait bound

As a trait bound, it means the type does not contain any non-static
references. Eg. the receiver can hold on to the type for as long as
they want and it will never become invalid until they drop it.

It's important to understand this means that any owned data always passes
a `'static` lifetime bound, but a reference to that owned data generally
does not:

```rust,editable,compile_fail
use std::fmt::Debug;

fn print_it( input: impl Debug + 'static ) {
    println!( "'static value passed in is: {:?}", input );
}

fn main() {
    // i is owned and contains no references, thus it's 'static:
    let i = 5;
    print_it(i);

    // oops, &i only has the lifetime defined by the scope of
    // main(), so it's not 'static:
    print_it(&i);
}
```
The compiler will tell you:
```ignore
error[E0597]: `i` does not live long enough
  --> src/lib.rs:15:15
   |
15 |     print_it(&i);
   |     ---------^^--
   |     |         |
   |     |         borrowed value does not live long enough
   |     argument requires that `i` is borrowed for `'static`
16 | }
   | - `i` dropped here while still borrowed
```

<!--
### See also:
-->
### 参照

<!--
[`'static` constants][static_const]
-->
[`'static` 定数][static_const]

[static_const]: ../../custom_types/constants.md
