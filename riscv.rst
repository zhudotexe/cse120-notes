RISC-V
======

RISC-V is an open-source ISA

multiple flavours: RV32I (this class), RV64I, RV128I, RV32E

Standard extensions:
    - M: mul/div
    - A: atomic operations
    - F: single precision FP
    - D: double precision FP
    - G: IMAFD

Main resource for this class: https://github.com/jameslzhu/riscv-card/raw/master/riscv-card.pdf

Assembly
--------
Ex: ``add x3, x5, x23``

This operation adds the value of ``x5`` and ``x23`` and stores it in ``x3``

.. note::

    RISC-V has 32 registers ``x0..x31``, and ``x0`` is hardcoded to always be ``0`` (write ops allowed but won't save)

Immediates
^^^^^^^^^^

A constant == immediate == literal == offset

Use ``addi``: ``addi dst, srci, immediate``

.. note::

    in arithmetic immediate operations, the immediate is a signed 12 bit number [-2^11..2^11-1] (-2048..2047)

Instruction Format
^^^^^^^^^^^^^^^^^^

**R-Type** (Register)

``[funct7 (7), rs2 (5), rs1 (5), funct3 (3), rd (5), opcode (7)]``

- funct7: additional opcode field
- rs2: register source 2
- rs1: register source 1
- funct3: additional opcode field
- rd: register destination
- opcode: instruction

**I-Type**

``[imm (12), rs1 (5), funct3 (3), rd (5), opcode (7)]``

**S/B-Type**

``[imm 11:5 (7), rs2 (5), rs1 (5), funct3 (3), imm 4:0 (5), opcode (7)]`` (not exactly)

**U/J-Type**

``[imm (20), rd (5), opcode (7)]`` (not exactly)

Example
"""""""

``add x7, x3, x4``

- funct7: 0x00
- rs2: 4
- rs1: 3
- funct3: 0x0
- rd: 7
- opcode: 0x3

= ``0b0000000_00100_00011_000_00111_0000011``

= ``0x418383``

``addi x8, x2, 48``

- imm: 0x30
- rs1: 2
- funct3: 0x0
- rd: 8
- opcode: 0x13

= ``0b000000110000_00010_000_01000_0010011``

= ``0x03010413``

Load/Store
^^^^^^^^^^

**Sign Extension**: think about ``LBU X1, (X3)`` vs ``LB X2, (X3)``

example: the data in memory2 after this is ``0xFFFFFFFF``. If it was LBU it would be ``0x000000FF``.

.. code-block:: asm

    memory1 = 0x000000FF
    memory2 = ...
    r4 = &memory1
    r5 = &memory2

    lb r2, (r4)
    sw r2, (r5)

.. note::
    Even though ``lb`` only loads 1 byte, it writes to all 32 bits of the register using sign extension!

Example
"""""""

``lbu x15, 0(x10)``

- imm: 0 (the offset from the register)
- rs1: 10
- funct3: 0x4
- rd: 15
- opcode: 0x3

= ``0b000000000000_01010_100_01111_0000011``

= ``0x00054783``

Conditional Branch
^^^^^^^^^^^^^^^^^^
Ex: ``beq`` = branch if equal, ``bne`` = branch if not equal

``beq reg1, reg2, label``

the label is an immediate that's added to the PC

``bne x1, x2, L``

``bltz x1, L`` = branch less than zero

``blez x1, L`` = branch <= 0


Jumps
^^^^^^^^^^^^^^^^^^
``j label``

essentially a goto label, the immediate (2^18) is added to the PC

``jr x1`` = goto value in x1

``jal L`` = goto L, set ra

``jalr x1`` = goto x1, set ra

Example
"""""""
``bne x7, x3, 100`` (100 = ``0b0110_0100``)

- imm = 0b0_0000_0110_0100
- rs2 = 3
- rs1 = 7
- funct3 = 0x1
- opcode = 0x63

.. note::
    B-type instructions move the bits of the immediate around all weirdly, so the actual imm parts are:

    - imm1 = 0000_0011
    - imm2 = 00100

    Also note that the last bit of the immediate is dropped, since instructions are always aligned to an even number

= ``0b00000011_00011_00111_001_00100_1100011``

Function Call/Return
""""""""""""""""""""
usually handled with jump-and-links

``x1`` is often called ``ra`` (return address)

``x2`` is often called ``sp`` (stack pointer)

See 04riscv.pdf, pg 19

AUIPC
^^^^^
"Add Upper Immediate to PC"

- used to build pc-relative addresses, uses the U-type format
- forms a 32-bit offset from the 20-bit U-immediate, filling in the lower 12 bits with 0
- adds this offset to the PC and places the result in ``rd``
- see riscv-card, pseudoinstruction ``call``

Comparisons
^^^^^^^^^^^
``slt x1, x2, x3`` = x1 = x2 < x3

``slti x1, x2, 100`` = x1 = x2 < 100

``sltu x1, x2, x3`` = unsigned comparison
