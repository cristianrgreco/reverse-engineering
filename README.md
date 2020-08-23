# Reverse Engineering

## Definitions

- ZF: zero flag

## Assembly

### Memory

#### Stack

A section of memory used during program execution. Each program usually has
a static size, usually to store function parameters. Function parameters are
pushed onto the stack, the function is called, and the function either addresses
the stack directly or pops the parameters from the stack.

Each thread gets its own stack.

The stack pointer points to the top of the stack. Freeing a block from the stack
is as simple as adjusting the pointer.

#### Heap

Memory set aside for dynamic allocation. Unlike the stack there is no enforced pattern
for usage. Allocation and deallocation of blocks can happen at any time.

This makes it harder to keep track of which parts of the heap are in use at any given
time, and as such there are heap allocators available to tune performance for different
usage patterns. Think Java's GC.

### Registers

Note that there are different registers depending on the size.

`rax` = 64-bit (long).
`eax` = 32-bit, (int).
`ax` = 16-bit, (short).
`ah`/`al` = 8-bit, (char).

**rax** = return values.
**rsp** = stack pointer (points to the top of the stack).
**rbp** = base pointer (sometimes used to store the old value of the stack pointer).

Other registers such as `rdx`, `rsi`, `rdi`, `r{8..15}` are usually used asscratch 
registers, though this depends as certain instructions, for example `movsb` takes the
source address from `esi` and the destination from `edi`, copies one byte and changes
both registers. So, they can be used as general purpose reigsters, but they also might
have a specific purpose for certain instructions.

### Instructions

Overview of the different CPU operations (Intel x86).

#### PUSH

Write a value to the stack:

```
push 0x1
```

#### POP

Restore the top value on the stack into a register:

```
push 0x1
pop eax ; eax = 0x1
```

#### MOV

Can be interpreted as as assignment statement:

```
mov rax, qword [objc_release_1000]
```

Will assign the result of `objc_release_1000` to `rax`.

#### CMP

Subtracts the two operands, setting the ZF when the difference is zero.

#### TEST

Performs a bitwise `AND` on the two operands, setting the ZF when the reult is zero.

```
0 & 0 ; ZF=0
0 & 1 ; ZF=0
1 & 1 ; ZF=1
```

#### XOR

Exclusive or, "are A and B not equal". 
Why not say that one must be true? Because for the result to be one, one of the operands must be true.

In this example, you can see if `eax` is `1`, then the result is `0`.

```
mov eax, 0x1
xor eax eax ; eax=0
```

Indeed, the `xor` operation is commonly used for clearing the register. The equivalent
of the above would be:

```
mov eax, 0x0
```

## Modifying Binaries

If for example we wanted to make `rax` return `1`, for example if this will be used in a 
boolean comparison, then we could replace the instruction as follows:

```
mov rax, 0x1
```

If there exists a function which performs some check, and if the check succeeds then
the program will jump to a desired location, then we can bypass the check and jump 
directly to the location, we could for example modify the instruction which jumps to the
check code to jump directly to the desired address by having Hopper assemble this instruction:

```
jmp 000123
```

## Objective-C

Consider this Objective-C code:

```
NSNumber *number = [[NSNumber alloc] initWithInt:1];
```

If we were to compile and then decompile this code, we would get:

```
int v1; int v2; 
v1 = objc_msgSend(_OBJC_CLASS_$_NSNUMBER, "alloc"); 
v2 = objc_msgSend(v1, "initWithInt", 1); 
```

You can see that every call to a method is performed by calling the runtime:

```
id objc_msgSend(id self, SEL op, …);
```

The first argument is a pointer to an object, and the second is to a selector. These
two arguments are mandatory. Any number of additional, optional arguments can be provided.

The runtime operates two types: `SEL` and `IMP`. 
The selector is the human readable name of the method. 
The implementation is a pointer to a C function.

We can see the main purpose of `ojbc_msgSend` is to find the implementation for a given
selector and object, and call it by passing all specified arguments.

There are therefore two specific nuances:

1. It is impossible to find a direct call to a method implementation. The selector is the key
for searching implementations by name.
2. The selector name is a good hint for understanding the executable code.

All selector names can be found in the `__objc_methname` setion of the `__TEXT` segment.

### Hopper

Make use of Hopper's pseudo-code generation feature!

Hopper is able to assemble instructions with Option+A when cursor is on the line, example:

```
jmp 000123
```

## Debugging

### LLDB

Read all registers:

```
register read
```

Read register value:

```
register read rbx
```

Read the next 5 bytes (c5) in hex (fh) (fu for decimal) where the byte size is 8 (qword):

```
memory read -s8 -fh -c5 0x000060000147f900
```

## Number systems

### Hexadecimal

Hexadecimal is useful as a single hex digit represents four bits (2x2x2x2, 2^4).

Each index in the value represents a power of 16. For example:

0x1  ; 1 * 1 = 1
0x10 ; 0 + (16 * 1) = 16
0x1A ; 10 + (16 * 1) = 26

## References

- [x86 JUMP reference](http://unixwiz.net/techtips/x86-jumps.html)
- [Modify function return values with Hopper Disassembler](https://www.programmersought.com/article/3300383897/)
- [Reverse Engineering: Cracking Sublime Text 3](https://web.archive.org/web/20181018131928/http://blog.fernandodominguez.me/cracking-sublime-text-3)
- [Cracking Tutorial - Sandwich](https://reverse.put.as/wp-content/uploads/2012/06/Sandwich_crackme_tut_qwertyoruiop.txt)
- [Reverse Engineering Blog](https://reverse.put.as/post/)
- [Collection of resources](https://github.com/michalmalik/osx-re-101)
