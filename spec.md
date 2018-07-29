
# Bale schema and encoding specification

A bale schema consists of zero or more **bindings** followed by a **type**.
Every type has an accompanying encoding, the type at the end of a
schema specifies the encoding of the schema as a whole.

All multibyte values are serialized in network byte order.

## Base types

The following base types are available:

### `u8`, `u16`, `u32`, `u64`
Unsigned integers.

### `i8`, `i16`, `i32`, `i64`
Two's complement integers.

### `f32`, `f64`
Single and double precision floating point numbers.

### `uv`
Variable size unsigned integer.

Encoded the same as [sqlite varints](https://sqlite.org/src4/doc/trunk/www/varint.wiki).

## Complex types

There are three constructs that can be used to form compound types
from existing ones.

### `tuple t0 t1... tn end`
A tuple type is written as a sequence of zero or more types enclosed within the
words `tuple` and `end`.
This type is encoded as the sequence of types `t0`... `tn` in order.

### `union t0 t1... tn end`
Written the same as a `tuple` except the keyword `union` is used at the start
instead.
Encoded as a `uv` followed by one of `t0`... `tn`.
The value of the preceding `uv` is an index(0-based) into `t0`... `tn` denoting
which of these types follows.

### `array t`
An array type is written as the word `array` followed by a single type.
Encoded as a `uv` denoting the number of `t`s that follow.

## Bindings
Types can be bound to names to facilitate reuse.

### `let name a0 a1... an be t`
An abstraction that binds `name` to a new type that when given
arguments for its parameters `a0` to `an`, yields `t` with all
occurrences of `a0` to `an` substituted by the given arguments.

Any type occurring after this binding in a schema may use this name.

## Comments and lexical conventions
A schema consists of a series of **words** which are space delimited sequences
of these characters:
`abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890._-<>?!`

**Words** are case sensitive.

All **words** following a `;` character are ignored until a newline.

**Words** followed directly by a `:` are also ignored, but only when
preceding a type within a `tuple` or `union`; this restriction may be
relaxed in the future.

## The Prelude

The following bindings are introduced implicitly at the beginning of a schema.

```
let none be
  union end

let void be
  tuple end

let bool be
  union
    false: void
    true: void
  end

let maybe x be
  union
    nothing: void
    just: x
  end

let map k v be
  array
    tuple
      key: k
      value: v
    end

let string be
  array u8

let utf8 be
  string

let 0 x be
  tuple end

let 1 x be
  tuple x end

let 2 x be
  tuple x x end

let 3 x be
  tuple x x x end

; These bindings of numbers to tuples continue in the obvious
; manner, however due to physical constraints the prelude
; is not reproduced here in it's entirety.
```

