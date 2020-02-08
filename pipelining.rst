Pipelining
==========
Take for example a task with multiple segments that takes different amounts of time.

Doing these tasks one at a time means that it takes a lot of time - instead, overlap non-conflicting parts of tasks!

- Pipelining doesn't improve the latency of a single task, but it massively improves throughput
- Pipeline rate limited by slowest stage
- potential speedup = # of pipe stages
- unbalanced lengths of pipe stages reduces speedup (bottlenecking)
- time to fill/drain pipeline reduces speedup

Datapath Pipeline
-----------------
A processor has 5 pipeline stages, controlled by flops:
    - IF: Instruction Fetch
    - ID: Instruction Decode (and reg read), AKA RF
    - EX: Execute (or calculate address)
    - MEM: Memory access
    - WB: Write register

But also, now we need to pipeline the control signals along with the data!

All the control signals are generated in the ID stage, but are controlled by flops until they reach their target at the
right stage


Pipeline Performance
--------------------
For a single cycle processor, the execution time = IC * time of the longest instruction

But you have to add some overhead for the time it takes for flops to work

Example:
    - Unpipelined
        - Period = 240ns
        - tif=60ns, tid=30ns, tex=50ns, tmem=80ns, twb=20ns
    - Pipelined
        - Period = 85ns
        - longest state is 80+5=85ns (5ns overhead)
    - Speedup: :math:`\frac{IC * CPI_u * Period_u}{IC * CPI_p * Period_p} = \frac{CPI_u * 240}{CPI_p * 85} \approx 2.8* \frac{CPI_u}{CPI_p}`

Hazards
-------
There are some cases where an instruction depends on the result of the previous instruction - a stall

In this class, we always stall at ID stage

.. math::

    speedup = \frac{depth}{1 + \frac{stalls}{inst}}

- Structural hazards
    - 2 or more instructions in the pipeline require the same hardware resource to progress
    - mitigation: older instruction has priority
    - see 08pipe p. 30 for example
- Data hazards
    - 1 instruction has a source operand that is the result of a previous instruction in the pipeline
- Control hazards
    - the execution of an instruction depends in the resolution of a previous branch instruction

Data Hazards
^^^^^^^^^^^^
Occurs when an instruction's source register is the dest for either of the 2 prior insts

The simplest solution is to stall the dependent at ID until the required reg has passed WB (2 cycle delay in consecutive)

- RAW: Read After Write (found in all cpus)
    - J tries to read before I writes
- WAW: Write After Write
    - J writes before I writes
- WAR: Write After Read
    - J writes before I reads

**RAW Mitigation: Bypassing/Forwarding**

- forward the result from a later stage to an earlier stage
- e.g. the result of an ALU op at the end of EX can be passed to the EX of the next instruction
- ALU operations cause no delay now, but load has a 1 clock delay (2 clock latency)
- see 08pipe, p. 43 on