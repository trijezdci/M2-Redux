## TO DO

### Bit Shifting

add transformation for bit shifting

(1) Logitech/MOCKA/ACK : `SYSTEM.SHIFT()` function

transform

```modula-2
foo := SYSTEM.SHIFT(val, shiftFactor);
```
as follows:

if `shiftFactor > 0`

```modula-2
foo := SYSTEM.SHL(val, shiftFactor);
```

if `shiftFactor < 0`

```modula-2
foo := SYSTEM.SHR(val, shiftFactor);
```

for non-constant `shiftFactor` replace with call to `BitShifts.ShiftInteger()` or `BitShifts.ShiftCardinal()`.

Casting may be necessary depending on type.


(2) TopSpeed/StonyBrook : pervasive `LSH()` and `RSH()` functions

transform

```modula-2
foo := LSH(val, shiftFactor);
bar := RSH(val, shiftFactor);
```
to
```modula-2
foo := SYSTEM.SHL(val, shiftFactor);
bar := SYSTEM.SHR(val, shiftFactor);
```

### Add to Libraries section:

#### BitShifts

```modula-2
DEFINITION MODULE BitShifts;
PROCEDURE ShiftInteger ( val, shiftFactor : INTEGER ) : INTEGER;
PROCEDURE ShiftLongInt ( val : LONGINT; shiftFactor : INTEGER ) : LONGINT;
PROCEDURE ShiftCardinal ( val : CARDINAL; shiftFactor : INTEGER ) : CARDINAL;
PROCEDURE ShiftLongCard ( val : LONGCARD; shiftFactor : INTEGER ) : LONGCARD;
END BitShifts.
```

+++
