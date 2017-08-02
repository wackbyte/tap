# tap

[![Documentation][docs-shield]][docs]
[![crates.io version][crate-shield]][crate]
[![Language (Rust)][rust-shield]][rust]

A simple crate exposing tapping functionality for all types, and extended
functionality for `Option`, `Result` & `Future`. Often useful for logging.

The tap operation takes, and then returns, full ownership of the variable being
tapped. This means that the closure may have mutable access to the variable,
even if the variable is otherwise immutable. Mutating closures require the
`|mut name|` pattern, and so can be easily distinguished from non-mutating taps.

## Features

* `future` - Exposes the `TapFutureOps` trait, providing `tap_ready`,
  `tap_not_ready` & `tap_err` (requires the *futures* crate).

  Futures do not provide mutable access for tap closures.

## Example

```rust
extern crate tap;

use tap::{TapOps, TapResultOps, TapOptionOps};

#[test]
fn filter_map() {
    let values: &[Result<i32, &str>] = &[Ok(3), Err("foo"), Err("bar"), Ok(8)];

    let _ = values.iter().filter_map(|result| {
        // It is especially useful in filter maps, allowing error information to
        // be logged/printed before the information is discarded.
        result.tap_err(|error| println!("Invalid entry: {}", error)).ok()
    });
}

#[test]
fn basic() {
    let mut foo = 5;

    // The `tap` extension can be used on all types
    if 10.tap(|v| foo += *v) > 0 {
        assert_eq!(foo, 15);
    }

    // Results have `tap_err` & `tap_ok` available.
    let _: Result<i32, i32> = Err(5).tap_err(|e| foo = *e);
    assert_eq!(foo, 5);

    // Options have `tap_some` & `tap_none` available.
    let _: Option<i32> = None.tap_none(|| foo = 10);
    assert_eq!(foo, 10);
}

#[test]
fn mutable() {
    let base = [1, 2, 3];
    let mutated = base.tap(|mut arr| for elt in arr.iter_mut() {
        *elt *= 2;
    });
    assert_eq!(mutated, [2, 4, 6]);
    //  base was consumed, and then returned into mutated, and is out of scope.
}

```

<!-- Links -->
[crate-shield]: https://img.shields.io/crates/v/tap.svg?style=flat-square
[crate]: https://crates.io/crates/tap
[rust-shield]: https://img.shields.io/badge/powered%20by-rust-blue.svg?style=flat-square
[rust]: https://www.rust-lang.org
[docs-shield]: https://img.shields.io/badge/docs-latest-green.svg?style=flat-square
[docs]: https://docs.rs/tap
