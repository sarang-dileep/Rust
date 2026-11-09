# Hello World — Rust

My first Rust program, built with Cargo. It prints `Hello, world!` to the
terminal — the traditional first step for setting up the toolchain and getting
comfortable with the build/run workflow.

## Prerequisites

Rust installed via [rustup](https://rustup.rs). This project uses the 2024
edition, so it needs Rust 1.85 or newer.

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Verify the install:

```sh
rustc --version
cargo --version
```

## Run

```sh
cargo run
```

Expected output:

```text
Hello, world!
```

## Build

```sh
cargo build            # debug build -> target/debug/hello_world
cargo build --release  # optimized build -> target/release/hello_world
```

You can also compile the single file without Cargo:

```sh
rustc src/main.rs && ./main
```

This produces a binary named `main` in the current directory; run it with
`./main`. Don't pass that binary back to `rustc` — it expects a source file.

## Project layout

```text
hello_world/
├── .gitignore      # ignores /target
├── Cargo.toml      # package metadata and dependencies
├── Cargo.lock      # locked dependency versions
└── src/
    └── main.rs     # program entry point
```

## Source

```rust
fn main() {
    let s = "Hello, world!";
    println!("{}", s);
}
```

`main` is the program's entry point. `s` is a string literal, which has the
type `&str` — a borrowed view of text stored in the compiled binary. Because
it's only being printed, no heap allocation is needed. `println!("{}", s)`
writes it to stdout, with `{}` as the placeholder for `s`.

## Learnings: `String` vs `&str`

Both represent UTF-8 text — the difference is **ownership**.

| | `String` | `&str` |
|---|---|---|
| Ownership | Owns its bytes | Borrows someone else's bytes |
| Storage | Heap-allocated, growable | Pointer + length into existing bytes |
| Mutability | Can grow/shrink (`push_str`, `push`, ...) | Immutable view |
| Copying | Moves (or clones/allocates) | Cheap — just a pointer and a length |
| Where it comes from | `String::from(...)`, `.to_string()`, `format!` | Literals, `s.as_str()`, `&some_string` |

```rust
let owned: String = String::from("hello"); // owned, on the heap
let borrowed: &str = "hello";              // borrowed literal, no allocation
let view: &str = &owned;                   // &String -> &str (deref coercion)
let owned2: String = borrowed.to_string(); // &str -> String
let owned3: String = borrowed.to_owned();  // same as above
```

Key points:

- String literals like `"Hello, world!"` are `&'static str` — baked into the
  binary, no heap allocation.
- Bare `str` is unsized, so it's almost always used behind a reference: `&str`.
- `String` derefs to `str`, so `&String` automatically works where `&str` is
  expected.
- Prefer `&str` for read-only function parameters; use `String` when you need
  to own, build, or return new text.
- Strings can't be indexed by integer (`s[0]`) because UTF-8 characters are
  variable-width. Use `.chars()`, `.bytes()`, or a byte slice like `&s[0..2]`.

### Mistakes I hit

Passing a variable straight to `println!`:

```rust
println!(s); // error: format argument must be a string literal
```

The first argument must be a literal format string:

```rust
println!("{}", s);
```

Trying to assign a `&str` to a `String`:

```rust
let s = String::new();
s = "Hello, world!"; // error: expected `String`, found `&str`
```

`String::new()` fixes `s` as a `String`, but `"Hello, world!"` is a `&str`;
`s` also wasn't `mut`. Fixes:

```rust
let s = String::from("Hello, world!"); // owned String from a literal
// or
let s: &str = "Hello, world!";         // just borrow the literal
// or
let mut s = String::new();
s.push_str("Hello, world!");           // build the String in place
```

## What I'm practicing

- Creating a project with `cargo new`
- The `main` function and the `println!` macro
- Owning vs. borrowing text: `String` vs `&str`
- Constructing and mutating a `String`
- Building and running with Cargo
