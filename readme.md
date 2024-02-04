Following is not entirely written by me and I am lazy to edit it.

Ini streaming parser
====================

Compatible with `no_std`.

This library implements a pretty bare-bones, but super fast, streaming INI parser.

Examples
--------

```rust
use ini_core as ini;

let document = "\
[SECTION]
# this is a comment
Key=Value";

let elements = [
	ini::Item::Section("SECTION"),
	ini::Item::Comment("this is a comment"),
	ini::Item::Property("Key", "Value"),
];

for (line, item) in ini::Parser::new(document).enumerate() {
	assert_eq!(item, elements[line]);
}
```

The parser is very much line-based, it will continue no matter what and return nonsense as an item:

```rust
use ini_core as ini;

let document = "\
[SECTION
nonsense";

let elements = [
	ini::Item::Error("[SECTION"),
	ini::Item::Action("nonsense"),
];

for (line, item) in ini::Parser::new(document).enumerate() {
	assert_eq!(item, elements[line]);
}
```

Lines starting with `[` but contain either no closing `]` or a closing `]` not followed by a newline are returned as `Item::Error`.
Lines missing a `=` or starting with `'` or `"` are returned as `Item::Action`.

Format
------

This fork adds some thinking of its own to unspecified abstract INI format.

* Newline is either `"\r\n"`, `"\n"` or `"\r"`. It can be mixed in a single document but this is not recommended.
* Section header is `"[" section "]" newline`. `section` can be anything except contain newlines.
* Property is `key "=" value newline`. `key` and `value` can be anything except contain newlines.
* Comment is `"[ # | / | \ |]" comment newline` and Blank is just `newline`. The comment character can be customized.

Note that padding whitespace is not trimmed by default:
Section `[ SECTION ]`'s name is `<space>SECTION<space>`.
Property `KEY = VALUE` has key `KEY<space>` and value `<space>VALUE`.
Comment `; comment`'s comment is `<space>comment`.

No further processing of the input is done, eg. if escape sequences are necessary they must be processed by the user.

Performance
-----------

# Unchecked for now.

Tested against the other INI parsers available on crates.io. Fair warning all parsers except `light_ini` use a document model which allocate using `HashMap` and `String` which isn't exactly fair... In any case `ini_core` still mops the floor with `light_ini` which is 2a callback-based streaming parser.

These tests were run with `-C target-cpu=native` on an AMD Ryzen 5 3600X.

Parsing a big INI file (241 KB, 7640 lines):

```text
running 5 tests
test configparser ... bench:   1,921,162 ns/iter (+/- 207,479) = 128 MB/s
test ini_core     ... bench:      99,860 ns/iter (+/- 2,396) = 2472 MB/s
test light_ini    ... bench:     320,363 ns/iter (+/- 33,904) = 770 MB/s
test simpleini    ... bench:   7,814,605 ns/iter (+/- 271,016) = 31 MB/s
test tini         ... bench:   2,613,870 ns/iter (+/- 188,756) = 94 MB/s

test result: ok. 0 passed; 0 failed; 0 ignored; 5 measured; 0 filtered out; finished in 19.99s
```

Parsing a smaller INI file (17.2 KB, 800 lines):

```text
running 5 tests
test configparser ... bench:     266,602 ns/iter (+/- 21,394) = 66 MB/s
test ini_core     ... bench:       8,179 ns/iter (+/- 845) = 2159 MB/s
test light_ini    ... bench:     149,990 ns/iter (+/- 16,379) = 117 MB/s
test simpleini    ... bench:     563,204 ns/iter (+/- 55,080) = 31 MB/s
test tini         ... bench:     340,383 ns/iter (+/- 28,667) = 51 MB/s

test result: ok. 0 passed; 0 failed; 0 ignored; 5 measured; 0 filtered out; finished in 15.75s
```

License
-------

Licensed under [MIT License](https://opensource.org/licenses/MIT), see [license.txt](license.txt).

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, shall be licensed as above, without any additional terms or conditions.
