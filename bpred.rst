Branch Prediction
=================

Static prediction v. dynamic prediction:

- static
    - always predict taken
    - always predict not taken
    - compiler/programmer hint
    - based on target and PC location


Dynamic Prediction
------------------

- Branch History Table (BHT) or Prediction History Table (PHT)
    - One entry for each branch PC
    - Taken/Not taken bit
- Branch Target Buffer (BTB)
    - One entry for each branch PC
    - Target address
- Increasingly important for long pipelines (IDx)
    - x86 vs. RISC-V instruction decode

Pattern History Table
^^^^^^^^^^^^^^^^^^^^^^^^
AKA Branch Prediction Buffer

- small memory indexed by lower portion of branch PC
    - similar to instruction cache, but every access is a hit
        - small, tagless, direct mapped
- simplest: remember the last outcome - that's a one bit PHT
    - targets highly biased branches

- looks at index ``(PC >> 2) % PHT_SIZE`` after IF of branch to predict
- update table as soon as branch outcome is resolves (after EX of branch)
- executes the appropriate instruction next based on the table (maybe after a branch delay slot)


1-Bit PHT
"""""""""
in a 1-bit PHT, mispredictions come in pairs

.. code-block:: text

    P: N T T T T
    O: T T T T N

This loop that runs 4 times has 2 mispredictions!

n-Bit PHT
"""""""""
usually 2 bits

The memory stored in the PHT at the index is 2 bits instead of 1

- taken increments (up to 11), not taken decrements (down to 00)

.. code-block:: python

    if branch_taken:
        if pht[(pc >> 2) % size] != 0b11:
            pht[(pc >> 2) % size] += 1
    else:
        if pht[(pc >> 2) % size] != 0b00:
            pht[(pc >> 2) % size] -= 1


- Use the MSB for the prediction

.. code-block:: text

    loop 1
    P:     N  N  T  T  T
    O:     T  T  T  T  N
    table: 01 10 11 11 10

    loop 2
    P:     N  N  T  T  T
    O:     T  T  T  T  N
    table: 11 11 11 11 10

Branch Target Buffer
^^^^^^^^^^^^^^^^^^^^
Now that we predict whether or not to take the branch, we need to know where it goes.

only predicted if ``(pht(pc) == 1 and is_br) or (is_jmp)``

``btb[(pc >> 2) % size] = nextPC`` (that is, next PC if branch is taken)

Exceptions/Interrupts
---------------------

- Unexpected events that require change in flow of control
    - e.g. switch from user to kernel/privileged mode
- different ISAs use different terms

- Exception
    - arises within cpu
    - e.g. undefined opcode, overflow, div by 0, syscall, etc
- Interrupt
    - e.g. from an external controller (network, I/O)

Dealing with these without a performance hit is hard.

Exceptions are another form of control hazard

- e.g. consider overflow of add during EX
    - prevent rd from being clobbered by add
    - complete previous instructions
    - nullify subsequent instructions
    - set cause and epc register values
    - transfer control to handler

The flow is pretty similar to a mispredicted branch - uses much of the same hardware.

- Nullify = turn an instruction into a nop (or bubble)
    - Reset its RegWrite and MemWrite signals
        - Does not affect the state of the system
    - Resets its branch and jump signals
        - Does not cause unexpected flow control
    - Mark that it should not raise any exceptions of its own
        - Does not cause unexpected flow control
    - Let it flow down the pipeline

Core Example
------------

Design option in In-order pipelining: how many cycles of execute?
    - should execute include the M cycle? (i.e. X0, X1 instead of X, M)
    - execute step not finished until the end of all (no forwarding until then)

Example timeline:

.. code-block:: text

    40: sub x11, x2, x4 | F0 F1 D  X0 X1 W
    44: and x12, x11,x5 |    F0 F1 D  D  X0 W
    48: or  x13, x2, x6 |       F0 F1 F1 D  X0 W
    4C: add x1,  x2, x1 |          F0 F0 F1 D  X0 X1 W
    50: slt x15, x6, X7 |                F0 F1 D  D  X0 W
    54: lw  x16, 50(x7) |                   F0 F1 F1 D  X0 W

Control issues
^^^^^^^^^^^^^^

- How many cycles for a fetch?
    - in this class, assume branch predictor at end of F0 unless stated
- When does the outcome and target of the branch resolve? (D, X, M?)
- Does the ISA have a branch delay slot?
    - can there be exceptions in the branch delay slot?
- Does it stall at decode for instructions after control flow?
    - if not stall, what does it fetch?
    - if it uses a predict, how?

.. code-block:: text

    Pipeline: F0, F1, D0, D1, X, W
    Default policy: not taken
    Outcome: predicted by end of F1
    Target: predicted end of F1
    No delay slot
    Branch actually taken

    40: sub  x11, x2, x4 | F0 F1 D0 D1 X  W
    44: bnz  x12, 80     |    F0 F1 D0 D1 X  W
    48: ori  x13,x11,0xF |       F0 -- -- -- -- --

    80: andi x15,x11,0xF |          F0 F1 D0 D1 X  W
    84: xori x12,x15,0x1 |             F0 F1 D0 D1 X  W

.. code-block:: text

    Pipeline: F0, F1, D0, D1, X, W
    Default policy: not taken
    Outcome: predicted by end of F1
    Target: predicted end of F1
    No delay slot
    Branch not actually taken (known at end of D1) *

    40: sub  x11, x2, x4 | F0 F1 D0 D1 X  W
    44: bnz  x12, 80     |    F0 F1 D0 D1 X  W
    48: ori  x13,x11,0xF |       F0 -- --

    80: andi x15,x11,0xF |          F0 F1 -- -- -- --
    84: xori x12,x15,0x1 |             F0 -- -- -- -- --

    48: ori  x13,x11,0xF |                F0 F1 D0 D1 X  W

.. code-block:: text

    Pipeline: F0, F1, F2, D, X, W
    One delay slot
    Default policy not taken
    Predict taken at end of F2, branch actually taken

    40: sub  x11, x2, x4 | F0 F1 F2 D  X  W
    44: bnz  x12, L4     |    F0 F1 F2 D  X  W
    48: ori  x13,x11,0xF |       F0 F1 F2 D  X  W           (delay slot, always executed)
    L4: andi x15,x11,0xF |          F0 --                   (flushed since prediction)
    L4: andi x15,x11,0xF |             F0 F1 F2 D  X  W

.. code-block:: text

    Pipeline: F0, F1, F2, D, X, W
    One delay slot
    Default policy not taken
    Predict taken at end of F2
    branch not actually taken (known at end of X)

    40: sub  x11, x2, x4 | F0 F1 F2 D  X  W
    44: bnz  x12, L4     |    F0 F1 F2 D  X  W
    48: ori  x13,x11,0xF |       F0 F1 F2 D  X  W               (delay slot, always executed)
    L4: andi x15,x11,0xF |          F0 --                       (flushed since prediction)
    L4: andi x15,x11,0xF |             F0 F1 --                 (flushed since incorrect outcome)
    L4: andi x15,x11,0xF |                   F0 F1 F2 D  X  W

.. code-block:: text

    Pipeline: F0, F1, F2, D, X, W
    One delay slot
    Default policy not taken
    Predict not taken at end of F2
    branch actually taken (known at end of X)

    40: sub  x11, x2, x4 | F0 F1 F2 D  X  W
    44: bnz  x12, L4     |    F0 F1 F2 D  X  W
    48: ori  x13,x11,0xF |       F0 F1 F2 D  X  W               (delay slot, always executed)
    L4: andi x15,x11,0xF |          F0 F1 F2 --                 (flushed since incorrect outcome)
    L4: andi x15,x11,0xF |                   F0 F1 F2 D  X  W

Forwarding Issues
^^^^^^^^^^^^^^^^^

- Does it have forwarding? Which branches do?
- Does WB have a half-write?

.. code-block:: text

    Pipeline: F, D, X0, X1, X2, M, W
    No forwarding
    Half writes

    40: sub x11, x2, x4 | F  D  X0 X1 X2 M  W
    44: and x12, x11,x5 |    F  D  D  D  D  D X0 X1 M  W        (ands only take 2 cycles)

    No half writes
    40: sub x11, x2, x4 | F  D  X0 X1 X2 M  W
    44: and x12, x11,x5 |    F  D  D  D  D  D  D  X0 X1 M  W    (half-write allows for M then X0)

    yes forwarding (half write doesn't matter here)
    40: sub x11, x2, x4 | F  D  X0 X1 X2 M  W
    44: and x12, x11,x5 |    F  D  D  D  X0 X1 M  W             (ands only take 2 cycles)

Exception Issues
^^^^^^^^^^^^^^^^

- Are there precise exceptions?
    - In order WB vs out of order WB
    - Always enforce WAW and WAR stalls at decode

.. code-block:: text

    F0 F1 D  X0 X1 X2 W
       F0 F1 D  X0 W        (this means no precise exceptions!)

    with precise exceptions:
    F0 F1 D  X0 X1 X2 W
       F0 F1 D  D  D  X0 W

More Examples
^^^^^^^^^^^^^

.. code-block:: text

    No delay slot
    branch target resolved at decode
    branch outcome resolved at execute
    F0, F1, D, X (X1 for add/sub), M, WB
    forwarding but no half-write
    predict not taken, actually taken

    40: sub  x1,  x2,  x4 | F0 F1 D  X0 X1 M  WB
    44: add  x12, x1,  x2 |    F0 F1 D  D  X0 X1 M  WB
    48: or   x13, x1,  x2 |       F0 F1 F1 D  D  D  X  M  WB
    4c: bne  x12, x13, 58 |          F0 F0 F1 F1 F1 D  D  X  M  WB
    50: slt  x15, x13, x7 |                f0 f0 f0 f1 f1 d  -- -- --
    54: mult x16, x15, x1 |                         f0 f0 f1 -- -- -- --
    58: lw   x16, 50(x16) |                               f0 F0 F1 D  X  M  WB


