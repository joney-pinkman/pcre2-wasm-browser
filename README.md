# pcre2-wasm

PCRE2 (Perl Compatible Regular Expressions) compiled to WebAssembly

## Installation

```bash
npm install @stephen-riley/pcre2-wasm
```

## Usage

Internally this module uses the [PCRE2](https://pcre.org/) library, running
in a WebAssembly instance. This has a side effect of requiring you do
a few unusual things when using this module:

### Initialization

Before calling any constructors or methods, you must first asynchronously initialize the module by calling `init`.

```javascript
import PCRE from '@stephen-riley/pcre2-wasm'

async function main () {
  await PCRE.init()
  // make other PCRE calls...
}

main()
```

### Memory Management

When you create a new `PCRE` instance, you are allocating memory within the
WebAssembly instance. Currently, there are no hooks in JavaScript that
let us automatically free this memory when the `PCRE` instance is garbage
collected by the JavaScript runtime. This means that in order to prevent
memory leaks, you must call `.destroy()` on a `PCRE` instance when it
is no longer needed.

## API

```javascript
import PCRE from '@stephen-riley/pcre2-wasm'
```

### `PCRE.init()`

Initializes the module, returning a Promise that is resolved once
initialization is complete. You must call this at least once and await the
returned Promise before calling any other `PCRE` methods or constructors.

### `PCRE.version()`

Returns a string with the PCRE2 version information.

### `new PCRE(pattern, flags)`

Creates a new PCRE instance, using `pcre2_compile()` to compile `pattern`,
using `flags` as the compile options. You must call `.destroy()` on the
returned instance when it is no longer needed to prevent memory leakage.

- `pattern`: A string containing a Perl compatible regular expression.
  Tip: use `String.raw` to avoid needing to escape backslashes.
- `flags`: An optional string with each character representing an option.
  Supported flags are `i`, `m`, `s`, and `x`. See
  [perlre](http://perldoc.perl.org/perlre.html) for details.

```javascript
const pattern = String.raw`\b hello \s* world \b`
const re = new PCRE(pattern, 'ix')

// ...

re.destroy()
```

In the event of a compilation error in the pattern or an unsupported flag, an `Error` will be thrown with an error message from PCRE2. Additionally, it will have an `offset` property indicating the character offset in `pattern` where the error was encountered.

```javascript
let re

try {
  re = new PCRE(String.raw`a)b`)
}
catch (err) {
  console.error(`Compilation failed: ${err.message} at ${err.offset}.`)
  // Prints: Compilation failed: unmatched closing parenthesis at 1.
}
```

### `re.destroy()`

Releases the memory allocated in the WebAssembly instance. You must call this method manually once you no longer have a need for the instance, or else your program will leak memory.

### `re.match(subject, startOffset?)`

Match the `subject` against the regular expression, starting at `startOffset` if specified (otherwise, start at position 0).

#### match output

Returns an object that lists numbered and named captures, each with a `start`, `end`, and `match` field.

For example, matching `000123` against `/^(?<leading_zeros>0+).*$/` would result in the following object:

```javascript
{
  length: 1,
  0: { start: 0, end: 6, match: "000123" },
  1: { start: 0, end: 4, match: "000" },
  leading_zeros: { start: 0, end: 4, match: "000" }
}
```

On no match, returns `null`.

On an error, `throw`s an `Error` object whose string is a PCRE2 error name from `PCRE2.h`.

### `re.matchAll(subject, startOffset?)`

Returns an array of `re.match()` results.

On no match, returns an empty array `[]`.

On an error, `throw`s an `Error` object whose string is a PCRE2 error name from `PCRE2.h`.

### `re.substitute(subject, replacement, startOffset?)`

Performs a single substitution on `subject` against the regular expression, using `replacement`, starting at `startOffset` if specified (otherwise 0).

Returns a string.

On an error, `throw`s an `Error` object whose string is a PCRE2 error name from `PCRE2.h`.

### `re.substituteAll(subject, replacement, startOffset?)`

Performs all substitutions on `subject` against the regular expression, using `replacement`, starting at `startOffset` if specified (otherwise 0).

Returns a string.

On an error, `throw`s an `Error` object whose string is a PCRE2 error name from `PCRE2.h`.

### `re.exec(subject, global?)`

If `global` is specified and is truthy, will execute `matchAll()`; otherwise executes `match()`.

## Contributing

Prerequisites for development include Docker, `make`, and `curl`.  All emscripten compiles (via `emcc`) are done in docker containers to control the build environment.

## Credits

This is a fork of [desertnet/pcre](https://github.com/desertnet/pcre), which provided the emscripten framework and initial API exposure of PCRE2.  Many thanks!
