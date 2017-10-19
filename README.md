# `cbindgen` &emsp; [![Build Status]][travis] [![Latest Version]][crates.io] [![Api Rustdoc]][rustdoc]

[Build Status]: https://api.travis-ci.org/eqrion/cbindgen.svg?branch=master
[travis]: https://travis-ci.org/eqrion/cbindgen
[Latest Version]: https://img.shields.io/crates/v/cbindgen.svg
[crates.io]: https://crates.io/crates/cbindgen
[Api Rustdoc]: https://img.shields.io/badge/api-rustdoc-blue.svg
[rustdoc]: https://eqrion.github.io/cbindgen/cbindgen

This project can be used to generate C bindings for Rust code. It is currently being developed to support creating bindings for [WebRender](https://github.com/servo/webrender/), but has been designed to support any project.

## Features

  * Builds bindings for a crate, its mods, its dependent crates, and their mods
  * Only the necessary types for exposed functions are given bindings
  * Can specify annotations for controlling some aspects of binding
  * Support for generic structs and unions
  * Support for exporting constants and statics
  * Customizable formatting, can be used in C or C++ projects
  * Support for generating `#ifdef`'s for `#[cfg]` attributes

## Use

### Command line

`cbindgen crate/ -o crate/bindings.h`

See `cbindgen --help` for more options.

### `build.rs`

`cbindgen` can also be used in build scripts. How this fits into compiling the native code depends on your project.

Here's an example build.rs script:
```rust
extern crate cbindgen;

use std::env;

fn main() {
    let crate_dir = env::var("CARGO_MANIFEST_DIR").unwrap();

    cbindgen::generate(&crate_dir)
      .unwrap()
      .write_to_file("bindings.h");
}

```

## Configuration

There are some options that can be used to configure the binding generation. They can be specified by creating a `cbindgen.toml` with the options in the binding crate root or at a path manually specified through the command line. Alternatively, build scripts can specify them using `cbindgen::generate_with_config`.

Here is a description of the options available in a config.

```toml
# An optional string of text to output at the beginning of the generated file
header = "/* Text to put at the beginning of the generated file. Probably a license. */"
# An optional string of text to output at the end of the generated file
trailer = "/* Text to put at the end of the generated file */"
# An optional name to use as an include guard
include_guard = "mozilla_wr_bindings_h"
# An optional string of text to output between major sections of the generated
# file as a warning against manual editing
autogen_warning = "/* Warning, this file is autogenerated by cbindgen. Don't modify this manually. */"
# Whether to include a comment with the version of cbindgen used to generate the
# file
include_version = true
# An optional namespace to output around the generated bindings
namespace = "ffi"
# An optional list of namespaces to output around the generated bindings
namespaces = ["mozilla", "wr"]
# The style to use for curly braces
braces = "[SameLine|NextLine]"
# The desired length of a line to use when formatting lines
line_length = 80
# The amount of spaces in a tab
tab_width = 2
# The language to output bindings in
language = "[C|C++]"

[parse]
# Whether to parse dependent crates and include their types in the generated
# bindings
parse_deps = true
# A white list of crate names that are allowed to be parsed
include = ["webrender", "webrender_traits"]
# A black list of crate names that are not allowed to be parsed
exclude = ["libc"]
# A list of crate names that should be run through `cargo expand` before
# parsing to expand any macros
expand = ["euclid"]

[fn]
# An optional prefix to put before every function declaration
prefix = "string"
# An optional postfix to put after any function declaration
postfix = "string"
# How to format function arguments
args = "[Auto|Vertical|Horizontal]"
# A rule to use to rename function argument names
rename_args = "[None|GeckoCase|LowerCase|UpperCase|PascalCase|CamelCase|SnakeCase|ScreamingSnakeCase|QualifiedScreamingSnakeCase]"

[struct]
# A rule to use to rename field names
rename_fields = "[None|GeckoCase|LowerCase|UpperCase|PascalCase|CamelCase|SnakeCase|ScreamingSnakeCase|QualifiedScreamingSnakeCase]"
# Whether to generate helper template specialization for generics
generic_template_specialization = true
# Whether to derive an operator== for all structs
derive_eq = false
# Whether to derive an operator!= for all structs
derive_neq = false
# Whether to derive an operator< for all structs
derive_lt = false
# Whether to derive an operator<= for all structs
derive_lte = false
# Whether to derive an operator> for all structs
derive_gt = false
# Whether to derive an operator>= for all structs
derive_gte = false

[enum]
# A rule to use to rename enum variants
rename_variants = "[None|GeckoCase|LowerCase|UpperCase|PascalCase|CamelCase|SnakeCase|ScreamingSnakeCase|QualifiedScreamingSnakeCase]"

```

## Examples

See `compile-tests/` for some examples of rust source that can be handled.

## Major differences between `cbindgen` and `rusty-cheddar`

1. `cbindgen` supports generics
2. `cbindgen` supports C++ output using `enum class` and `template specialization`
3. `cbindgen` supports generating bindings including multiple modules and crates

There may be other differences, but those are the ones that I know of. Please correct me if I misrepresented anything.

## How it works

1. All the structs, unions, enums, type aliases, constants, statics, and functions that are representable in C are gathered
2. A dependency graph is built using the extern "C" functions as roots
    * This removes unneeded types from the bindings and sorts the structs that depend on each other
3. Some code generation is done to specialize generics that are specified as type aliases
4. The items are printed in dependency order in C syntax

## Future work

1. Better support for types with fully specified names
2. Support for generating a FFI interface for a Struct+Impl
3. ...
