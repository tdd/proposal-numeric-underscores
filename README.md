# This proposal lives somewhere else now

It's been merged with the **accepted proposal led by Sam Goto**, which [you can find here](https://github.com/tc39/proposal-numeric-separator). Contribute there!

# Motivation

Large numeric literals are difficult for the human eye to parse quickly, especially when there are long digit repetitions.  This impairs both the ability to get the correct value / order of magnitude…

```js
1000000000   // Is this a billion? a hundred millions? Ten millions?
101475938.38 // what scale is this? what power of 10?
```

…but also fails to convey some use-case information, such as fixed-point arithmetic using integers.  For instance, financial computations often work in 4- to 6-digit fixed-point arithmetics, but even storing amounts as cents is not immediately obvious without separators in literals:

```js
const FEE = 12300
// is this 12,300? Or 123, because it's in cents?

const AMOUNT = 1234500
// is this 1,234,500? Or cents, hence 12,345? Or financial, 4-fixed 123.45?
```

Using underscores (`_`, U+005F) as separators helps improve readability for numeric literals, both integers and floating-point (and in JS, it's all floating-point anyway):

```ruby
1_000_000_000      # Ah, so a billion
101_475_938.38     # And this is hundreds of millions

FEE = 123_00       # $123 (12300 cents, apparently)
FEE = 12_300       # $12,300 (woah, that fee!)
AMOUNT = 12345_00  # 12,345 (1234500 cents, apparently)
AMOUNT = 123_4500  # 123.45 (4-fixed financial)
AMOUNT = 1_234_500 # 1,234,500
```

Also, this works on the fractional and exponent parts, too:

```ruby
0.000_001 # 1 millionth
1e10_000  # 1^10000 -- granted, far less useful / in-range…
```

This would make a very valuable addition to literal syntax.

# Prior art

Underscores are allowed anywhere in integer and floating-point literals in the following languages (at least):

- Ruby
- Python 3.6+
- Java 7+
- Rust
- Swift

So yes, C++, C# and Go don't have it (yet), but that still seems like JS is missing out here.

# Syntax changes

(Changes expressed over ES2016's spec text.)

The only section of lexical grammar that seems impacted would be numeric literals, specifically digit definitions:

## 11.8.3 Numeric Literals

```
DecimalDigit:: one of
  0 1 2 3 4 5 6 7 8 9 _

BinaryDigit:: one of
  0 1 _

OctalDigit:: one of
  0 1 2 3 4 5 6 7 _

HexDigit:: one of
  0 1 2 3 4 5 6 7 8 9 a b c d e f A B C D E F _
```

# Cross-cutting concerns

Most number-testing and number-conversion APIs and procedural steps are likely affected, usually automatically by transitivity of abstract operations used in their algorithms, in order to maintain consistency.

## *ToNumber*, *ToXx* and the `Number` constructor

The semantics described in *7.1.3.1. ToNumber Applied to the String Type* already rely on digit definitions from 11.8.3, so the algorithm there doesn't seem to need any adjustment.

Because the `Number` constructor (as described in 20.1.1.1), various abstract operations (such as *ToInteger*, *ToIntXx* and *ToUintXx*) and most other "standard lib" operations rely on *ToNumber* semantics, they also do not seem to need adjustment.

## Global object functions `isFinite` and `isNaN`

Both rely on *ToNumber* semantics (18.2.2 and 18.2.3), so they adjust automatically.

## Global object functions `parseInt` and `parseFloat`

`parseFloat` relies on *StrDecimalLiteral* syntax, which in turns relies on *DecimalDigit*, so we're covered.

`parseInt` uses conventional radix prefixes, which we don't need to change, then delegates to *ToInt32*, which as seen above relies on our adjusted syntax definitions already.

## Methods of the `Number` constructor

All detection methods (`isXx` methods) operate on actual numbers, such as number literals, so they do not introduce extra conversion / parsing rules.  The `parseFloat` and `parseInt` methods are just convenience aliases over their homonyms in the global object.
