# json-writer-rs

[![CI](https://github.com/zotta/json-writer-rs/actions/workflows/ci.yaml/badge.svg)](https://github.com/zotta/json-writer-rs/actions/workflows/ci.yaml)
[![license](https://img.shields.io/github/license/zotta/json-writer-rs?color=blue)](./LICENSE)
[![Current Crates.io Version](https://img.shields.io/crates/v/json-writer.svg)](https://crates.io/crates/json-writer)

Simple and fast JSON writer for Rust.

## No-std support

In no_std mode, almost all of the same API is available and works the same way.
To depend on json-writer in no_std mode, disable our default enabled "std" feature in
Cargo.toml.

```toml
[dependencies]
json-writer = { version = "0.3", default-features = false }
```
