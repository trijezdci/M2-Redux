# M2-Redux
Syntax simplification utility for classic Modula-2 source code.

M2-Redux is a feature-reduction and syntax-simplification utility for classic Modula-2 source code. It performs lexical substitutions and syntax transformations on classic Modula-2 source files following the guidelines in the paper [On the Maintenance of Classic Modula-2 Compilers](https://arxiv.org/abs/1809.07080).

The conversion of Modula-2 source code with this utility allows downstream classic Modula-2 compilers to remove support for outdated, redundant and non-essential features altogether. The converted source code can still be compiled on such a feature reduced compiler with no or very little manual modification.

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
CONST Space = 24C; (* octal representation *)
```
will be replaced by
```modula-2
CONST Space = CHR(20); (* decimal representation *)
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
i := -(a) + b; (* --math-conform-unary-minus *)
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

### Local (Nested) Modules

### Access Mode for Imported Variables

### DIV and MOD with Negative Operands
