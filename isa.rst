Instruction Set Architecture
============================
Chapter 2

an ISA is the hardware/software interface
- agreement between programmer (compiler constructor) and hardware
- defines the visible state of the system (registers, memory)
- defines how instructions change processor state
- defines the instructions and encodings

it does not define:
- how instructions are implemented
- how fast/slow instructions are
- how much powers instructions consume

Many different chips implement the same ISA, implementations are short lived but ISAs live a long time

ISA Classification History
--------------------------
Chronologically:

- Accumulator
    - only one register
    - simple hardware
    - lots of memory traffic
- Stack-based
    - instructions access top of stack
    - code density (good)
    - stack can be bottleneck
    - operative instructions operate on top of stack
- Registers
    - modern architecture
    - RISC (Reduced Instruction Set Computer), AKA Load-Store architecture
    - CISC (Complex ...), can operate on memory too

See slides: 03isa, pg 9 for classification

Registers have smaller addresses (less to encode) and are faster to access

Stack Architecture
------------------

Example encoding ``a = b * (c+d*b)``

In this class, when using a value on the stack, pop it from the stack

A, B, C, D are in memory, stack in processor

.. code-block::

    # instructions  stack (top left)
    LD D            D
    LD B            B   D
    MULT            D*B
    LD C            C   D*B
    ADD             C+(D*B)
    LD B            B   C+(D*B)
    MULT            B*(C+(D*B))
    ST A            (empty)

Accumulator Architecture
------------------------
The acc lives in the processor

.. code-block::

    # inst      accumulator
    LD B        B
    MULT D      B*D
    ADD C       C+(B*D)
    MULT B      B*(C+(B*D))
    ST A

Reg-Mem Arch (CISC)
-------------------
Multiple registers in the processor, as well as memory access

Accumulator code works here too

Note: Parentheses here indicate a memory operation

.. code-block::

    # inst              R1      R2      R3      R4
    LD   R1, (B)        B
    MULT R2, R1, (D)    B       B*D
    ADD  R3, R2, (C)    B       B*D     B*D+C
    MULT R4, R1, R3     B       B*D     B*D+C   B*(B*D+C)
    ST   R4, (A)        ...

Load-Store Arch (RISC)
----------------------
Arithmetic operations can only operate on registers (no memory input)

.. code-block::

    # inst              R1      R2      R3      R4      R5
    LD   R1, (B)        B
    LD   R2, (D)        B       D
    LD   R3, (C)        B       D       C
    MULT R4, R1, R2     B       D       C       B*D
    ADD  R4, R4, R3     B       D       C       C+B*D
    MULT R5, R4, R1     B       D       C       C+B*D   B*(C+B*D)
    ST   R5, (A)        ...

RISC v. CISC
^^^^^^^^^^^^
ISAs like x86, IBM 360 use CISC
    - (but x86 chips use RISC internally by translating into RISC microops)

ISAs like MIPS, SUN Sparc, RISC-V use RISC

See 03isa, pg 17 for table

Instruction Length/Format
-------------------------

- Fixed Length
    - address of next instruction is easy to compute
    - bad code density: common instructions are just as long as the rare
- Variable Length
    - Better code density
        - x86 averages 3 bytes (from 1-16 per inst)
        - common instructions are shorter
    - fetch and decode are more complex
- Compromise: N fixed sixes

Generic RISC Instructions
-------------------------

Arithmetic/Logical: add, mult, or, xor, and...

Data Movement: load/store, register-to-register

Control: compares, conditional branches, unconditional jump, syscall, procedure call/return

Floating Point

Misc

Control In Depth
^^^^^^^^^^^^^^^^
Conditional branches:

- ``branch <cond> <target>``
    - most commonly bne/beq
    - ``bne R1, R2, target``
    - ``beq target`` - usually follows a ``cmp x y`` instruction

Branches are conditional flow transfer, jumps are unconditional
    - ``jmp <target>``

Function calls and returns are typically implemented with jump instructions

Targets are commonly PC-relative

Jumpl: Jump and Link
    - ``jumpl <reg> <target>``
    - Stores the address of the next instruction in ``reg`` and sets PC to ``target``

Memory Addressing
-----------------

- Byte addressing
    - since 1980, every machine can address 8-bit bytes
    - MIPS memory is a linear array of 2^32 bytes (32 bit addresses)
- Everything is byte-aligned in this class, but operands are not bytes
    - typically a word (4 bytes)
    - or double word (8 bytes)
    - or sometimes half words (2 bytes)

Addressing Modes
^^^^^^^^^^^^^^^^

.. code-block::

    # name          syntax      semantics
    Register        Rs          Reg[s]
    Immediate       #n          n
    # everything below here hits mem
    Reg Indirect    (Rs)        Mem[Reg[s]]
    Direct          (n)         Mem[n]
    Displacement    n(Rs)       Mem[n+Reg[s]]
    # less support below here
    Indexed         (Rs+Rt)     Mem[Reg[s]+Reg[t]]
    Mem Indirect    @(Rs)       Mem[Mem[Reg[s]]]
    Auto Increment  (Rs)+       Mem[Reg[s]]; Reg[s] += d
    Auto Decrement  -(Rs)       Reg[s] -= d; Mem[Reg[s]]
    Scaled          n(Rs)[Rt]   Mem[n+Reg[s]+Reg[t]*d]

