---
title: "Expressions"
linkTitle: "Expressions"
weight: 60
description: >

---

The nomenclature in Blech is similar to that in C.
There are typed memory locations (objects), left hand side expressions (lvalues) and right hand side expressions (rvalues).
Blech does not allow to create function references and thus we have no designators.

## Operator precedence
The expression grammar below will not specify the precedence of operators.
This allows for a concise presentation.
Therefore this section specifies the precedence separately.

All operators are left associative in Blech.
The following table summarizes the operator precedence in Blech, from lowest precedence (least binding) to highest precedence (most binding). Operators in the same box have the same precedence. Unless the syntax is explicitly given, operators are binary.


Operator precedence low to high:

| Operators | Description
| --- | --- | 
| `:`, `as`, `as!` | Type annotation and type cast
| `or` | Boolean disjunction
| `and` | Boolean conjunction
| `<`, `>`, `<=`, `>=`, `==`, `!=` | Comparisons
| `|` | Bitwise OR 
| `^` | Bitwise XOR 
| `&` | Bitwise AND 
| `>>`, `<<`, `<>>`, `<<>`, `+>>` | Shifts, rotations and signed right shift
| `+`, `-` | Addition and subtraction
| `*`, `/`, `%` | Multiplication, division and remainder
| `not` x, ``-``x, ``~``x | Negation, unary minus, bit inversion


## Atoms

Atoms are the building blocks of expressions.
They are composed using operators.

```abnf
Atom ::= Identifier | Literal | "{}" | ArrayLiteral | StructLiteral | ParenthForm
```

Note that the term "atom" is not to be taken too literally because array and structure literals may contain arbitrary expressions themselves.

### Identifiers

An identifier occurring as an atom is a name. See section [Identifiers](../lexical/#identifiers) for a lexical definition.

### Literals

Blech supports various numeric literals:

```abnf
Literal ::= BoolLiteral | Integer | FloatNumber
```
Evaluation of a literal yields an object of the given type (bytes, natural number, signed integer, floating point number) with the given value.
The value may be approximated in the case of floating point literals. 

Numeric literals are untyped on their own.
This makes their use more flexible and permits writing more generic code.
Of course, before code generation every expression (and thus every literal) must be assigned a type which is the task of the type checker.
This section explains how to use literals and what type errors may happen.

Literals are assigned a type either explicitly using a _type annotation_ expression or implicitly when a unique type can be deduced from the context.

#### Annotated literals

For example, the literal `1` could represent a natural number, a signed integer or a byte word of some size. It could even be a floating point number.
It can be explicitly annotated using a type annotation.

```blech
1: nat16    // 2 bytes long natural number
1: int8     // 1 byte signed integer
1: bits32   // 4 bytes long word
1: float32  // single precision floating point number
```

The lexical analysis of `Literal` above suggests there are three kinds of literals.
In fact, we distinguish two kinds of integers.
Binary, octal and hexadecimal integer literals form the group of _bits literals_.
They may be interpreted as a byte word of some length or as a natural number of some length.

```blech
0xFF: nat32    // 4 byte natural number with value 255
0b101: bit8    // 1 byte word with value 5
```

<a name="bitsandints"></a>
They cannot be treated as a signed integer because depending on the length and the machine's representation of negative numbers they might yield a negative or positive value.

```blech
0xFF: int8    // error! Would be -1 if represented in two's complement
0xFF: int16   // error! Would be 255 in a two byte word
```

Also, bits literals cannot be interpreted as floating point numbers.
*For the exact specification of floating point numbers a hexadecimal float literal will be introduced.*

_Decimal integers_ may be interpreted as any kind of numeric type as long as the value fits the domain. The example above shows the straightforward use case with the literal `1`. Below some corner cases are exemplified:

```blech
-2: nat16     // error! Outside range of natural numbers
-200: int8    // error! Outside range [-128..127]
-2: bits8     // 1 byte word with value 254
-200: float32 // single precision floating point number
```

Float literals may only be interpreted as either single or double precision floating point numbers even if their fraction part is zero as in `10.0`.

In summary: annotations may only be used where they do not change the value of the expression and the value fits into the domain of the type.

#### Type deduction for literals

When a literal occurs as part of a binary expression where one side is already fully typed, the literal's type may be automatically deduced.

```txt {linenos=true}
var a: int32 = 9
let b = a + 17    
```

In line 1, the 9 is deduced to have type `int32` because the left hand side is already typed.
Note that a bits literal cannot be used to initialise an integer for reasons explained [above](#bitsandints).

In line 2, the literal `17` is deduced to be `int32`, then a signed 32-bit addition is performed and `b` is also deduced to be an `int32`.

### Complex literals

#### Reset literal

Empty braces `{}` have a special meaning.
They represent the default value for any struct or literal.
Using the reset literal in a declaration has the same effect as leaving out the initialisation:

```blech
var x: [8]float32 = {}
var y: [8]float32
```

Both `x` and `y` will be initialised with zeros in all cells.
The reset literal, as the name suggests, is most useful to set a modified data structure back to its default values:

```blech
var x: [8]float32 = {1, 2, 3, 4, 5, 6, 7, 8}
x = {}    // now x == {0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0}
```

The reset literal cannot be assigned to a structure that contains immutable fields (or, recursively, substructures with immutable fields).
The same is true for arrays with struct payloads.
See the section on [Structure types](../types/#structure-types) for more details.

#### Array literals

```abnf
ArrayLiteral ::= "{" CellInit ("," CellInit)* "}"
CellInit     ::= [IndexExpr "="] Expr
IndexExpr    ::= "[" Expr "]"
```

The array literal has to fit the shape of the array it is assigned to.
That is, the `Expr` in `IndexExpr` must evaluate to a non-negative number within array bounds.
It is however permitted to specify fewer values than the number given by the array dimensions.
The missing values are implicitly set to the data type's default value.
It is possible to set specific array cells by also specifying an index for a value.
Subsequent unindexed values will be set for the next array cells in order.

The `Expr` in `CellInit` must match the array's data type.

##### Example: Setting array values

```blech
var x: [2][3]nat8 = {{1, 2, 3},{4, 5, 6}} // all explicit
// x is
// 1  2  3
// 4  5  6

x[1] = {7, 8}                             // third value implicitly 0
// x is
// 1  2  3
// 7  8  0

x[1] = {[1] = 9, 10}                      // first value implicitly 0
// x is                                   // the index of `10` implicitly is 2
// 1  2  3
// 0  9  10

x = {{[1] = 11},{[0] = 12, [2] = 13}}
// x is
// 0  11  0
// 12  0  13

x = { {[2]=14} }                          // second row implicitly zeroed out
// x is
// 0  0  14
// 0  0  0
```

#### Struct literals

```abnf
StructLiteral ::= "{" FieldInit ("," FieldInit)* "}"
FieldInit     ::= Identifier "=" Expr
```

The identifiers must match the field names of the struct to be assigned.
The `Expr` must match the corresponding field's data type.
Immutable (`let` declared) fields may only be set in the initialisation of the structure.
Fields that are not specified in the literal are implicitly set to their data type's default value.

##### Example: Setting struct values

Assume the following declarations:

```blech
struct S
    var a: int8
    var b: int8
end

struct T
    let x: bool
    var y: S
end
```

The following code may be written (in some local scope):

```blech
var t: T = {x = true, y = {a = 1, b = 2}}
// t is
// t.x == true
// t.y.a == 1
// t.y.b == 2

t = {y = {a = 7}} // error! Cannot assign immutable field t.x

t.y = {a = 7}     // implicitly b = 0
// t is
// t.x == true
// t.y.a == 7
// t.y.b == 0
```

## Parenthesised form

A parenthesised form is an expression enclosed in parentheses:

```abnf
ParenthForm ::=  "(" Expr ")"
```

A parenthesised expression yields whatever that expression yields.

## Primaries

Primaries represent the most tightly bound operations of the language. Their syntax is:

```abnf
Primary ::=  Atom | Selection | Subscription | FunctionCallExpr
```

## Field selection

A field selection is a primary expression followed by a period and a name:

```abnf
Selection ::=  Primary "." Identifier
```

`Primary` must evaluate to a struct instance that contains a field with the name given by `Identifier`.

##  Subscriptions

A subscripting expression selects an item of an array:

```abnf
Subscription ::=  Primary "[" Expression "]"
```

The index expression must return a value that is non-negative and smaller than the array length.
Otherwise the program will crash in debug build mode and saturate to array index bounds in release mode.

{{< alert title="Important" color="warning">}}
The current implementation relies on C semantics and has no build modes. It will not necessarily crash, since C may read any addressable memory.
{{< /alert >}}

## Calls

A call calls a function with a possibly empty series of arguments:

```abnf
FunctionCallExpr ::= Identifier RhsArgList LhsArgList
```


## All computation expressions

All expressions above are concerned with retrieving a single value from some data structure. (With the exception of function calls).
Now all expressions are presented which take a value (or two) and produce a new value from it (those).
The following rule gives an overview of the remaining expression syntax.

```abnf
Expr ::=
    Primary                                            (highest precedence) 
    | "-" Expr | "~" Expr | "not" Expr
    | Expr "*" Expr | Expr "/" Expr | Expr "%" Expr
    | Expr "+" Expr | Expr "-" Expr
    | Expr ">>" Expr | Expr "<<" Expr | Expr "<>>" Expr | Expr "<<>" Expr | Expr "+>>" Expr
    | Expr "&" Expr
    | Expr "^" Expr
    | Expr "|" Expr
    | Expr "<" Expr | Expr ">" Expr | Expr "<=" Expr | Expr ">=" Expr | Expr "==" Expr | Expr "!=" Expr
    | Expr "and" Expr
    | Expr "or" Expr
    | Expr ":" Type | Expr "as" Type | Expr "as!" Type (lowest precedence)
```

Operator precedence has been discussed [above](#operator-precedence).

### Unary operations

The unary `-` (minus) operator yields the negation of its numeric argument.
If the argument is a literal without an annotation, it may not be a binary, octal or hexadecimal number because these are supposed to be some `bitX` type without a known length (yet) and the result of a minus cannot be defined.

The unary `~` (invert) operator yields the bitwise inversion of its `bitsX` argument.
It cannot be applied to a literal without a type annotation.

The unary `not` operator yields the opposite of its Boolean argument.

### Binary arithmetic operations

The binary arithmetic operations require that the arguments are of some numeric type.

The arithmetic operations work as expected on all arithmetic types.
See the sections on [arithmetic types](../types/#integer-types) for details regarding overflow handling.

The arguments' types may differ only in size.
When they differ, the smaller size value is lifted implicitly to the larger size.
The operation is then carried out on (possibly lifted) arguments of the same type.

#### Lifting types

```blech
var x: int8 = 7
var y: int16 = 300
var z = x + y
var u = x + 1
```

`x` and `y` are both signed integers but have different sizes.
In the context of the addition in line 3, the smaller type is lifted to the larger, effectively making `x` an `int16`.
Then, 16-bit signed addition is carried out producing an `int16` typed result.
This result is stored into `z` making it a `int16` variable, too.

In line 4, the literal `1` is deduced to be of type `int8` in the context of this expression (cf. paragraph on [type deduction](#type-deduction-for-literals) for literals).
Then, 8-bit signed addition is carried out producing an `int8` typed result.
This result is stored into `u` making it a `int8` variable, too.

The following snippet shows typical caveats.

```txt {linenos=true}
var x: int32 = 49 - 7    // error! Cannot determine type of '49' and '7'

var y: int8 = (49: int8) - 128 // error 128 does not fit into int8

var a: bits8 = 0x1
var b: nat8 = 2
var c = a + b            // error! Type mismatch
```

In line 1, the context of `49` is `7` and vice versa.
Both do not have a concrete type.
It is therefore not clear which implementation of `-` should be invoked.
The code only specifies that the result will be stored in an `int32` memory location but several different types (`int8`, `int16`, `int32`) would fit in there.

In line 3, the previous issue was resolved by specifying that `49` should be treated as an `int8`.
Thus the other operand must be an `int8` as well but the given literal is outside the `int8` domain.
(In this particular case, writing `(49: int8) + (-128)` would solve the problem due to the asymmetry of signed integers.) 

The last line shows an operation on different types (of the same size).
Addition for natural numbers must not overflow while addition for bits will wrap around.
It is not clear which one should be used here.
Either `a` or `b` need to be explicitly cast using the `as` operator to resolve this issue.


### Bitwise operations
{{< alert color="warning">}}
// TODO
{{< /alert >}}

### Comparison operations

Unlike C, in Blech equality and inequality have the same priority as the ordering operators.
Furthermore the precedence of comparison operators is lower than that of any arithmetic, shifting or bitwise operation.

Note that all operators are left associative.
Hence chaining comparisons is possible syntactically but makes little sense.

#### Comparison chaining

```blech
var b = 4 < 5 <= false // evaluates to true <= false which is false
var c = 4 < 5 <= 6 // type error: cannot compare true <= 6
```

This is different to C because in Blech Booleans and numbers are incomparable.

#### Comparison lifting

Comparison operators lift their arguments to a common sized type like arithmetic operators do (see above).
Additionally, comparisons permit using literals that would require a larger domain.
This allows writing (in)equalities without cluttering the code with trivial type casts.

##### Example: Comparison with literals outside the domain

```blech
var x: int8 = 7
var y = x < 1000
```

The literal `1000` is a (signed) integral number and would fit into an `int16`.
Therefore we allow to implicitly lift `x` to `int16` and carry out the comparison.
This makes sense because the result of all comparison operators is always a Boolean value.
There are no "surprises" about the outcome.

### Logical operators

The operators `and` and `or` may only be applied to Boolean typed arguments.

The expression `a and b` is true if and only if `a` is true and `b` is true.
The evaluation is performed lazily: first `a` is evaluated. If it is false, the expression returns `false` without evaluating `b`.

The expression `a or b` is true unless both `a` is false and `b` is false.
The evaluation is performed lazily: first `a` is evaluated. If it is true, the expression returns `true` without evaluating `b`.

### Representation annotation and change

Type annotations specify a concrete type for a literal which may represent values from different types.

Type casts _change_ the type of an already typed expression.
Casts are only permitted where the (machine) representation of a value will not change and casts assume that the value fits into the target type's domain.

#### Safe casts

```blech
var x: int16 = 100
let y = x as nat8    // ok, y == 100 of type nat8

x = x * 3
let z = x as nat8    // runtime error: 300 outside nat8 domain 0..255

x = -256             // assuming 2's complement on the machine:
                     // x == 0xb_1111_1111_0000_0000
let u = x as bits8   // runtime error
let v = x as bits16  // ok, v == 65280

let f: float32 = 1
x = f as int16       // error! impossible to cast floating point to integral types
```

The restrictions on the casts do not rule out runtime errors.
At the same time they prevent some manipulations that are possible in C.
For example, it is not possible to interpret a floating point as a bits type and change individual bits.

{{< alert color="warning">}}
// TODO is the description of `as` correct? Are there restrictions, bugs, features?
{{< /alert >}}
### Evaluation order

Undefined at the moment. Evaluated by the C compiler.

