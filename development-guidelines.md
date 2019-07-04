[![Embark logo](media/embark-logo-bg.jpg)](http://embark.games)

# Embark Rust development guidelines

Guidelines and recommendations we use at [Embark](http://embark.games) for developing our [Rust 🦀](https://www.rust-lang.org/) projects.

## Guidelines

This is not meant to be an exhaustive collection and resource of all best practices in Rust, that is best covered by the [Rust Book](https://doc.rust-lang.org/book/) and other references. Instead this is list of specific guidelines and recommendations that we've run into and want to highlight and standardize on within Embark.

---

### 001 - Run rustfmt on save

__Motivation:__ We want to keep a consistent code base where we don't have to argue about style, so we use rustfmt on everything and verify on CI that it has been run. So highly recommend everyone to configure their editor to run rustfmt on save as it is more error prone to run it before commit.

With VS Code this should be automatic for our repos, see [section](#visual-studio-code).

---

### 002 - Prefer synchronization primitives from [`parking_lot`](https://docs.rs/crate/parking_lot/0.7.1) instead of `std::sync`

__Motivation:__ The `parking_lot` implementations are smaller, faster, and avoids poisioning errors which makes them more ergonomic and easier to use.

Example old:

```rust
use std::sync::{Mutex,RwLock};
let mutex = Mutex::new(1u32);
*mutex.lock().unwrap() += 5;
```

Example new:

```rust
use parking_lot::{Mutex,RwLock};
let mutex = Mutex::new(1u32);
*mutex.lock() += 2;
```

__Todo:__ Would be great to automatically verify this through something like a custom clippy lint, or even better: have the `parking_lot` primitives eventually find their way into std.

---

### 003 - Opt-in for Clippy and Rust 2018 style warnings for every crate

__Motivation:__ Keeps our code high quality and catches common errors and inefficiencies. We already verify on CI that clippy finds no warnings, and by opting in to warnings in every crate it also enables tools like RLS in VS Code to know that Clippy warnings

Add this to the top of the main file (`lib.rs` or `bin.rs`) for all of our crates:

```rust
#![warn(clippy::all)]
#![warn(rust_2018_idioms)]
```

__Todo:__ Would be great if we could configure this globally for our workspace instead of for each crate (more error prone). [#22](https://github.com/EmbarkStudios/rust-ecosystem/issues/22).

---

## Development environments

### Visual Studio Code

We primarily use [VS Code](https://code.visualstudio.com/) as development environment for Rust.

Extensions:

* Main:
  * [Rust (rls)](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust)
  * [Crates](https://marketplace.visualstudio.com/items?itemName=serayuzgur.crates) - easy Cargo.toml dependency version management
  * [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) - for debugging on Windows
  * [Native Debug](https://marketplace.visualstudio.com/items?itemName=webfreak.debug) - for debugging on Linux & Mac using GDB/LLDB
* Other:
  * GitHub
  * GitHub Pull Requests
  * GitLens
  * Better TOML
  * markdownlint
  * Markdown Preview Github Styling
  * [Shader languages support for VS Code](https://marketplace.visualstudio.com/items?itemName=slevesque.shader)
  * [TODO Highlight](https://marketplace.visualstudio.com/items?itemName=wayou.vscode-todo-highlight)

With VS Code this should be automatic for our repos, see  as long as one has the Rust extension installed and our repo contains a `.vscode/settings.json` with:

As we want to [run rustfmt on save](#001---run-rustfmt-on-save), all of our repos should have a `.vscode/settings.json` with:

```json
{
    "editor.formatOnSave": true,
}
```
