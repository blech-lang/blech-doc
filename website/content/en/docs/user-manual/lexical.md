---
title: "Lexical elements"
linkTitle: "Lexical elements"
weight: 20
description: >

---


## Comments
Single line comments are C++ style. They begin with `//` and end with the end of line.

Multi-line comments are C style. They begin with `/*` and end with `*/`. 
Comment nesting is supported.

### Declaration Comments
A special comment syntax allows to write comments for declarations (functions, activities, types, variables, ...) which will be transported over to the generated C code.
This allows to document the API and make this documentation available to the C programmer who integrates generated code in a larger scope.

There are two supported styles of declaration comments.
Java style multi-line comments `/** */` and .Net style single line comments `///`.

Note that using declaration comments before something that is not a declaration, e.g. an assignment, is a syntax error.

## Built-in Operators and Separators

| Operators and Separators | |
| --- | --- |
| Logical operators | `not` `and` `or`
| Arithmetic operators | `+` `-` `*` `/` `%` 
| Bitwise operators | `~` `&` `|` `^` `>>` `<<`
| Relational operators | `==` `!=` `<` `>` `<=` `>=`
| Type conversion | `as` `as!`
| Separators | `(` `)` `[` `]` `{` `}` `.` `,` `;` `:` 


## Keywords

`abort`
`activity`
`and`
`as` 
`await`
`bits8`
`bits16` 
`bits32`
`bits64`
`blech`
`bool`
`cobegin`
`const`
`do`
`else`
`elseif`
`end`
`exposes`
`extern`
`false`
`float32`
`float64`
`function`
`if`
`import`
`int8`
`int16`
`int32`
`int64`
`internal`
`let`
`module`
`nat8`
`nat16`
`nat32`
`nat64`
`not`
`or`
`param`
`prev`
`repeat`
`run`
`reset`
`return`
`returns`
`signature`
`singleton`
`struct`
`then`
`true`
`type`
`until`
`var`
`weak`
`when`
`while`
`with`

Additionally, the current compiler implementation reserves keywords for concepts that were designed into the language but not yet implemented. Reserved keywords include:
`assert`
`assume`
`default`
`emit`
`enum`
`error`
`extension`
`in`
`next`
`of`
`past`
`print`
`ref`
`shares`
`signal`
`suspend`
`throw`
`throws`
`try`
`typealias`
`unit`

## Identifiers
An identifier is any token that is not a keyword and starts with a letter or underscore and continues with an arbitrary number of letters, digits or underscores.
The precise definition is given by the following grammar rule

```abnf
Identifier ::=  "_"* ("a"..."z" | "A"..."Z")+ ("_" | "a"..."z" | "A"..."Z" | "0"..."9")*
```
Note that identifiers have an infix of at least one letter.

## Wildcard
Additionally we reserve a token that consists of underscores only.
We call this the "wildcard". Wildcards are useful when you want to discard the result of a computation without declaring a dummy variable.

```blech
_ = f()
```
In this example you cannot just call `f` like in C because the Blech compiler will complain that `f` is declared to return a value but there is no location to store this value in. The wildcard makes the intention to discard the returned value *explicit*.

## Literals

```abnf
BoolLiteral ::= "true" | "false"

Digit        ::= "0"..."9"
NonZeroDigit ::= "1"..."9"

BinInteger ::= "0b" ( ["_"] ( "0" | "1" ) )+
OctInteger ::= "0o" ( ["_"] "0"..."7" )+
HexInteger ::= "0x" ( ["_"] (Digit | "a"..."f" | "A"..."F") )+  
DecInteger ::= NonZeroDigit (["_"] Digit)* | "0" (["_"] "0")*

Integer ::= DecInteger | BinInteger | OctInteger | HexInteger

FloatNumber   ::=  PointFloat | ExponentFloat
PointFloat    ::=  [DigitPart] Fraction | DigitPart "."
DigitPart     ::=  Digit (["_"] Digit)*
Fraction      ::=  "." DigitPart
ExponentFloat ::=  (DigitPart | PointFloat) exponent
Exponent      ::=  ("e" | "E") ["+" | "-"] DigitPart
```