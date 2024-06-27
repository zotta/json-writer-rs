# json-writer-rs

[![CI](https://github.com/zotta/json-writer-rs/actions/workflows/ci.yaml/badge.svg)](https://github.com/zotta/json-writer-rs/actions/workflows/ci.yaml)
[![license](https://img.shields.io/github/license/zotta/json-writer-rs?color=blue)](./LICENSE)
[![Current Crates.io Version](https://img.shields.io/crates/v/json-writer.svg)](https://crates.io/crates/json-writer)


 Simple and fast crate for writing JSON to a string without creating intermediate objects.

# Usage

Basic usage:
```rust
use json_writer::JSONObjectWriter;
let number: i32 = 42;
let mut object_str = String::new();
{
    let mut object_writer = JSONObjectWriter::new(&mut object_str);
    object_writer.value("number", number);
}
assert_eq!(&object_str, "{\"number\":42}");
```

Various examples:

```rust
use json_writer::{to_json_string, NULL, JSONObjectWriter, JSONArrayWriter };
// Values
assert_eq!(to_json_string("Hello World\n"), "\"Hello World\\n\"");
assert_eq!(to_json_string(3.141592653589793f64), "3.141592653589793");
assert_eq!(to_json_string(true), "true");
assert_eq!(to_json_string(false), "false");
assert_eq!(to_json_string(NULL), "null");

// Options of values
assert_eq!(to_json_string(Option::<u8>::Some(42)), "42");
assert_eq!(to_json_string(Option::<u8>::None), "null");

// Slices and vectors
let numbers: [u8; 4] = [1,2,3,4];
assert_eq!(to_json_string(&numbers[..]), "[1,2,3,4]");
let numbers_vec: Vec<u8> = vec!(1u8,2u8,3u8,4u8);
assert_eq!(to_json_string(&numbers_vec), "[1,2,3,4]");
let strings: [&str; 4] = ["a","b","c","d"];
assert_eq!(to_json_string(&strings[..]), "[\"a\",\"b\",\"c\",\"d\"]");

// Hash-maps:
let mut map = std::collections::HashMap::<String,String>::new();
map.insert("Hello".to_owned(), "World".to_owned());
assert_eq!(to_json_string(&map), "{\"Hello\":\"World\"}");

// Objects:
let mut object_str = String::new();
let mut object_writer = JSONObjectWriter::new(&mut object_str);

// Values
object_writer.value("number", 42i32);
object_writer.value("slice", &numbers[..]);

// Nested arrays
let mut nested_array = object_writer.array("array");
nested_array.value(42u32);
nested_array.value("?");
nested_array.end();

// Nested objects
let nested_object = object_writer.object("object");
nested_object.end();

object_writer.end();
assert_eq!(&object_str, "{\"number\":42,\"slice\":[1,2,3,4],\"array\":[42,\"?\"],\"object\":{}}");
```

## Writing large files

You can manually flush the buffer to a file in order to write large files without running out of memory.

Example:

```rust
use json_writer::JSONArrayWriter;
fn write_numbers(file: &mut std::fs::File) -> std::io::Result<()> {
    let mut buffer = String::new();
    let mut array = JSONArrayWriter::new(&mut buffer);
    for i in 1i32 ..= 1000000i32 {
        array.value(i);
        if array.buffer_len() > 2000 {
            // Manual flush
            array.output_buffered_data(file)?;
        }
    }
    array.end();
    std::io::Write::write_all(file, buffer.as_bytes())?;

    return Ok(());
}
```

# Limitations

Because there is no intermediate representations, all values must be written in the order they appear in the JSON output.
The Borrow checker ensures sub-objects are closed before anything else can be written after them.
```rust compile_fail
use json_writer::JSONObjectWriter;
let mut object_str = String::new();
let mut object_writer = JSONObjectWriter::new(&mut object_str);
let mut nested_a = object_writer.object("a");
let mut nested_b = object_writer.object("b");

// Compile error: The borrow checker ensures the values are appended in the correct order.
// You can only write one object at a time.
nested_a.value("id", "a");
nested_b.value("id", "b");
```

The writer does **not** check for duplicate keys

```rust
use json_writer::JSONObjectWriter;
let mut object_str = String::new();
{
    let mut object_writer = JSONObjectWriter::new(&mut object_str);
    object_writer.value("number", 42i32);
    object_writer.value("number", 43i32);
}
assert_eq!(&object_str, "{\"number\":42,\"number\":43}");
```

