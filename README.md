# DocTest

**DocTest** is an experimental tool
for testing Swift example code in documentation.

**This is still a work-in-progress, and not yet ready for production**

> DocTest is inspired by 
> [Python's `doctest`](https://docs.python.org/3/library/doctest.html).

* * *

The hardest part of software documentation is writing it in the first place.
But once you clear that initial hurdle,
the challenge becomes keeping documentation correct and up-to-date.

There's no built-in feedback mechanism for documentation like there is for code.
If you write invalid code,
your compiler will tell you.
If you write valid but incorrect code,
your test suite will tell you.
But if you write documentation with invalid or incorrect example code,
nothing breaks (at least not immediately).

DocTest offers a way to annotate Swift code examples in documentation
with expectations about its behavior that can be tested automatically,
just like unit tests.

## Usage

```
OVERVIEW: A utility for syntax testing documentation in Swift code.

USAGE: swift-doc-test <input> [--swift-launch-path <swift-launch-path>] [--package] [--assumed-filename <assumed-filename>]

ARGUMENTS:
  <input>                 Swift code or a path to a Swift file 

OPTIONS:
  --swift-launch-path <swift-launch-path>
                          The path to the swift executable. (default:
                          /usr/bin/swift)
  -p, --package           Whether to run the REPL through Swift Package Manager
                          (`swift run --repl`). 
  --assumed-filename <assumed-filename>
                          The assumed filename to use for reporting when
                          parsing from standard input. (default: Untitled.swift)
  -h, --help              Show help information.

```

## How It Works

DocTest launches and interacts with
the Swift <abbr title="Read-Eval-Print-Loop">REPL</abbr>,
passing each code statement through standard input
and reading its result through standard output and/or standard error.

Consider the following function declaration
within a Swift package:

~~~swift
/**
    Returns the sum of two integers.

    ```swift
    add(1 1) // 3.0
    ```
*/
func add(_ a: Int, _ b: Int) -> Int { ... }
~~~

There are three problems with the example code
provided in the documentation for `add(_:_:)`:

1. It doesn't compile
   (missing comma separator between arguments)
2. It suggests an incorrect result
   (one plus one equals two, not three)
3. It suggests an incorrect type of result
   (the sum of two integers is an integer,
   which isn't expressible by a floating-point literal)

We can use DocTest to identify these problems automatically
by adding `"doctest"` to the start of the fenced code block.
This tells the documentation test runner to evaluate the code sample.

~~~swift
/**
    Returns the sum of two integers.

    ```swift doctest
    add(1 1) // Double = 3.0
    ```
*/
func add(_ a: Int, _ b: Int) -> Int { ... }
~~~

Run the `swift-doctest` command
from the root directory of the Swift package,
specifying the `--package` flag
(to invoke the Swift REPL via the Swift Package Manager)
and passing the path to the file containing the `add(_:_:)` function.
This will scan for all of code blocks annotated with
<code>```swift doctest</code>
run them through the Swift REPL,
and test the output with any annotated expectations.

```terminal
$ swift doctest --package path/to/file.swift
TAP version 13
1..1
not ok 1 - `add(1 1)` did not produce `Double 3.0`
  ---
  column: 1
  file: path/to/file.swift.md
  line: 1
  ...
  
```

> Test results are reported in [TAP format](https://testanything.org).

## License

MIT

## Contact

Mattt ([@mattt](https://twitter.com/mattt))
