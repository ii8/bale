
# Bale

A minimal schema language and binary encoding for serializing data structures.

## An example schema
```
; A 24 bit color value
let color be
  tuple
    r: u8
    g: u8
    b: u8
  end

; Tuples are sequences of multiple data items
tuple
  ; We can have optional items with `maybe`
  position: maybe tuple
    ; `uv`s are compact variable size numbers
    x: uv
    y: uv
  end

  width: u32

  ; A choice of one of multiple data items
  union
    pixels: array color
    image: string
  end
end
```

See the [specification](/spec.md) for more detail.

## Some examples of encodings

| Schema      | Value    | Encoded   |
|-------------|----------|-----------|
| `void`      |          |           |
| `bool`      | false    | `00`      |
| `bool`      | true     | `01`      |
| `maybe u16` | just 12  | `01 0c`   |
| `maybe i8`  | nothing  | `00`      |
| `2 u8`      | 3, 4     | `03 04`   |
| `uv`        | 2200     | `f8 a8`   |

## Implementations

- [Pony](https://github.com/ii8/pony-bale)
- [Lua](https://github.com/ii8/lbale)

### Tooling

The [baler](https://github.com/ii8/baler) program can validate schemas and
decode/encode data to and from a diagnostic notation.

## Differences from XDR

- No four byte alignment
- Use of variable size integers instead of a fixed 32 bits for length encoding.
- Let bindings can have parameters unlike XDR typedefs
- Nicer schema language syntax (in my opinion)
- Fewer predefined types

## Code generation

Code generation is not required and encoding/decoding bale is trivial, this
means you are not limited to a subset of languages.

But generating code is still possible if you really want to.

## Correspondences

| Bale          | Regular Expressions        | Type Theory        |
|---------------|----------------------------|--------------------|
| tuples        | concatenation              | product types      |
| unions        | alternation                | sum types          |
| arrays        | the Kleene star            |                    |
| `union end`   | ∅                          | the **empty** type |
| `tuple end`   | ε                          | the **unit** type  |
| `bool`        |                            | 1 + 1              |
| decoding      | non-deterministic matching |                    |

