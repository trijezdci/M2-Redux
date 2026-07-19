# M2-Redux
Syntax simplification utility for classic Modula-2 source code.

M2-Redux is a feature-reduction and syntax-simplification utility for classic Modula-2 source code. It performs lexical substitutions and syntax transformations on classic Modula-2 source files following the guidelines in the paper [On the Maintenance of Classic Modula-2 Compilers](https://arxiv.org/abs/1809.07080).

The conversion of Modula-2 source code with this utility allows downstream classic Modula-2 compilers to remove support for outdated, redundant and non-essential features altogether. The converted source code can still be compiled on such a feature reduced compiler with no or very little manual modification.

## License

M2-Redux is released under the [General Public License 2.0 (GPLv2)](https://www.gnu.org/licenses/old-licenses/lgpl-2.0-standalone.html).

## Conversions

### Operator Synonyms

M2-Redux will replace all occurences of synonym operators `&` and `~` with their keyword equivalents and all occurences of `<>` with `#`.

```modula-2
IF ~foo & (bar <> baz) THEN ...
```
will be replaced by
```modula-2
IF NOT foo AND (bar # baz) THEN ...
```

### Octal Number Literals

M2-Redux will replace all occurences of octal literals with their decimal equivalents.

```modula-2
CONST Foo = 377B; (* octal representation *)
```
will be replaced by
```modula-2
CONST Foo = 255; (* decimal representation *)
```

### Character Code Literals

M2-Redux will replace all occurences of character code literals with their decimal equivalents wrapped in a function call to the built-in `CHR()` function.

```modula-2
CONST Space = 40C; (* octal representation *)
```
will be replaced by
```modula-2
CONST Space = CHR(32); (* decimal representation *)
```

### Suffix Literals (Optional)

M2-Redux **may** replace all occurences of `H` suffixed base-16 literals with their `0x` prefixed equivalents. This option requires the use of command line option `--suffix-literals`.

```modula-2
CONST Foo = 0DEADBEEFH; (* suffix representation *)
```
may be replaced by
```modula-2
CONST Foo = 0xDEADBEEF; (* prefix representation *)
```

### Unary Minus

M2-Redux will insert parentheses into multi-term expressions with unary minus for disambiguation. This requires the use of command line option `--ebnf-conform-unary-minus` or `--math-conform-unary-minus`.

```modula-2
i := -a + b;
```
will be replaced by
```modula-2
i := -(a + b); (* --ebnf-conform-unary-minus *)
```
respectively
```modula-2
i := (-a) + b; (* --math-conform-unary-minus *)
```
depending on the command line option passed.

### Cast Syntax

M2-Redux will replace all occurences of classic Modula-2 type cast syntax with a function call to the `SYSTEM.CAST()` function adopted from ISO Modula-2.

```modula-2
int := INTEGER(card); (* legacy syntax *)
```
will be replaced by
```modula-2
int := SYSTEM.CAST(INTEGER, card); (* ISO M2 syntax *)
```

### Type Conversion Functions

M2-Redux will replace calls to built-in type conversion functions `INT()`, `CARD()`, `TRUNC()`, `FLOAT()` and `LFLOAT()` with equivalent calls to built-in function `VAL()`.

```modula-2
int := INT(card);
card := CARD(int);
int := TRUNC(real);
real := FLOAT(int);
longReal := LFLOAT(longInt);
```
will be replaced by
```modula-2
int := VAL(INTEGER, card);
card := VAL(CARDINAL, int);
int := VAL(INTEGER, real);
real := VAL(REAL, int);
longReal := VAL(LONGREAL, longInt);
```

### Array Type Definitions

M2-Redux will replace all occurences of long form array type definitions with their short form equivalents.

```modula-2
TYPE A = ARRAY [0..9] OF ARRAY [0..9] OF B; (* long form *)
```
will be replaced by
```modula-2
TYPE A = ARRAY [0..9], [0..9] OF B; (* short form *)
```

### Local (Nested) Modules

M2-Redux will remove all occurences of local modules and create an equivalent standalone library.

```modula-2
IMPLEMENTATION MODULE Foo;
  ...
  MODULE Bar;
    ...
  END Bar;
  ...
END Foo.
```
will be replaced by
```modula-2
IMPLEMENTATION MODULE Foo;
  IMPORT Bar;
  ...
END Foo.
```
and a standalone library will be created with a definition module that marks the library private with a `PRIVATETO` pragma
```modula-2
DEFINITION MODULE Bar (*$PRIVATETO=Foo*);
  << interface for module Bar >>
END Bar.
```
where the implementation module contains the body of the formerly local module.
```modula-2
IMPLEMENTATION MODULE Bar;
  << body of local module Bar >>
END Bar.
```

## Access Mode for Imported Variables

M2-Redux will replace all L-value occurences of imported variables with procedure calls to a setter procedure.

```modula-2
FROM Foo IMPORT foo;
foo := expr;
```
will be replaced by
```modula-2
FROM Foo IMPORT foo, setFoo;
SetFoo(expr);
```

### DIV and MOD with Negative Operands

M2-Redux will replace all occurences of sub-expression terms that use `DIV` and `MOD` on operands of type `INTEGER` and `LONGINT` with equivalent library function calls to `tdiv()`, `tmod()`, `fdiv()` and `fmod()`. This requires the use of command line options `--pim2`, `--pim3` or `--pim4`.

```modula-2
quotient := i DIV j;
modulus := i MOD j;
```
will be replaced by
```modula-2
(* --pim2 and --pim3 *)
quotient := tdiv(i, j); (* truncated integer division *)
modulus := tmod(i, j);
```
respectively
```modula-2
(* --pim4 *)
quotient := fdiv(i, j); (* floored integer division *)
modulus := fmod(i, j);
```
depending on the command line option passed.

### Export Lists

M2-Redux will remove all occurences of directives `EXPORT` and `EXPORT QUALIFIED`.

### Function SIZE

M2-Redux will remove any import of function `SYSTEM.SIZE()` and replace any call to function `SYSTEM.SIZE()`, qualified or unqualified, that has an argument of a variant record type with an equivalent call to pervasive function `SIZE()`.

```modula-2
size := SYSTEM.SIZE(variantRecord);
```
will be replaced by
```modula-2
size := SIZE(variantRecord);
```

M2-Redux will further replace any call to function `SYSTEM.SIZE()`, qualified or unqualified, or to pervasive function `SIZE()` that has an argument that is not of a variant record type with an equivalent call to pervasive function `TSIZE()`.

```modula-2
TYPE FooType = << any non-variant type >>;
VAR fooVar : FooType;
size := SYSTEM.SIZE(fooVar);
```
will be replaced by
```modula-2
TYPE FooType = << any non-variant type >>;
VAR fooVar : FooType;
size := TSIZE(FooType);
```

### Function TSIZE

M2-Redux will remove any import of function `SYSTEM.TSIZE()` and replace any call to function `SYSTEM.TSIZE()`, qualified or unqualified, with an equivalent call to pervasive function `TSIZE()`.

```modula-2
size := SYSTEM.TSIZE(Type);
```
will be replaced by
```modula-2
size := TSIZE(Type);
```

### Functions MIN and MAX

M2-Redux will replace any call to pervasive function `MIN()` with an equivalent call to pervasive function `TMIN()` and any call to pervasive function `MAX()` with an equivalent call to pervasive function `TMAX()`.

```modula-2
min := MIN(Type);
max := MAX(Type);
```
will be replaced by
```modula-2
min := TMIN(Type);
max := TMAX(Type);
```

### Foreign Function Interfacing (FFI)

#### Module Header Normalisation

M2-Redux will replace non-standard foreign definition module headers with a standard definition module header that marks the interface with an `FFI` pragma.
```modula-2
FOREIGN DEFINITION MODULE Foobar; (* Logitech, GPM, MOCKA *)
```
and
```modula-2
DEFINITION MODULE FOR "C" Foobar; (* GNU *)
```
will be replaced by
```modula-2
DEFINITION MODULE Foobar (*$FFI="C"*);
```

#### Foreign Function Identifier Mapping

M2-Redux will further replace non-standard procedure headers with a foreign definition module that use or map to foreign identifiers with standard procedure headers that map to foreign identifiers via an `FFIDENT` pragma.
```modula-2
PROCEDURE foo_bar ( baz : Bam ); (* Logitech, MOCKA *)
```
and
```modula-2
PROCEDURE ["C"] / foo_bar ( baz : Bam ); (* TopSpeed *)
```
and
```modula-2
(*$FOREIGN="foo_bar"*) PROCEDURE FooBar ( baz : Bam ); (* ACK *)
```
will be replaced by
```modula-2
PROCEDURE FooBar ( baz : Bam ) (*$FFIDENT="foo_bar"*);
```

#### Hoisting Foreign Function Declarations from Mixed Modules

Where modules contain both standard and foreign function declarations, M2-Redux will remove all foreign function declarations and place them in a separate foreign function interface module.
```modula-2
DEFINITION MODULE Foobar; (* Standard definition module *)
PROCEDURE BarBaz ( bam : Boo ); (* standard procedure declaration *)
PROCEDURE ["C"] / baz_bam ( boo : Bee ); (* foreign procedure declaration *)
END Foobar.
```
will be replaced by
```modula-2
DEFINITION MODULE Foobar0 (*$FFI="C"; PRIVATETO=Foobar*);
PROCEDURE BazBam ( boo : Bee ) (*$FFIDENT="baz_bam"*);
END Foobar0.
```
and
```modula-2
DEFINITION MODULE Foobar;
PROCEDURE BarBaz ( bam : Boo );
TYPE BazBamFn = PROCEDURE ( Bee );
VAR BazBam : BazBamFn;
END Foobar.
```
and
```modula-2
IMPLEMENTATION MODULE Foobar;
IMPORT Foobar0;
...
BEGIN
  BazBam := Foobar0.BazBam;
END Foobar0.
```

### Bit Operations

M2-Redux will replace all occurences of sub-expression terms that use bitwise logical 16- and 32-bit operations `NOT`, `AND`, `OR` and `XOR` with equivalent 32-bit function calls to `SYSTEM.BWNOT()`, `SYSTEM.BWAND()`, `SYSTEM.BWOR()` and `SYSTEM.BWXOR()` respectively.

#### Input Syntax Forms

The following syntax forms are recognised in the input:

| Dialect | (1) bitwise not | (2) bitwise and | (3) bitwise or | (4) bitwise xor |
| :--- | :--- | :--- | :--- | :--- |
| **TopSpeed** | `NOT a` | `a AND b` | `a OR b` | `a XOR b` |
| **Stony Brook** | `BNOT a` | `a BAND b` | `a BOR b` | `a BXOR b` |
| **FST (16-bit)** | `BitOps.Not(a)` | `BitOps.And(a, b)` | `BitOps.Or(a, b)` | `BitOps.Xor(a, b)` |
| **FST (32-bit)** | `BitOps.NotLong(a)` | `BitOps.AndLong(a, b)`| `BitOps.OrLong(a, b)` | `BitOps.XorLong(a, b)`|
| **Logitech** | `BitBlockOps.Not(a)` | `BitBlockOps.And(a, b)` | `BitBlockOps.Or(a, b)` | `BitBlockOps.Xor(a, b)` |
| **ACK** | `SYSTEM.NOT(a)` | `SYSTEM.AND(a, b)` | `SYSTEM.OR(a, b)` | `SYSTEM.XOR(a, b)` |
| **MOCKA** | `SYSTEM.NOT(a)` | `SYSTEM.AND(a, b)` | `SYSTEM.OR(a, b)` | `SYSTEM.XOR(a, b)` |
| **GPM** | `SYSTEM.BITNOT(a)`| `SYSTEM.BITAND(a,b)`| `SYSTEM.BITOR(a,b)`| `SYSTEM.BITXOR(a,b)`|

where `a` and `b` are of types `WORD`, `CARDINAL`, `INTEGER`, `LONGINT` or `LONGCARD`.

#### Transformations

Types `CARDINAL` and `INTEGER` are always assumed to be of the same size as `WORD`. By default M2-Redux assumes a `WORD` size of 32 bits and a `LONGCARD` and `LONGINT` size of 64 bits. If the source code was written for targets with a `WORD` size of 16 bits, the use of command line option `--assume-16bit-wordsize` is required, except for source code that is written for FST or GPM.

The following transformations are carried out in the output:

|  Operation  | 32-bit (default) | 16-bit (`--assume-16bit-wordsize`) |
| :--- | :--- | :--- |
| **(1) bitwise not** | `SYSTEM.BWNOT(a)`    | `SYSTEM.BWAND(SYSTEM.BWNOT(a), 0FFFFH)` |
| **(2) bitwise and** | `SYSTEM.BWAND(a, b)` | `SYSTEM.BWAND(a, b)` |
| **(3) bitwise or**  | `SYSTEM.BWOR(a, b)`  | `SYSTEM.BWAND(SYSTEM.BWOR(a, b), 0FFFFH)` |
| **(4) bitwise xor** | `SYSTEM.BWXOR(a, b)` | `SYSTEM.BWAND(SYSTEM.BWXOR(a, b), 0FFFFH)` |


## Libraries

M2-Redux provides libraries with replacement functions for the non-portable `DIV` and `MOD` operations. The libraries are released under the [Library General Public License 2.0 (LGPLv2)](https://www.gnu.org/licenses/old-licenses/lgpl-2.0-standalone.html).

### IntMath

```modula-2
DEFINITION MODULE IntMath;
(* truncated integer division *)
PROCEDURE tdiv ( i, j : INTEGER ) : INTEGER;
(* modulus of truncated integer division *)
PROCEDURE tmod ( i, j : INTEGER ) : INTEGER;
(* floored integer division *)
PROCEDURE fdiv ( i, j : INTEGER ) : INTEGER;
(* modulus of floored integer division *)
PROCEDURE fmod ( i, j : INTEGER ) : INTEGER;
END IntMath.
```

### LongIntMath

```modula-2
DEFINITION MODULE LongIntMath;
(* truncated integer division *)
PROCEDURE tdiv ( i, j : LONGINT ) : LONGINT;
(* modulus of truncated integer division *)
PROCEDURE tmod ( i, j : LONGINT ) : LONGINT;
(* floored integer division *)
PROCEDURE fdiv ( i, j : LONGINT ) : LONGINT;
(* modulus of floored integer division *)
PROCEDURE fmod ( i, j : LONGINT ) : LONGINT;
END LongIntMath.
```

+++
