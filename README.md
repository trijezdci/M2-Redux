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
CONST Space = 24B; (* octal representation *)
```
will be replaced by
```modula-2
CONST Space = CHR(20); (* decimal representation *)
```

### Suffix Literals (Optional)

M2-Redux **may** replace all occurences of base-16 literals with suffix `H` with their `0x` prefixed equivalents. This option requires the use of command line option `--suffix-literals`.

```modula-2
CONST Foo = 0DEADBEEFH; (* suffix representation *)
```
may be replaced by
```modula-2
CONST Foo = 0xDEADBEEF; (* prefix representation *)
```

### Unary Minus

### Cast Syntax

### Type Conversion Functions

### Array Type Definitions

### Local (Nested) Modules

### Access Mode for Imported Variables

### DIV and MOD with Negative Operands
