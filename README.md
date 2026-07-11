# M2-Redux
Syntax simplification utility for classic Modula-2 source code.

M2-Redux is a feature-reduction and syntax-simplification utility for classic Modula-2 source code. It performs lexical substitutions and syntax transformations on classic Modula-2 source files following the guidelines in the paper [On the Maintenance of Classic Modula-2 Compilers](https://arxiv.org/abs/1809.07080).

The conversion of Modula-2 source code with this utility allows downstream classic Modula-2 compilers to remove support for outdated, redundant and non-essential features altogether. The converted source code can still be compiled on such a feature reduced compiler with no or very little manual modification.

## License

M2-Redux is released under the [General Public License 2.0 (GPLv2)](https://www.gnu.org/licenses/old-licenses/lgpl-2.0-standalone.html).

## Conversions

### Operator Synonyms

M2-Redux will replace all occurences of synonym operators `&` and `~` with their keyword equivalents.

```modula-2
IF ~foo & bar THEN ...
```
will be replaced by
```modula-2
IF NOT foo AND bar THEN ...
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
and where the implementation module contains the body of the formerly local module.
```modula-2
IMPLEMENTATION MODULE Bar;
  << body of local module Bar >>
END Bar.
```

## Access Mode for Imported Variables

M2-Redux will replace all L-value occurences of imported variables into procedure calls to a setter procedure.

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
