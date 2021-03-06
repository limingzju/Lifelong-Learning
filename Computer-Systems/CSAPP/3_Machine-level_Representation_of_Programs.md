# Machine-Level Representation of Programs

reverse engineering := trying to understand the process by which a system was created by studying the system and working backward

## A Historical Perspective

* 8086 := (1978, 29 K transistors). One of the first single-chip, 16-bit microprocessors.
* 8088 := a variant of the 8086 with an 8-bit external bus, formed the heart of the original IBM personal computers.
* 8087 := a floating-point coprocessor (45 K transistors) to operate alongside an 8086 or 8088 processor.
* 80286 := (1982, 134 K transistors).
* i386 := (1985, 275K transistors). Expanded the architecture to 32bits. Addedthe flat addressing model.
* i486 := (1989, 1.2 M transistors).
* Pentium := (1993, 3.1 M transistors).
* PentiumPro := (1995, 5.5 M transistors). P6 microarchitecture. Added a class of “conditional move” instructions to the instruction set.
* Pentium II := (1997, 7 M transistors).
* Pentium III := (1999, 8.2 M transistors). Introduced SSE, a class of instructions for manipulating vectors of integer or floating-point data.
* Pentium 4 := (2000, 42 M transistors). Extended SSE to SSE2, adding new data types.
* Pentium 4E := (2004,125Mtransistors).Addedhyperthreading,amethodtorun two programs simultaneously on a single processor, as well as EM64T, Intel’s implementation of a 64-bit extension to IA32 developed by Ad- vanced Micro Devices (AMD), which we refer to as x86-64.
* Core 2 := (2006,291Mtransistors).Returned back to a microarchitecture similar to P6. First multi-core Intel microprocessor, where multiple processors are implemented on a single chip. Did not support hyperthreading.
* Core 7 := (2008, 781 M transistors). Incorporated both hyperthreading and multi-core, with the initial version supporting two executing programs on each core and up to four cores on each chip.

Moore's Law := the number of transistors per chip would double every year for the next 10 years.

## Program Encodings

Two kinds of machine code we will consider:

* Object code
* Executable code

For machine level programming:

* *instruction set architecture*, or **“ISA,”** (including IA32 and x86-64) defines the processor state, the format of the instructions, and the effect each of these instructions will have on the state.
* The memory addresses used by a machine-level program are virtual addresses, providing a memory model that appears to be a very large byte array.

Parts of the processor state are visible that normally are hidden from the C programmer:

* The *program counter* (commonly referred to as the “PC,” and called %eip in IA32)
* The *integer register* file contains eight named locations storing 32-bit values.
* The *condition code registers* hold status information about the most recently executed arithmetic or logical instruction.
* A set of *floating-point registers* store floating-point data.

The program memory contains the executable machine code for the program, some information required by the operating system, a run-time stack for managing procedure calls and returns, and blocks of memory allocated by the user (for example, by using the malloc library function).

Several features about machine code and its disassembled representation are worth noting:

* IA32 instructions can range in length from 1 to 15 bytes.
* from a given starting position, there is a unique decoding of the bytes into machine instructions.
* The disassembler determines the assembly code based purely on the byte sequences in the machine-code file.
* The disassembler uses a slightly different naming convention for the instructions than does the assembly code generated by gcc.

## Data Formats

|C declaration|Intel data type|Assembly code suffix|Size(bytes)|
|-|-|-|-|
|char|Byte|b|1|
|short|Word|w|2|
|int|Double word|l|4|
|long int|Double word|l|4|
|long long int|-|-|4|
|char *|Double word|l|4|
|float|Single precision|s|4|
|double|Double precision|l|8|
|long double|Extented precision|t|10/12|

## Accessing Information

IA32 Integer Registers contains a set of eight *registers* storing 32-bit values:

|31...16|15...8|7...0||
|-|-|-|-|
|%eax %ax|%ah|%al||
|%ecx %cx|%ch|%cl||
|%edx %dx|%dh|%dl||
|%ebx %bx|%bh|%bl||
|%esi %si||||
|%edi %di||||
|%esp %sp|||Stack Pointer|
|%ebp %bp|||Frame Pointer|

### Operand Specifiers

* *immediate* is for constant values. written with a ‘$’ followed by an integer
* *register* denotes the contents of one of the registers.
* *memory* reference accesses some memory location according to a computed address (often called the *effective address*).

The most general form of memory references is Imm(Eb,Ei,s). Such a reference has four components: an immediate offset Imm, a base register Eb, an index register Ei, and a scale factor s, where s must be 1, 2, 4, or 8. The operand value is M[Imm + R[Eb] + R[Ei]·s].

### Data Movement Instructions

|Instruction|Effect|Description|
|-|-|-|
|MOV  S,D|D←S|Move|
|movb|Move byte|-|
|movw|Move word|-|
|movl|Move double word|-|
|MOVS S,D|D←SignExtend(S)|Move with sign extension|
|movsbw|Move sign-extended byte to word|-|
|movsbl|Move sign-extended byte to double word|-|
|movswl|Move sign-extended word to double word|-|
|MOVZ S,D|D←ZeroExtend(S)|Move with zero extension|
|movsbw|Move zero-extended byte to word|-|
|movsbl|Move zero-extended byte to double word|-|
|movswl|Move zero-extended word to double word|-|
|pushl S|R[%esp]←R[%esp]-4;M[R[%esp]]←S|Push double word|
|popl D|D←M[R[%esp]];R[%esp]←R[%esp]+4|Pop double word|

IA32 imposes the restriction that a move instruction cannot have both operands refer to memory locations.

Since the stack is contained in the same memory as the program code and other forms of program data, programs can access arbitrary positions within the stack using the standard memory addressing methods. For example, assuming the topmost element of the stack is a double word, the instruction movl 4(%esp),%edx will copy the second double word from the stack to register %edx.

## Arithmetic and Logical Operations

|Instruction|Effect|Description|
|-|-|-|
|leal S,D|D←&S|Load effective address|
|INC D|D←D+1|Increment|
|DEC D|D←D-1|Decrement|
|NEG D|D←-D|Negate|
|NOT D|D←~D|Complement|
|ADD S,D|D←D+S|Add|
|SUB S,D|D←D-S|Subtract|
|IMUL S,D|D←D*S|Multiply|
|XOR S,D|D←D^S|Exclusive-or|
|OR S,D|D←D\|S|Or|
|AND S,D|D←D&S|And|
|SAL k,D|D←D<<k|Left shift|
|SHL k,D|D←D<<k|Left shift (same as SAL)|
|SAR k,D|D←D>>_Ak|Arithmetic right shift|
|SHR k,D|D←D>>_Lk|Logical right shift|

### Special Arithmetic Operations

|Instruction|Effect|Description|
|-|-|-|
|imull S|R[%edx]:R[%eax] ← S × R[%eax]|Signed full multiply|
|mull S|R[%edx]:R[%eax] ← S × R[%eax]|Unsigned full multiply|
|cltd|R[%edx]:R[%eax] ← SignExtend(R[%eax])|Convert to quad word|
|idivl S|R[%edx] ← R[%edx]:R[%eax] mod S; R[%eax] ← R[%edx]:R[%eax] ÷ S|Signed divide|
|divl S|R[%edx] ← R[%edx]:R[%eax] mod S; R[%eax] ← R[%edx]:R[%eax] ÷ S|Unsigned divide|

## Control

Machine code provides two basic low-level mechanisms for implementing conditional behavior: it tests data values and then either alters the control flow or the data flow based on the result of these tests.

### Condition Codes

* CF: Carry Flag. The most recent operation generated a carry out of the most significant bit. Used to detect overflow for unsigned operations.
* ZF: Zero Flag. The most recent operation yielded zero.
* SF: Sign Flag. The most recent operation yielded a negative value.
* OF: Overflow Flag. The most recent operation caused a two’s-complement overflow—either negative or positive.

### Accessing the Condition Codes

|Instruction|Based on|Description|
|-|-|-|
|CMP S2,S1|S1-S2|Compare|
|TEST S2,S1|S1 & S2|Test|

For C assignment t=a+b:

* CF: (unsigned) t < (unsigned) a Unsigned overflow
* ZF: (t == 0) Zero
* SF: (t < 0) Negative
* OF:(a < 0 == b < 0) && (t < 0 != a < 0) Signed overflow

|Instruction|Synonym|Effect|Set condition|
|-|-|-|-|
|sete D|setz|D←ZF|Eaual/Zero|
|setne D|setnz|D←~ZF|Not equal/not zero|
|sets D||D←SF|Negative|
|setns D||D←~SF|Nonnegative|
|setg D|setnle|D←~(SF^OF)&~ZF|Greater (signed >)|
|setge D|setnl|D←~(SF^OF)|Greater or equal (signed >=)|
|setl D|setnge|D←SF^OF|Less (signed <)|
|setle D|setng|D←(SF^OF)\|ZF|Less or equal (signed <=)|
|seta D|setnbe|D←~CF&~ZF|Above (unsigned >)|
|setae D|setnb|D←~CF|Above or equal (unsigned >=)|
|setb D|setnae|D←CF|Below (unsigned <)|
|setbe|setna|D←CF\|ZF|Below or equal (unsigned <=)|

### Jump Instructions and Their Encodings

|Instruction|Synonym|Jump condition|Description|
|-|-|-|-|
|jmp Label||1|Direct Jump|
|jmp *Operand||1|Indirect jump|
|je Label|jz|ZF|Equal/zero|
|jne Label|jnz|~ZF|Not equal/not zero|
|js Label||SF|Negative|
|jns Label||~SF|Nonnegative|
|jg Label|jnle|~(SF^OF)&~ZF|Greater (signed >)|
|jge Label|jnl|~(SF^OF)|Greater or equal (signed >=)|
|jl Label|jnge|SF^OF|Less (signed <)|
|jle Label|jng|(SF^OF)\|ZF|Less or equal (signed <=)|
|ja Label|jnbe|~CF&~ZF|Above (unsigned >)|
|jae Label|jnb|~CF|Above or equal (unsigned >=)|
|jb Label|jnae|CF|Below (unsigned <)|
|jbe Label|jna|CF\|ZF|Below or equal (unsigned <=)|

The value of the program counter when performing PC-relative addressing is the address of the instruction following the jump, not that of the jump itself. This convention dates back to early implementations, when the processor would update the program counter as its first step in executing an instruction.

### Translating Conditional Branches

The general form of an if-else statement in C is given by the template:

```s
if (test-expr)
    then-statement
else
    else-statement
```

where we use C syntax to describe the control flow:

```s
    t = test-expr;
    if (!t)
        goto false;
    then-statement
    goto done;
false:
    else-statement
done:
```

### Loops

The general form of a do-while statement is as follows:

```s
do
    body-statement
    while (test-expr);
```

This general form can be translated into conditionals and goto statements as follows:

```s
loop:
    body-statement
    t = test-expr;
    if (t)
        goto loop;
```

A key to understanding how the generated assembly code relates to the original source code is to find a mapping between program values and registers.

The general form of a while statement is as follows:

```s
while (test-expr)
    body-statement
```

One common approach to transform the code into a do-while loop by using a conditional branch to skip the first execution of the body if needed:

```s
    t = test-expr;
    if (!t)
        goto done;
loop:
    body-statement
    t = test-expr;
    if (t)
        goto loop;
done:
```

The general form of a for loop is as follows:

```s
for (init-expr; test-expr; update-expr)
    body-statement
```

This, in turn, can be transformed into goto code as:

```s
    init-expr;
    t = test-expr;
    if (!t)
        goto done;
loop:
    body-statement
    update-expr;
    t = test-expr;
    if (t)
        goto loop;
done:
```

### Conditional Move Instructions

|Instruction|Synonym|Move condition|Description|
|-|-|-|-|
|cmove S,R|cmovz|ZF|Equal/zero|
|cmovne S,R|cmovnz|~ZF|Not equal/not zero|
|cmovs S,R||SF|Negative|
|cmovns S,R||~SF|Nonnegative|
|cmovg S,R|cmovnle|~(SF^OF)&~ZF|Greater (signed >)|
|cmovge S,R|cmovnl|~(SF^OF)|Greater or equal (signed >=)|
|cmovl S,R|cmovnge|SF^OF|Less (signed <)|
|cmovle S,R|cmovng|(SF^OF)\|ZF|Less or equal (signed <=)|
|cmova S,R|cmovnbe|~CF&~ZF|Above (unsigned >)|
|cmovae S,R|cmovnb|~CF|Above or equal (unsigned >=)|
|cmovb S,R|cmovnae|CF|Below (unsigned <)|
|cmovbe S,R|cmovna|CF\|ZF|Below or equal (unsigned <=)|

Processors employ sophisticated branch prediction logic to try to guess whether or not each jump instruction will be followed. As long as it can guess reliably (modern microprocessor designs try to achieve success rates on the order of 90%), the instruction pipeline will be kept full of instructions. Mispredicting a jump, on the other hand, requires that the processor discard much of the work it has already done on future instructions and then begin filling the pipeline with instructions starting at the correct location. As we will see, such a misprediction can incur a serious penalty, say, 20–40 clock cycles of wasted effort, causing a serious degradation of program performance.

To understand how conditional operations can be implemented via condi- tional data transfers, consider the following general form of conditional expression and assignment:

```s
v = test-expr ? then-expr : else-expr;
```

With traditional IA32, the compiler generates code having a form shown by the
following abstract code:

```s
    if (!test-expr)
        goto false;
    v = true-expr;
    goto done;
false:
    v = else-expr;
done:
```

If one of those two expressions then-expr and else-expr could possibly generate an error condition or a side effect, this could lead to invalid behavior.

### Switch Statements

A jump table is an array where entry i is the address of a code segment implementing the action the program should take when the switch index equals i. The code performs an array reference into the jump table using the switch index to determine the target for a jump instruction.
