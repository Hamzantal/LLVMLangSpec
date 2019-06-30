Constants
---------

LLVM has several different basic types of constants. This section
describes them all and their syntax.

### Simple Constants

**Boolean constants**

:   The two strings \'`true`\' and \'`false`\' are both valid constants
    of the `i1` type.

**Integer constants**

:   Standard integers (such as \'4\') are constants of the
    `integer <t_integer>`{.interpreted-text role="ref"} type. Negative
    numbers may be used with integer types.

**Floating-point constants**

:   Floating-point constants use standard decimal notation (e.g.
    123.421), exponential notation (e.g. 1.23421e+2), or a more precise
    hexadecimal notation (see below). The assembler requires the exact
    decimal value of a floating-point constant. For example, the
    assembler accepts 1.25 but rejects 1.3 because 1.3 is a repeating
    decimal in binary. Floating-point constants must have a
    `floating-point <t_floating>`{.interpreted-text role="ref"} type.

**Null pointer constants**

:   The identifier \'`null`\' is recognized as a null pointer constant
    and must be of `pointer type <t_pointer>`{.interpreted-text
    role="ref"}.

**Token constants**

:   The identifier \'`none`\' is recognized as an empty token constant
    and must be of `token type <t_token>`{.interpreted-text role="ref"}.

The one non-intuitive notation for constants is the hexadecimal form of
floating-point constants. For example, the form
\'`double    0x432ff973cafa8000`\' is equivalent to (but harder to read
than) \'`double 4.5e+15`\'. The only time hexadecimal floating-point
constants are required (and the only time that they are generated by the
disassembler) is when a floating-point constant must be emitted but it
cannot be represented as a decimal floating-point number in a reasonable
number of digits. For example, NaN\'s, infinities, and other special
values are represented in their IEEE hexadecimal format so that assembly
and disassembly do not cause any bits to change in the constants.

When using the hexadecimal form, constants of types half, float, and
double are represented using the 16-digit form shown above (which
matches the IEEE754 representation for double); half and float values
must, however, be exactly representable as IEEE 754 half and single
precision, respectively. Hexadecimal format is always used for long
double, and there are three forms of long double. The 80-bit format used
by x86 is represented as `0xK` followed by 20 hexadecimal digits. The
128-bit format used by PowerPC (two adjacent doubles) is represented by
`0xM` followed by 32 hexadecimal digits. The IEEE 128-bit format is
represented by `0xL` followed by 32 hexadecimal digits. Long doubles
will only work if they match the long double format on your target. The
IEEE 16-bit format (half precision) is represented by `0xH` followed by
4 hexadecimal digits. All hexadecimal formats are big-endian (sign bit
at the left).

There are no constants of type x86\_mmx.

### Complex Constants {#complexconstants}

Complex constants are a (potentially recursive) combination of simple
constants and smaller complex constants.

**Structure constants**

:   Structure constants are represented with notation similar to
    structure type definitions (a comma separated list of elements,
    surrounded by braces (`{}`)). For example:
    \"`{ i32 4, float 17.0, i32* @G }`\", where \"`@G`\" is declared as
    \"`@G = external global i32`\". Structure constants must have
    `structure type <t_struct>`{.interpreted-text role="ref"}, and the
    number and types of elements must match those specified by the type.

**Array constants**

:   Array constants are represented with notation similar to array type
    definitions (a comma separated list of elements, surrounded by
    square brackets (`[]`)). For example:
    \"`[ i32 42, i32 11, i32 74 ]`\". Array constants must have
    `array type <t_array>`{.interpreted-text role="ref"}, and the number
    and types of elements must match those specified by the type. As a
    special case, character array constants may also be represented as a
    double-quoted string using the `c` prefix. For example:
    \"`c"Hello World\0A\00"`\".

**Vector constants**

:   Vector constants are represented with notation similar to vector
    type definitions (a comma separated list of elements, surrounded by
    less-than/greater-than\'s (`<>`)). For example:
    \"`< i32 42, i32 11, i32 74, i32 100 >`\". Vector constants must
    have `vector type <t_vector>`{.interpreted-text role="ref"}, and the
    number and types of elements must match those specified by the type.

**Zero initialization**

:   The string \'`zeroinitializer`\' can be used to zero initialize a
    value to zero of *any* type, including scalar and
    `aggregate <t_aggregate>`{.interpreted-text role="ref"} types. This
    is often used to avoid having to print large zero initializers (e.g.
    for large arrays) and is always exactly equivalent to using explicit
    zero initializers.

**Metadata node**

:   A metadata node is a constant tuple without types. For example:
    \"`!{!0, !{!2, !0}, !"test"}`\". Metadata can reference constant
    values, for example:
    \"`!{!0, i32 0, i8* @global, i64 (i64)* @function, !"str"}`\".
    Unlike other typed constants that are meant to be interpreted as
    part of the instruction stream, metadata is a place to attach
    additional information such as debug info.

### Global Variable and Function Addresses

The addresses of `global variables <globalvars>`{.interpreted-text
role="ref"} and `functions <functionstructure>`{.interpreted-text
role="ref"} are always implicitly valid (link-time) constants. These
constants are explicitly referenced when the
`identifier for the global <identifiers>`{.interpreted-text role="ref"}
is used and always have `pointer <t_pointer>`{.interpreted-text
role="ref"} type. For example, the following is a legal LLVM file:

``` {.llvm}
@X = global i32 17
@Y = global i32 42
@Z = global [2 x i32*] [ i32* @X, i32* @Y ]
```

### Undefined Values {#undefvalues}

The string \'`undef`\' can be used anywhere a constant is expected, and
indicates that the user of the value may receive an unspecified
bit-pattern. Undefined values may be of any type (other than \'`label`\'
or \'`void`\') and be used anywhere a constant is permitted.

Undefined values are useful because they indicate to the compiler that
the program is well defined no matter what value is used. This gives the
compiler more freedom to optimize. Here are some examples of
(potentially surprising) transformations that are valid (in pseudo IR):

``` {.llvm}
%A = add %X, undef
%B = sub %X, undef
%C = xor %X, undef
```

> Safe:
>
> :   \%A = undef %B = undef %C = undef
>
This is safe because all of the output bits are affected by the undef
bits. Any output bit can have a zero or one depending on the input bits.

``` {.llvm}
%A = or %X, undef
%B = and %X, undef
```

> Safe:
>
> :   \%A = -1 %B = 0
>
> Safe:
>
> :   \%A = %X ;; By choosing undef as 0 %B = %X ;; By choosing undef as
>     -1
>
> Unsafe:
>
> :   \%A = undef %B = undef
>
These logical operations have bits that are not always affected by the
input. For example, if `%X` has a zero bit, then the output of the
\'`and`\' operation will always be a zero for that bit, no matter what
the corresponding bit from the \'`undef`\' is. As such, it is unsafe to
optimize or assume that the result of the \'`and`\' is \'`undef`\'.
However, it is safe to assume that all bits of the \'`undef`\' could be
0, and optimize the \'`and`\' to 0. Likewise, it is safe to assume that
all the bits of the \'`undef`\' operand to the \'`or`\' could be set,
allowing the \'`or`\' to be folded to -1.

``` {.llvm}
%A = select undef, %X, %Y
%B = select undef, 42, %Y
%C = select %X, %Y, undef
```

> Safe:
>
> :   \%A = %X (or %Y) %B = 42 (or %Y) %C = %Y
>
> Unsafe:
>
> :   \%A = undef %B = undef %C = undef
>
This set of examples shows that undefined \'`select`\' (and conditional
branch) conditions can go *either way*, but they have to come from one
of the two operands. In the `%A` example, if `%X` and `%Y` were both
known to have a clear low bit, then `%A` would have to have a cleared
low bit. However, in the `%C` example, the optimizer is allowed to
assume that the \'`undef`\' operand could be the same as `%Y`, allowing
the whole \'`select`\' to be eliminated.

``` {.text}
%A = xor undef, undef

%B = undef
%C = xor %B, %B

%D = undef
%E = icmp slt %D, 4
%F = icmp gte %D, 4
```

> Safe:
>
> :   \%A = undef %B = undef %C = undef %D = undef %E = undef %F = undef
>
This example points out that two \'`undef`\' operands are not
necessarily the same. This can be surprising to people (and also matches
C semantics) where they assume that \"`X^X`\" is always zero, even if
`X` is undefined. This isn\'t true for a number of reasons, but the
short answer is that an \'`undef`\' \"variable\" can arbitrarily change
its value over its \"live range\". This is true because the variable
doesn\'t actually *have a live range*. Instead, the value is logically
read from arbitrary registers that happen to be around when needed, so
the value is not necessarily consistent over time. In fact, `%A` and
`%C` need to have the same semantics or the core LLVM \"replace all uses
with\" concept would not hold.

``` {.llvm}
%A = sdiv undef, %X
%B = sdiv %X, undef
```

> Safe:
>
> :   \%A = 0
>
> b: unreachable

These examples show the crucial difference between an *undefined value*
and *undefined behavior*. An undefined value (like \'`undef`\') is
allowed to have an arbitrary bit-pattern. This means that the `%A`
operation can be constant folded to \'`0`\', because the \'`undef`\'
could be zero, and zero divided by any value is zero. However, in the
second example, we can make a more aggressive assumption: because the
`undef` is allowed to be an arbitrary value, we are allowed to assume
that it could be zero. Since a divide by zero has *undefined behavior*,
we are allowed to assume that the operation does not execute at all.
This allows us to delete the divide and all code after it. Because the
undefined operation \"can\'t happen\", the optimizer can assume that it
occurs in dead code.

``` {.text}
a:  store undef -> %X
b:  store %X -> undef
Safe:
a: <deleted>
b: unreachable
```

A store *of* an undefined value can be assumed to not have any effect;
we can assume that the value is overwritten with bits that happen to
match what was already there. However, a store *to* an undefined
location could clobber arbitrary memory, therefore, it has undefined
behavior.

### Poison Values {#poisonvalues}

Poison values are similar to
`undef values <undefvalues>`{.interpreted-text role="ref"}, however they
also represent the fact that an instruction or constant expression that
cannot evoke side effects has nevertheless detected a condition that
results in undefined behavior.

There is currently no way of representing a poison value in the IR; they
only exist when produced by operations such as
`add <i_add>`{.interpreted-text role="ref"} with the `nsw` flag.

Poison value behavior is defined in terms of value *dependence*:

-   Values other than `phi <i_phi>`{.interpreted-text role="ref"} nodes
    depend on their operands.
-   `Phi <i_phi>`{.interpreted-text role="ref"} nodes depend on the
    operand corresponding to their dynamic predecessor basic block.
-   Function arguments depend on the corresponding actual argument
    values in the dynamic callers of their functions.
-   `Call <i_call>`{.interpreted-text role="ref"} instructions depend on
    the `ret <i_ret>`{.interpreted-text role="ref"} instructions that
    dynamically transfer control back to them.
-   `Invoke <i_invoke>`{.interpreted-text role="ref"} instructions
    depend on the `ret <i_ret>`{.interpreted-text role="ref"},
    `resume <i_resume>`{.interpreted-text role="ref"}, or
    exception-throwing call instructions that dynamically transfer
    control back to them.
-   Non-volatile loads and stores depend on the most recent stores to
    all of the referenced memory addresses, following the order in the
    IR (including loads and stores implied by intrinsics such as
    `@llvm.memcpy <int_memcpy>`{.interpreted-text role="ref"}.)
-   An instruction with externally visible side effects depends on the
    most recent preceding instruction with externally visible side
    effects, following the order in the IR. (This includes `volatile
    operations <volatile>`{.interpreted-text role="ref"}.)
-   An instruction *control-depends* on a `terminator
    instruction <terminators>`{.interpreted-text role="ref"} if the
    terminator instruction has multiple successors and the instruction
    is always executed when control transfers to one of the successors,
    and may not be executed when control is transferred to another.
-   Additionally, an instruction also *control-depends* on a terminator
    instruction if the set of instructions it otherwise depends on would
    be different if the terminator had transferred control to a
    different successor.
-   Dependence is transitive.

Poison values have the same behavior as
`undef values <undefvalues>`{.interpreted-text role="ref"}, with the
additional effect that any instruction that has a *dependence* on a
poison value has undefined behavior.

Here are some examples:

``` {.llvm}
entry:
  %poison = sub nuw i32 0, 1           ; Results in a poison value.
  %still_poison = and i32 %poison, 0   ; 0, but also poison.
  %poison_yet_again = getelementptr i32, i32* @h, i32 %still_poison
  store i32 0, i32* %poison_yet_again  ; memory at @h[0] is poisoned

  store i32 %poison, i32* @g           ; Poison value stored to memory.
  %poison2 = load i32, i32* @g         ; Poison value loaded back from memory.

  store volatile i32 %poison, i32* @g  ; External observation; undefined behavior.

  %narrowaddr = bitcast i32* @g to i16*
  %wideaddr = bitcast i32* @g to i64*
  %poison3 = load i16, i16* %narrowaddr ; Returns a poison value.
  %poison4 = load i64, i64* %wideaddr  ; Returns a poison value.

  %cmp = icmp slt i32 %poison, 0       ; Returns a poison value.
  br i1 %cmp, label %true, label %end  ; Branch to either destination.

true:
  store volatile i32 0, i32* @g        ; This is control-dependent on %cmp, so
                                       ; it has undefined behavior.
  br label %end

end:
  %p = phi i32 [ 0, %entry ], [ 1, %true ]
                                       ; Both edges into this PHI are
                                       ; control-dependent on %cmp, so this
                                       ; always results in a poison value.

  store volatile i32 0, i32* @g        ; This would depend on the store in %true
                                       ; if %cmp is true, or the store in %entry
                                       ; otherwise, so this is undefined behavior.

  br i1 %cmp, label %second_true, label %second_end
                                       ; The same branch again, but this time the
                                       ; true block doesn't have side effects.

second_true:
  ; No side effects!
  ret void

second_end:
  store volatile i32 0, i32* @g        ; This time, the instruction always depends
                                       ; on the store in %end. Also, it is
                                       ; control-equivalent to %end, so this is
                                       ; well-defined (ignoring earlier undefined
                                       ; behavior in this example).
```

### Addresses of Basic Blocks {#blockaddress}

`blockaddress(@function, %block)`

The \'`blockaddress`\' constant computes the address of the specified
basic block in the specified function, and always has an `i8*` type.
Taking the address of the entry block is illegal.

This value only has defined behavior when used as an operand to the
\'`indirectbr <i_indirectbr>`{.interpreted-text role="ref"}\'
instruction, or for comparisons against null. Pointer equality tests
between labels addresses results in undefined behavior \-\-- though,
again, comparison against null is ok, and no label is equal to the null
pointer. This may be passed around as an opaque pointer sized value as
long as the bits are not inspected. This allows `ptrtoint` and
arithmetic to be performed on these values so long as the original value
is reconstituted before the `indirectbr` instruction.

Finally, some targets may provide defined semantics when using the value
as the operand to an inline assembly, but that is target specific.

### Constant Expressions {#constantexprs}

Constant expressions are used to allow expressions involving other
constants to be used as constants. Constant expressions may be of any
`first class <t_firstclass>`{.interpreted-text role="ref"} type and may
involve any LLVM operation that does not have side effects (e.g. load
and call are not supported). The following is the syntax for constant
expressions:

`trunc (CST to TYPE)`

:   Perform the `trunc operation <i_trunc>`{.interpreted-text
    role="ref"} on constants.

`zext (CST to TYPE)`

:   Perform the `zext operation <i_zext>`{.interpreted-text role="ref"}
    on constants.

`sext (CST to TYPE)`

:   Perform the `sext operation <i_sext>`{.interpreted-text role="ref"}
    on constants.

`fptrunc (CST to TYPE)`

:   Truncate a floating-point constant to another floating-point type.
    The size of CST must be larger than the size of TYPE. Both types
    must be floating-point.

`fpext (CST to TYPE)`

:   Floating-point extend a constant to another type. The size of CST
    must be smaller or equal to the size of TYPE. Both types must be
    floating-point.

`fptoui (CST to TYPE)`

:   Convert a floating-point constant to the corresponding unsigned
    integer constant. TYPE must be a scalar or vector integer type. CST
    must be of scalar or vector floating-point type. Both CST and TYPE
    must be scalars, or vectors of the same number of elements. If the
    value won\'t fit in the integer type, the result is a
    `poison value <poisonvalues>`{.interpreted-text role="ref"}.

`fptosi (CST to TYPE)`

:   Convert a floating-point constant to the corresponding signed
    integer constant. TYPE must be a scalar or vector integer type. CST
    must be of scalar or vector floating-point type. Both CST and TYPE
    must be scalars, or vectors of the same number of elements. If the
    value won\'t fit in the integer type, the result is a
    `poison value <poisonvalues>`{.interpreted-text role="ref"}.

`uitofp (CST to TYPE)`

:   Convert an unsigned integer constant to the corresponding
    floating-point constant. TYPE must be a scalar or vector
    floating-point type. CST must be of scalar or vector integer type.
    Both CST and TYPE must be scalars, or vectors of the same number of
    elements.

`sitofp (CST to TYPE)`

:   Convert a signed integer constant to the corresponding
    floating-point constant. TYPE must be a scalar or vector
    floating-point type. CST must be of scalar or vector integer type.
    Both CST and TYPE must be scalars, or vectors of the same number of
    elements.

`ptrtoint (CST to TYPE)`

:   Perform the `ptrtoint operation <i_ptrtoint>`{.interpreted-text
    role="ref"} on constants.

`inttoptr (CST to TYPE)`

:   Perform the `inttoptr operation <i_inttoptr>`{.interpreted-text
    role="ref"} on constants. This one is *really* dangerous!

`bitcast (CST to TYPE)`

:   Convert a constant, CST, to another TYPE. The constraints of the
    operands are the same as those for the
    `bitcast instruction <i_bitcast>`{.interpreted-text role="ref"}.

`addrspacecast (CST to TYPE)`

:   Convert a constant pointer or constant vector of pointer, CST, to
    another TYPE in a different address space. The constraints of the
    operands are the same as those for the
    `addrspacecast instruction <i_addrspacecast>`{.interpreted-text
    role="ref"}.

`getelementptr (TY, CSTPTR, IDX0, IDX1, ...)`, `getelementptr inbounds (TY, CSTPTR, IDX0, IDX1, ...)`

:   Perform the
    `getelementptr operation <i_getelementptr>`{.interpreted-text
    role="ref"} on constants. As with the
    `getelementptr <i_getelementptr>`{.interpreted-text role="ref"}
    instruction, the index list may have one or more indexes, which are
    required to make sense for the type of \"pointer to TY\".

`select (COND, VAL1, VAL2)`

:   Perform the `select operation <i_select>`{.interpreted-text
    role="ref"} on constants.

`icmp COND (VAL1, VAL2)`

:   Perform the `icmp operation <i_icmp>`{.interpreted-text role="ref"}
    on constants.

`fcmp COND (VAL1, VAL2)`

:   Perform the `fcmp operation <i_fcmp>`{.interpreted-text role="ref"}
    on constants.

`extractelement (VAL, IDX)`

:   Perform the
    `extractelement operation <i_extractelement>`{.interpreted-text
    role="ref"} on constants.

`insertelement (VAL, ELT, IDX)`

:   Perform the
    `insertelement operation <i_insertelement>`{.interpreted-text
    role="ref"} on constants.

`shufflevector (VEC1, VEC2, IDXMASK)`

:   Perform the
    `shufflevector operation <i_shufflevector>`{.interpreted-text
    role="ref"} on constants.

`extractvalue (VAL, IDX0, IDX1, ...)`

:   Perform the
    `extractvalue operation <i_extractvalue>`{.interpreted-text
    role="ref"} on constants. The index list is interpreted in a similar
    manner as indices in a
    \'`getelementptr <i_getelementptr>`{.interpreted-text role="ref"}\'
    operation. At least one index value must be specified.

`insertvalue (VAL, ELT, IDX0, IDX1, ...)`

:   Perform the
    `insertvalue operation <i_insertvalue>`{.interpreted-text
    role="ref"} on constants. The index list is interpreted in a similar
    manner as indices in a
    \'`getelementptr <i_getelementptr>`{.interpreted-text role="ref"}\'
    operation. At least one index value must be specified.

`OPCODE (LHS, RHS)`

:   Perform the specified operation of the LHS and RHS constants. OPCODE
    may be any of the `binary <binaryops>`{.interpreted-text role="ref"}
    or `bitwise
    binary <bitwiseops>`{.interpreted-text role="ref"} operations. The
    constraints on operands are the same as those for the corresponding
    instruction (e.g. no bitwise operations on floating-point values are
    allowed).
